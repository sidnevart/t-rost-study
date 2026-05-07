# 03. Как думать про execution engines (на примере TQ)

> **Цель файла:** научиться видеть **класс задач**, который решает execution engine, и его границы. TQ — частный случай. Если поймёшь общий каркас, сможешь оценить, заменять ли TQ, оставлять, обходить — на основе **своих** наблюдений, не моих.

---

## 3.1 Что вообще делает execution engine

Любой execution engine (TQ, Airflow, Temporal, Camunda, Dagster, Step Functions, Sidekiq) обещает примерно то же:

```text
1. Persist work       — сохранить «что нужно сделать»
2. Schedule           — выбрать, что взять следующим
3. Execute            — запустить worker, подождать ответа
4. Track state        — статусы, ретраи, прогресс
5. Coordinate nodes   — кто-чем занят, кто умер
6. Recover            — что делать, если упало в середине
7. Express graphs     — зависимости, цепочки, fan-out/fan-in
```

Когда смотришь на любой engine — найди, как он реализует каждый из 7. Это работает и для TQ, и для Airflow, и для самописного pool-of-goroutines.

---

## 3.2 Оси, по которым отличаются engines

```text
Persistence
  - in-memory      (быстро, теряется при рестарте)
  - log-based      (Kafka, NATS Streaming)
  - DB-backed      (PostgreSQL, Redis, dedicated)

Scheduling
  - pull (workers сами идут за работой)
  - push (scheduler толкает в worker-ов)

Concurrency model
  - thread / coroutine / goroutine / process
  - синхронные блокирующие vs async

State machine
  - финитный набор статусов
  - переходы атомарны / не атомарны

Coordination
  - leader election (one scheduler, others standby)
  - leaderless (все равны, conflict через locks)
  - sharded (каждый owns shard)

Time semantics
  - polling interval
  - event-driven trigger
  - deadline-aware vs static priority

Failure handling
  - retry policy (count, backoff, deadletter)
  - poison pill detection
  - timeout handling

Durability vs speed
  - sync write на каждый шаг (durable, slow)
  - batch writes (faster, can lose recent state)
```

Когда читаешь TQ — отметь, какой выбор по каждой оси. Эти выборы и есть «характер» engine.

---

## 3.3 Важная разница: библиотека vs сервис

```text
Library (TQ, Sidekiq, Resque):
+ tight integration с приложением
+ shared DB / shared deploy
- coupled lifecycle с приложением
- общий resource pool

Service (Airflow, Temporal, Step Functions):
+ изолированный lifecycle
+ multi-tenant
- сетевой round-trip к engine
- отдельная операционная нагрузка
```

TQ — **библиотека**. Это важно: «улучшить TQ» = выпустить новую версию библиотеки и заставить всех потребителей обновиться. Это медленно. **Это не плохо и не хорошо**, это свойство, которое влияет на стратегию изменений.

---

## 3.4 Когда execution engine — переоценен (не уместен)

```text
- Задача = одно действие, синхронное, < 100ms.
  → engine = lookup overhead. Просто вызови функцию.

- Задача требует HTTP-style request/response.
  → engine добавляет latency. Используй sync RPC.

- Задачи десятки тысяч в секунду.
  → DB-backed engines захлёбываются. Нужен log-based (Kafka) или специализированный.

- Нужна exactly-once.
  → большинство engines дают at-least-once. Idempotent worker обязателен.

- Нужна streaming-семантика (continuous data flow).
  → нужен Kafka Streams / Flink / Beam, не engine.
```

Это **рамка вопросов**, а не вердикт «уберём TQ». Чтобы сказать это про конкретный класс задач, тебе нужны **числа** (см. `02-architecture/03-capacity-and-performance-thinking.md`).

---

## 3.5 Когда engine незаменим

```text
- Многошаговый процесс с явными зависимостями.
- Steps могут падать независимо и retry-иться независимо.
- Нужен audit trail — кто, когда, что.
- Длинные процессы (минуты+), которые переживают рестарт worker-а.
- Несколько worker pool-ов с разными ресурсами.
```

---

## 3.6 Что изучить, чтобы понимать engines

| Тема | Что даёт | Источник |
|---|---|---|
| Queueing theory basics | M/M/c, Little's law, утилизация и latency | книга «Performance Modeling and Design of Computer Systems», Mor Harchol-Balter |
| State machine design | как корректно описать lifecycle | DDIA гл. 7-8 |
| FOR UPDATE SKIP LOCKED | DB-backed queue паттерн | блог 2ndquadrant, Postgres docs |
| Coordination primitives | leases, fencing, heartbeat | DDIA гл. 8-9 |
| Workflow engines comparison | Airflow / Temporal / Step Functions | их доки + post-mortems из чужого опыта |
| Coroutines (для TQ) | Kotlin Coroutines deep dive | книга Moskała |

---

## 3.7 Как смотреть на TQ

Не пиши за меня выводы. Сделай это сам:

### Найди и прочитай
- `application/api/common-api/.../configuration/TQConfig.kt` — все настройки.
- `application/engine/scheduler/core/.../TQScheduler.kt` — главный цикл.
- `application/engine/storage/postgresql/.../task/TQJdbiTaskDao.kt` — все SQL.
- `db/changelog/` — миграции.
- `docs/intro.md`, `docs/API/configuration/TQConfig.md`, `docs/API/core-processes/continuous-processes.md`.

### Для каждого пункта 3.1 (Persist / Schedule / Execute / ... / Express graphs) — ответь
```text
- КАК это реализовано в TQ?
- Какие trade-off-ы выбраны?
- Что бы было сложно поменять, а что — легко?
```

### Для каждой оси 3.2 — пометь TQ
- pull/push? Persistence? И т.д.

### Сформируй СВОИ гипотезы про границы
```text
- Для каких classes of tasks TQ — overhead?
- Для каких — незаменим?
- Где на наших данных мы видим симптомы переоценённости / недооценённости?
```

---

## 3.8 Упражнения

```text
УПРАЖНЕНИЕ A. Сравни TQ с Airflow по 3.2. Сделай side-by-side таблицу.
              Это — твоё упражнение, не мой текст.

УПРАЖНЕНИЕ B. Напиши mini-engine на 200 строк (Kotlin/Go/Python — твой выбор) с 3
              из 7 функций (3.1). Поделись им с тимлидом — обсуди свои выборы.

УПРАЖНЕНИЕ C. Прочти 1 post-mortem про какой-нибудь queueing engine (Sidekiq blog, Cadence
              war stories, Airflow upgrade incidents). Извлеки 3 урока.

УПРАЖНЕНИЕ D. Сформулируй 3 СВОИХ вопроса про TQ, на которые код не даёт ответа.
              Не из моего списка в 01-context/07.
```

---

## DoD

```text
[ ] Я могу нарисовать на доске 7 функций engine и для каждой показать в TQ конкретный код.
[ ] Я могу аргументированно сказать, для какого класса задач у нас TQ уместен / нет.
[ ] У меня есть mini-engine ≥ 200 строк.
[ ] У меня есть 3 СВОИХ вопроса.
```
