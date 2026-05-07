# 07. Time, freshness, ordering — фундамент, на котором строится «свежесть»

> **Цель файла:** научиться видеть **разные виды времени** в распределённой системе и через них формулировать вопросы про свежесть данных. Без этого «аудитория свежая» — пустая фраза.

---

## 7.1 Виды времени в distributed system

```text
1. Wall-clock time          — то, что показывают часы (NTP-синхронизированные, дрейфуют)
2. Event time               — момент события в реальности (когда клиент сделал tx)
3. Ingestion time           — момент попадания в нашу систему
4. Processing time          — момент обработки конкретным узлом
5. Logical time / version   — порядок без привязки к физическому времени (LSN, lamport)
6. Watermark                — гарантия "все события с event_time ≤ T — учтены"
```

Большинство багов "почему результат странный" — это путаница между этими видами времени. Особенно event vs processing.

---

## 7.2 «Свежесть» — это не одно число

Когда кто-то говорит «аудитория свежая», он может иметь в виду:

```text
A. data_as_of      — на какой момент актуальны данные источника (event time)
B. computed_at     — когда compute закончил работу (processing time)
C. delivered_at    — когда result попал в downstream
D. consumed_at     — когда downstream применил
E. observation gap — насколько early/late event попал в compute
```

Различия:
```text
data_as_of → computed_at      = compute lag
computed_at → delivered_at    = pipeline lag
delivered_at → consumed_at    = consumer lag
data_as_of → consumed_at      = total staleness (то, что обычно болит)
```

Если в коде ты не находишь этих 4 timestamp-ов на разных уровнях — система **не различает** виды свежести. Это либо OK для бизнеса, либо open question.

---

## 7.3 Watermark — главный инструмент потоковой системы

```text
Watermark T   = "все события с event_time ≤ T уже учтены".

Если ты поставишь компонент, который читает события и обновляет state,
его правильность зависит от watermark:
  - rebuild всё < T  → детерминированный результат для T.
  - применил event с event_time > current_watermark → могло пропуститься.

Watermark — функция от сетевых задержек, retry, late arrivals.
Streaming Systems book — главный источник для понимания.
```

---

## 7.4 Reproducibility

Это свойство, отдельное от свежести:

```text
Если я пересчитаю result для (definition, data_as_of=T) — получу ли тот же ответ?
```

Случаи:
```text
- Источник иммутабельный (партиции по дате, replay-able log) → да
- Источник mutable (UPDATE по pk) → нет, без снапшотинга
- Streaming + late arrivals → почти всегда нет
- ClickHouse partition by date → да для прошлого, нет для текущего
```

Reproducibility — критичная задача аналитики и аудита. Если её нет — часть инсайтов «недосказуема».

---

## 7.5 Ordering и его стоимость

```text
Per-key ordering        — обычно дешёво (Kafka partition by key)
Global ordering         — обычно дорого (single partition / consensus)
Causal ordering         — компромисс через vector clocks / hybrid logical clocks
No ordering             — событие самостоятельно

Цена strong ordering = ниже throughput / выше latency.
```

Когда читаешь любой Kafka topic — найди partitioning key. Это и есть unit of ordering. Если ordering critical, но key плохой — баг.

---

## 7.6 Late arrivals и out-of-order events

В реальности события приходят:
- с задержкой,
- в неправильном порядке,
- иногда дважды,
- иногда теряются.

Что должна делать система:
```text
1. Принять как есть.
2. Решить, что делать с late events:
   a) drop (если за watermark)
   b) reprocess (re-emit обновлённый result)
   c) accept silently (drift накапливается)
3. Обеспечить idempotent merge (для b).
```

В A-P/Target — найди, что выбрано на каждом ребре. Если нет ответа — open question.

---

## 7.7 Cache freshness — не та же задача

```text
Cache TTL       — время, после которого result считается stale.
Cache invalidation — момент, когда мы знаем "данные изменились, кэш невалиден".

Trade-off:
  TTL слишком большой → stale data
  TTL слишком маленький → low hit rate
  Invalidation — сложно, потому что нужно знать ВСЕ зависимости.

Phil Karlton: "two hard things in CS — cache invalidation, naming things".
```

Cache freshness требует:
- знания зависимостей (от каких источников зависит result),
- механизма invalidate (push event vs pull check),
- защиты от stampede (один компромис: stale-while-revalidate).

---

## 7.8 Что изучить

| Тема | Что даёт | Источник |
|---|---|---|
| **Streaming Systems** Akidau et al. | event vs processing time, watermarks, triggers | book |
| **DDIA гл. 8-9** | distributed time, ordering, consistency | book |
| Lamport timestamps + vector clocks | logical time | Lamport's original paper (короткий) |
| Hybrid Logical Clocks | physical + logical | HLC paper Kulkarni et al. |
| CRDTs | eventually consistent merging | crdt.tech overview |
| Kafka exactly-once semantics | реальная цена ordering | Confluent docs |
| Cache patterns | read-through, write-through, refresh-ahead | блоги Cloudflare/Netflix |

---

## 7.9 Упражнения

```text
УПРАЖНЕНИЕ A. Для одного результата сборки аудитории — найди все 4 timestamp-а
              (data_as_of / computed_at / delivered_at / consumed_at).
              Если timestamp-а нет — записал бы ты его?

УПРАЖНЕНИЕ B. Найди в коде A-P/Target: какие 2 точки используют разные виды времени
              (например, scheduling по wall-clock, но событийная логика по event time)?
              Может ли это привести к багу?

УПРАЖНЕНИЕ C. Прочти Lamport's «Time, Clocks, and the Ordering of Events» (10 страниц).
              Сформулируй, какой вид времени реализован в `tq_task.date_started`.

УПРАЖНЕНИЕ D. Сформулируй СВОЁ определение «свежей аудитории», основанное на
              нашей реальности и бизнес-нагрузке. Не моё.
```

---

## DoD

```text
[ ] Я могу различить 6 видов времени в коде нашего проекта.
[ ] Я понимаю, что значит watermark, и где он у нас (если есть).
[ ] У меня есть СВОЁ определение свежести аудитории, согласованное с PM-ом.
[ ] Я знаю, какие из наших данных reproducible, какие — нет, и почему.
```
