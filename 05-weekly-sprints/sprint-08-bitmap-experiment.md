# Sprint 08 — Bitmap Experiment

## Цель недели
Прогнать конкретный bitmap-эксперимент на синтетических данных, оценить порядок выигрыша и ограничения. Ответить: «есть ли смысл в bitmap-serving для нашего use-case».

## Почему это важно
Это последний sprint перед тем, как ты будешь принимать решение о гипотезе. Bitmap — один из инструментов. Эксперимент покажет, где его границы в твоём конкретном use-case.

---

## GPT-ускоритель

**Перед началом спринта** → вставь из `gpt-prompts.md` → Sprint 08 → Шаг 2 (Weekly Sprint).
**Если застрял на RoaringBitmap internals** → вставь из `gpt-prompts.md` → Track 09 → Шаг 3 (Deep Explanation).
**В конце спринта** → вставь из `gpt-prompts.md` → Track 09 → Шаг 4 (Self-check). Финальный вывод должен содержать числа — без них не считается.

---

## День 1 — Postановка эксперимента
- Сценарий: 10M client_id-ов, 100 «сегментов» (city / age / subscription).
- Цель: поддержать `intersect(A, B)`, `union(A, B)`, `count(A)` за миллисекунды.

**Практика:** сгенерировать данные.

**Артефакт:** dataset (CSV или CH-table).

**DoD:** есть тестовый набор.

---

## День 2 — RoaringBitmap (Java)
- Импортировать `org.roaringbitmap.RoaringBitmap`.
- Загрузить 100 сегментов.
- Замерить:
  - in-memory size,
  - time to build,
  - time intersect 2 bitmap-ов разных размеров.

**Практика:** Kotlin pet-project с RoaringBitmap.

**Артефакт:** числа.

**DoD:** есть сравнение «1M intersect = X ms».

---

## День 3 — ClickHouse bitmap
- `groupBitmapAnd`, `groupBitmapOr` в CH.
- Сравнить с standalone RoaringBitmap.

**Практика:** запросы на тех же данных.

**Артефакт:** числа.

**DoD:** понимаю, когда CH-native лучше, когда — отдельный сервис.

---

## День 4 — Range predicates и hybrid
- Что делать, если params содержит range (income from-to)?
- Гибрид: bitmap для discrete + CH для range, потом intersect.

**Практика:** прототип hybrid pipeline.

**Артефакт:** замер hybrid vs pure-CH.

**DoD:** есть оценка границ применимости bitmap.

---

## День 5 — Rollup
- Заполнить эксперимент в `06-practice/01-experiment-backlog.md` числами.
- Написать вывод: «что я увидел, что подтвердилось, что опровергнуто».

**Практика:** написать заключение по шаблону эксперимента.

**Артефакт:** 1 страница с числами и выводом.

**DoD:** вывод опирается на числа, а не на интуицию.

---

## Итог недели

```text
- Bitmap-эксперимент (стандалон + CH-native + hybrid).
- Числа с дисперсией.
- Твой вывод о границах применимости bitmap.
```

## Финальный артефакт
`practice/sprint-08-bitmap-report.md` — твои числа + вывод. Без числового вывода — артефакт не считается сделанным.

## Self-check
1. Сколько памяти занимает bitmap для 1M client_id?
2. На каких операциях bitmap проигрывает CH?
3. Кто и когда обновляет bitmap в realtime-сценарии?
4. Чем CH `groupBitmapAnd` хуже отдельного serving-сервиса (latency, throughput)?
5. Реалистично ли поддерживать 1000+ сегментов в bitmap-сервисе?

---

## После 8 спринтов

```text
К этому моменту у тебя есть:
- Полная end-to-end карта (sprint 1-2).
- Числа для compute/export/delivery (sprint 3).
- Сравнение путей выполнения на одном use-case (sprint 4).
- Понимание TQ + Coroutines (sprint 5).
- PG-queue mastery (sprint 6).
- CH-baseline (sprint 7).
- Bitmap experiment (sprint 8).

Теперь у тебя достаточно наблюдений, чтобы сформулировать собственную гипотезу.
Прочитай 07-final-rfc/01-final-rfc-outline.md — там критерии готовности RFC.
```
