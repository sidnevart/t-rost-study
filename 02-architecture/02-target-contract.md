# 02. Как думать про контракты между сервисами

> **Цель файла:** научить тебя читать ЛЮБОЙ контракт (REST/Kafka/gRPC/Avro/Protobuf) с правильными вопросами. Если ты освоишь это — A-P↔Target будет частным случаем, а не уникальной задачей.

---

## 2.1 Что такое «контракт» и какие свойства у него важны

Контракт — это **обещание** между двумя системами. Свойства:

```text
1. Shape           — какие поля, типы, обязательность
2. Semantics       — что эти поля МЕАНТ значить (не то же, что shape)
3. Compatibility   — как контракт меняется во времени
4. Reliability     — какие гарантии (delivery, ordering, dedup, ack)
5. Authority       — кто владелец, кто принимает решения о смене
6. Observability   — можно ли увидеть, что происходит на границе
```

Большинство инженеров видят только Shape. Senior-engineer держит все 6 в голове.

---

## 2.2 Семантика доставки (это критично)

Любой message-passing контракт надо классифицировать по 3 осям:

```text
A. Delivery guarantee
   - At-most-once   — может потеряться, не повторится
   - At-least-once  — не потеряется, может повториться
   - Exactly-once   — почти миф (требует idempotent receiver или consensus)

B. Ordering
   - Per-partition / per-key   — Kafka типично
   - Global                    — обычно дорого
   - None                      — каждый event сам по себе

C. Failure mode visibility
   - Sender знает, что не дошло (sync ack)
   - Sender не знает (fire-and-forget)
   - Sender знает с задержкой (DLQ, status callback)
```

### Что отсюда следует
Когда ты читаешь любой контракт в проекте, ты должен уметь сказать:
- Какой guarantee использован?
- На чём держится ordering, если он важен?
- Что делает sender при неудаче?

Если на любой из 3 у тебя нет ответа из кода — это **открытый вопрос**, а не «и так понятно».

---

## 2.3 Push vs Pull: рамка решения

```text
Push:
+ low latency для consumer
+ простая модель «producer знает downstream»
- producer должен знать всех consumers
- back-pressure сложно (consumer может задыхаться)
- replay сложен (нужно хранить историю)

Pull:
+ consumer контролирует темп
+ replay — естественный
+ multi-consumer без знания producer
- higher latency (polling)
- consumer должен знать, КАК спросить
```

Ни один не «лучше». Каждый случай — пара вопросов:
1. Кому важнее latency: producer-у (хочет толкнуть) или consumer-у (хочет читать в своём ритме)?
2. Замены consumer-ов: часто или редко? Появление новых: дорого или дёшево?

---

## 2.4 Идемпотентность — фундамент

Любая at-least-once система требует идемпотентного receiver-а. Идемпотентность даётся одним из:

```text
1. Natural idempotency — операция уже идемпотентна по природе
   (UPDATE x SET v=5, INSERT ... ON CONFLICT DO NOTHING).
2. Idempotency key — receiver хранит "видел этот id" + result.
3. State versioning — receiver проверяет version и игнорирует stale.
4. Tombstone / dedup window — окно «не принимать одинаковое за N часов».
```

Когда читаешь контракт, спроси: **где ровно** реализована идемпотентность? Если её нет — **это не значит, что её не надо**, это значит, что система делает вид, что at-least-once = exactly-once. Это всегда боль.

---

## 2.5 Эволюция контракта — главный политический вопрос

Контракт = capital. Стоит дороже, чем большинство кода. Менять его страшно. Поэтому:

```text
- Backward compatibility   — старый клиент работает с новым сервером
- Forward compatibility    — новый клиент работает со старым сервером
- Full compatibility       — оба
```

Avro даёт BACKWARD/FORWARD/FULL — это политические решения, а не технические. Зачем тебе понимать:
- Когда ты предложишь изменение — ты должен знать, какой режим у нашего регистра.
- Как выкатить изменение, если режим строгий: добавить поле с default, не удалять, не переименовывать.
- Как договориться с downstream-ами о breaking change (deprecation period).

---

## 2.6 Как читать контракт нашего проекта

Не копируй мои выводы. Сделай **ты**:

### Шаг 1. Найди ВСЕ контракты
```text
- Avro:       contract/avro/*.avsc (в обоих репо)
- Proto:      contract/grpc-api/proto/*.proto, contract/proto/*.proto
- OpenAPI:    contract/openapi/*.{yaml,json}, contract/typespec/*.tsp
- JSON:       contract/json/*.go
- Kafka cfg:  application.yaml topics
```

### Шаг 2. Для каждого контракта ответь 6 вопросов из 2.1
- Shape, Semantics, Compatibility, Reliability, Authority, Observability.
- Где не знаешь — это твой открытый вопрос.

### Шаг 3. Расположи на карте контрактов
```text
A-P ↔ Target ↔ T-Segment ↔ каналы (cb-platform, extra-cb, push, ...)
```
- Какой контракт на каком ребре?
- Какие гарантии каждое ребро даёт?
- Где в графе самое слабое звено по reliability?

---

## 2.7 Что тебе изучать, чтобы это видеть

| Тема | Что даёт |
|---|---|
| **Enterprise Integration Patterns** (Hohpe & Woolf) | язык всех patterns: routing, transformation, dedup, retry, DLQ |
| Kafka в DDIA, гл. 11 | log-based messaging, exactly-once в реальности |
| Avro/Protobuf schema evolution rules | как менять контракт без поломки |
| CAP / PACELC | граница между consistency и availability |
| Idempotency patterns | natural / key / version / tombstone |
| Distributed transactions / saga | когда «два сервиса должны быть согласованы» |

---

## 2.8 Упражнения

```text
УПРАЖНЕНИЕ A. Найди в коде A-P/Target КАЖДЫЙ контракт. Сделай таблицу:
   contract | shape source | semantics doc | compat mode | guarantees | owner | observability

УПРАЖНЕНИЕ B. На каком из контрактов проще всего сейчас выкатить новое поле?
   На каком — сложнее всего? Почему?

УПРАЖНЕНИЕ C. Найди 3 места, где «exactly-once» имитируется через at-least-once + idempotency.
   Если не нашёл — это либо у нас её нет, либо ты не нашёл. Что из двух?

УПРАЖНЕНИЕ D. Если бы ты сейчас вводил breaking change в `auditory-export.avsc`,
   опиши план migration на 1 страницу. Не реализуй — просто план.
```

---

## DoD

```text
[ ] У меня есть таблица всех контрактов.
[ ] Я могу для каждого назвать guarantees + owner + observability.
[ ] У меня есть 5 СВОИХ вопросов про контракты, на которые код не отвечает.
[ ] Я понимаю, чем отличается изменение Avro vs изменение REST vs изменение Kafka topic.
```
