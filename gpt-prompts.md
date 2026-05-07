# GPT-промпты для Learning OS

Готовые промпты для каждого трека и спринта по 4-шаговой системе.
Вставляй `[ТЕМА]` и `[ЧТО ХОЧУ УМЕТЬ]` из заголовка трека/спринта.

---

## Как пользоваться

1. **Шаг 1 → Learning Curator** — запускаешь 1 раз перед треком/спринтом. Получаешь карту ресурсов.
2. **Шаг 2 → Weekly Sprint** — запускаешь 1 раз, получаешь план на 5 дней.
3. **Шаг 3 → Deep Explanation** — запускаешь, когда упёрся в непонятную тему внутри трека.
4. **Шаг 4 → Self-check** — запускаешь в конце трека/спринта для верификации знаний.

---

## Track 01 — Audience Platform Domain

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: Audience Platform Domain — модель аудитории, lifecycle, типы фильтров (AuditoryParams), STATIC vs DYNAMIC.

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes.

Цель:
Уметь нарисовать на доске полный lifecycle аудитории (DRAFT→GATHERING→GATHERED→ARCHIVED) и объяснить разницу STATIC vs DYNAMIC без подсказки. Понять, как AuditoryParams DSL транслируется в фильтры CH-запроса.

Подбери мне ресурсы по системе:
1. Official docs — только самое нужное
2. Best article — объяснение простыми словами
3. Deep article/book chapter — для глубины
4. Video/talk — если полезно
5. Practice task — что сделать руками
6. Work integration — как применить на работе
7. Self-check questions — 10 вопросов

Ограничения:
- Не давай 20 ссылок. Максимум 5–7 ресурсов.
- Для каждого ресурса объясни, зачем он нужен.
- Дай порядок изучения.
- В конце дай DoD: что я должен уметь после изучения.
```

### Шаг 3 — Deep Explanation (если тема «AuditoryParams DSL»)
```
Объясни тему: Domain-Specific Language (DSL) для параметров аудитории — как набор filter-объектов (city, age, subscription, MCC, etc.) превращается в SQL/ClickHouse запрос.

Формат:
1. Простое объяснение в 5 предложениях
2. Ментальная модель
3. Как это работает технически
4. Где это ломается
5. Пример из backend/microservices
6. Пример на Kotlin/Java/Go
7. Типовые ошибки
8. 10 вопросов для проверки
9. Мини-практика на 30 минут

Не пиши водно. Объясняй так, чтобы я мог применить это на работе.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: Audience Platform Domain — lifecycle аудитории, AuditoryParams, STATIC vs DYNAMIC, gathering_id.

Задавай вопросы по одному.
Формат:
- сначала простой вопрос
- потом практический кейс
- потом вопрос на trade-offs
- потом вопрос уровня собеседования middle+/senior

После каждого моего ответа:
1. Оцени ответ по шкале 1–10
2. Укажи пробелы
3. Дай правильную версию
4. Задай следующий вопрос

Не хвали без причины.
```

---

## Track 02 — Target Integration

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: Интеграция двух микросервисов через REST + Kafka + Avro. Паттерны: request-response поверх Kafka, Avro schema evolution, JWT-авторизация между сервисами, idempotency.

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes.

Цель:
Уметь объяснить полный контракт A-P↔Target: REST-направления, Kafka-топики, формат Avro, JWT. Понять риски нарушения контракта и где idempotency гарантирована/не гарантирована.

Подбери ресурсы по системе (max 5–7), с порядком изучения и DoD.
```

### Шаг 3 — Deep Explanation (если тема «Avro schema evolution»)
```
Объясни тему: Avro Schema Evolution — BACKWARD, FORWARD, FULL совместимость.

Формат:
1. Простое объяснение в 5 предложениях
2. Ментальная модель
3. Как это работает технически
4. Где это ломается
5. Пример из backend/microservices (Kafka + Schema Registry)
6. Пример на Kotlin/Java/Go
7. Типовые ошибки
8. 10 вопросов для проверки
9. Мини-практика на 30 минут

Не пиши водно.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: интеграция микросервисов через Kafka — Avro schema evolution, idempotency, at-least-once delivery, deduplication.

Задавай вопросы по одному (простой → кейс → trade-off → senior-уровень).
После каждого ответа: оценка 1–10, пробелы, правильная версия, следующий вопрос.
Не хвали без причины.
```

---

## Track 03 — TQ / Kotlin Coroutines

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: Kotlin Coroutines — structured concurrency, CoroutineScope, Dispatchers, suspend functions, CancellationException, SupervisorJob. JVM internals: GC, JIT, JFR profiling, async-profiler.

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes.

Цель:
Читать чужой Kotlin Coroutines-код без шпаргалки. Профилировать JVM-приложение через JFR/async-profiler. Объяснить, почему SupervisorJob у scheduler-а, а не обычный Job.

Подбери ресурсы по системе (max 5–7), с порядком изучения и DoD.
```

### Шаг 2 — Weekly Sprint
```
Составь мне weekly learning sprint.

Тема недели: Kotlin Coroutines + JVM profiling для понимания task-queue (TQ) на Kotlin.

Цель недели:
Написать mini-TQ-engine (~200 строк Kotlin) с coroutines. Профилировать JVM через async-profiler. Читать TQScheduler.kt без шпаргалки.

Мой контекст:
Backend engineer, Kotlin/Java/Go, микросервисы, Kafka, ClickHouse, Postgres, K8s.

Ограничения:
- В день 60–90 минут.
- Каждый день должен заканчиваться артефактом.
- Не больше 1–2 ресурсов в день.
- Должна быть практика руками.
- В конце недели должен быть итоговый проект или документ.

Формат: Понедельник: Цель / Ресурсы / Практика / Артефакт / DoD. И так на 7 дней.
```

### Шаг 3 — Deep Explanation (если тема «suspend function state machine»)
```
Объясни тему: Kotlin suspend function — как работает state machine под капотом (Continuation, CPS transformation, bytecode).

Формат:
1. Простое объяснение в 5 предложениях
2. Ментальная модель
3. Как это работает технически (с примером bytecode)
4. Где это ломается
5. Пример из backend/microservices
6. Пример на Kotlin
7. Типовые ошибки
8. 10 вопросов для проверки
9. Мини-практика на 30 минут — написать suspend-функцию и посмотреть её bytecode через kotlinc + javap

Не пиши водно.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: Kotlin Coroutines — structured concurrency, CancellationException, SupervisorJob, Dispatchers, suspend state machine.

Задавай вопросы по одному (простой → кейс → trade-off → senior-уровень).
После каждого ответа: оценка 1–10, пробелы, правильная версия, следующий вопрос.
Не хвали без причины.
```

---

## Track 04 — PostgreSQL Queues / SKIP LOCKED / LISTEN-NOTIFY

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: PostgreSQL как очередь задач — FOR UPDATE SKIP LOCKED, LISTEN/NOTIFY, partial indexes, autovacuum tuning, MVCC.

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes.

Цель:
Понять, как TQ использует PG как очередь. Читать EXPLAIN ANALYZE без угадывания. Написать LISTEN/NOTIFY-паттерн как альтернативу cron-poll-у. Настроить autovacuum для write-heavy очереди.

Подбери ресурсы по системе (max 5–7), с порядком изучения и DoD.
```

### Шаг 2 — Weekly Sprint
```
Составь мне weekly learning sprint.

Тема недели: PostgreSQL как task queue — SKIP LOCKED, LISTEN/NOTIFY, autovacuum, partial indexes.

Цель недели:
Написать LISTEN/NOTIFY demo на 150 строк (Go/Kotlin). Замерить throughput SKIP LOCKED vs число потребителей. Читать EXPLAIN ANALYZE.

Мой контекст:
Backend engineer, Kotlin/Java/Go, микросервисы, Kafka, ClickHouse, Postgres, K8s.

Ограничения: 60–90 минут в день. Артефакт каждый день. Практика руками.

Формат: Понедельник: Цель / Ресурсы / Практика / Артефакт / DoD. И так на 7 дней.
```

### Шаг 3 — Deep Explanation (если тема «MVCC + SKIP LOCKED»)
```
Объясни тему: PostgreSQL MVCC и FOR UPDATE SKIP LOCKED — как работает механизм, почему эффективен для очередей.

Формат:
1. Простое объяснение в 5 предложениях
2. Ментальная модель
3. Как это работает технически
4. Где это ломается
5. Пример из backend/microservices (task queue)
6. Пример на SQL
7. Типовые ошибки
8. 10 вопросов для проверки
9. Мини-практика на 30 минут

Не пиши водно.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: PostgreSQL как очередь — SKIP LOCKED, LISTEN/NOTIFY, partial indexes, HOT update, autovacuum для write-heavy workload.

Задавай вопросы по одному (простой → кейс → trade-off → senior-уровень).
После каждого ответа: оценка 1–10, пробелы, правильная версия, следующий вопрос.
Не хвали без причины.
```

---

## Track 05 — Distributed Coordination

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: Distributed Systems — CAP, consensus, fencing tokens, Saga pattern, split-brain, leader election, at-least-once + idempotency = exactly-once.

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes.

Цель:
Понять, какие гарантии даёт TQ на PG без consensus. Спроектировать compensating actions для одного use-case в A-P/Target. Читать post-mortems distributed систем и видеть причину.

Подбери ресурсы по системе (max 5–7), с порядком изучения и DoD.
```

### Шаг 3 — Deep Explanation (если тема «Saga pattern»)
```
Объясни тему: Saga Pattern — choreography vs orchestration, compensating transactions, distributed consistency без 2PC.

Формат:
1. Простое объяснение в 5 предложениях
2. Ментальная модель
3. Как это работает технически
4. Где это ломается
5. Пример из backend/microservices (audience building pipeline)
6. Пример на Kotlin/Java/Go
7. Типовые ошибки
8. 10 вопросов для проверки
9. Мини-практика на 30 минут

Не пиши водно.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: Distributed systems — CAP, fencing tokens, Saga, split-brain, at-least-once + idempotent = exactly-once, leader election без Raft.

Задавай вопросы по одному (простой → кейс → trade-off → senior-уровень).
После каждого ответа: оценка 1–10, пробелы, правильная версия, следующий вопрос.
Не хвали без причины.
```

---

## Track 06 — DAG Algorithms & Critical Path

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: DAG Algorithms — topological sort, critical path method (CPM), levelization, parallel execution limits, cycle detection.

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes.

Цель:
Написать скрипт на Go/Kotlin, который читает граф TQ-задач из PG и находит critical path. Понять, почему ширина DAG не всегда = speedup. Аргументировать, когда DAG избыточен.

Подбери ресурсы по системе (max 5–7), с порядком изучения и DoD.
```

### Шаг 3 — Deep Explanation (если тема «Critical Path Method»)
```
Объясни тему: Critical Path Method (CPM) в контексте task queues — earliest start, latest start, float, critical path.

Формат:
1. Простое объяснение в 5 предложениях
2. Ментальная модель
3. Как это работает технически
4. Где это ломается
5. Пример из backend/microservices (TQ task graph)
6. Пример на Kotlin/Go
7. Типовые ошибки
8. 10 вопросов для проверки
9. Мини-практика на 30 минут

Не пиши водно.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: DAG algorithms — topological sort, critical path, levelization, почему depth × interval = lower bound на latency.

Задавай вопросы по одному (простой → кейс → trade-off → senior-уровень).
После каждого ответа: оценка 1–10, пробелы, правильная версия, следующий вопрос.
Не хвали без причины.
```

---

## Track 07 — ClickHouse Performance

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: ClickHouse performance — MergeTree, primary key vs sorting key, granule, EXPLAIN PIPELINE, materialized views, projections, partition pruning.

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes.

Цель:
Читать EXPLAIN PIPELINE и понимать bottleneck. Написать materialized view для повторяющегося запроса и замерить speedup. Знать, когда CH-native path лучше DAG.

Подбери ресурсы по системе (max 5–7), с порядком изучения и DoD.
```

### Шаг 3 — Deep Explanation (если тема «MergeTree primary key vs index»)
```
Объясни тему: ClickHouse MergeTree — как работает primary key, sparse index, granule, partition pruning, и чем это отличается от PostgreSQL.

Формат:
1. Простое объяснение в 5 предложениях
2. Ментальная модель
3. Как это работает технически
4. Где это ломается
5. Пример из backend (audience query)
6. Пример SQL
7. Типовые ошибки
8. 10 вопросов для проверки
9. Мини-практика на 30 минут

Не пиши водно.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: ClickHouse — MergeTree, granule, sparse index, partition pruning, EXPLAIN ANALYZE vs EXPLAIN PIPELINE, когда materialized view помогает.

Задавай вопросы по одному (простой → кейс → trade-off → senior-уровень).
После каждого ответа: оценка 1–10, пробелы, правильная версия, следующий вопрос.
Не хвали без причины.
```

---

## Track 08 — Data Locality / S3 / Predicate Pushdown

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: Data locality в распределённых системах — S3 transfer patterns, CopyObject vs download/upload, Parquet vs CSV, columnar formats, predicate pushdown.

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes.

Цель:
Найти и устранить двойную упаковку S3 в pipeline. Написать бенчмарк CSV vs Parquet (байты, время чтения). Понять, когда CopyObject невозможен.

Подбери ресурсы по системе (max 5–7), с порядком изучения и DoD.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: S3 data transfer patterns — CopyObject, pre-signed URLs, Parquet vs CSV, columnar formats, predicate pushdown в CH s3().

Задавай вопросы по одному (простой → кейс → trade-off → senior-уровень).
После каждого ответа: оценка 1–10, пробелы, правильная версия, следующий вопрос.
Не хвали без причины.
```

---

## Track 09 — Bitmap / Inverted Index / Audience Serving

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: Bitmap indexes и inverted indexes для audience serving — RoaringBitmap, groupBitmapAnd в ClickHouse, use cases и ограничения.

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes.

Цель:
Написать Kotlin-проект с RoaringBitmap для 10M client_id. Замерить intersect vs CH-native groupBitmapAnd. Понять, когда bitmap проигрывает обычному SQL.

Подбери ресурсы по системе (max 5–7), с порядком изучения и DoD.
```

### Шаг 3 — Deep Explanation (если тема «RoaringBitmap internals»)
```
Объясни тему: RoaringBitmap — внутренняя структура (array containers, bitmap containers, run containers), когда какой используется, сериализация.

Формат:
1. Простое объяснение в 5 предложениях
2. Ментальная модель
3. Как это работает технически
4. Где это ломается
5. Пример из backend (audience segments)
6. Пример на Kotlin/Java
7. Типовые ошибки
8. 10 вопросов для проверки
9. Мини-практика на 30 минут

Не пиши водно.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: Bitmap indexes — RoaringBitmap internals, groupBitmapAnd в CH, range predicates, когда bitmap проигрывает SQL.

Задавай вопросы по одному (простой → кейс → trade-off → senior-уровень).
После каждого ответа: оценка 1–10, пробелы, правильная версия, следующий вопрос.
Не хвали без причины.
```

---

## Track 10 — Cache / Incremental / CDC

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: Caching patterns для вычислений — cache-aside, write-through, single-flight, TTL, cache stampede. Incremental processing и CDC (Change Data Capture).

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes.

Цель:
Написать hash-функцию для AuditoryParams (нормализация + sha256) и замерить потенциальный cache-hit rate. Спроектировать single-flight + TTL cache для audience results.

Подбери ресурсы по системе (max 5–7), с порядком изучения и DoD.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: caching — cache stampede, single-flight, нормализация для cache keys, incremental vs full recompute, CDC для cache invalidation.

Задавай вопросы по одному (простой → кейс → trade-off → senior-уровень).
После каждого ответа: оценка 1–10, пробелы, правильная версия, следующий вопрос.
Не хвали без причины.
```

---

## Track 11 — JVM + Go Performance Profiling

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: Performance profiling — Go pprof (cpu, heap, goroutine, trace), race detector, escape analysis, benchmarks с -benchmem. JVM: async-profiler, JFR, G1GC, safepoint bias.

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes.

Цель:
Снять flame graph для A-P (Go) через pprof. Найти top-3 CPU + top-3 alloc hot spots. Снять профиль Target (JVM) через async-profiler. Написать benchmark с -benchmem.

Подбери ресурсы по системе (max 5–7), с порядком изучения и DoD.
```

### Шаг 3 — Deep Explanation (если тема «Go escape analysis»)
```
Объясни тему: Go escape analysis — что уходит на heap, что остаётся на stack, как читать -gcflags="-m" output, как влияет на GC pressure.

Формат:
1. Простое объяснение в 5 предложениях
2. Ментальная модель
3. Как это работает технически
4. Где это ломается
5. Пример из backend/microservices
6. Пример на Go
7. Типовые ошибки
8. 10 вопросов для проверки
9. Мини-практика на 30 минут

Не пиши водно.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: performance profiling — Go pprof flame graph, escape analysis, -race, benchmarks. JVM async-profiler, safepoint bias, G1GC pause analysis.

Задавай вопросы по одному (простой → кейс → trade-off → senior-уровень).
После каждого ответа: оценка 1–10, пробелы, правильная версия, следующий вопрос.
Не хвали без причины.
```

---

## Track 12 — Business / Product / Leadership

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: Engineering leadership — stakeholder management, positioning for internal RFC, strategy kernel (diagnosis + policy + actions), discovery conversations.

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes. Хочу переходить к staff/principal level.

Цель:
Провести 3 discovery-разговора с PM/тимлидом/downstream-командами за квартал. Построить stakeholder map для RFC. Написать strategy kernel для RFC (1 страница).

Подбери ресурсы по системе (max 5–7), с порядком изучения и DoD.
```

### Шаг 3 — Deep Explanation (если тема «stakeholder management для RFC»)
```
Объясни тему: Stakeholder management при продвижении технических инициатив (RFC, архитектурные изменения) — как выявить stakeholders, понять их интересы, снять возражения.

Формат:
1. Простое объяснение в 5 предложениях
2. Ментальная модель
3. Как это работает на практике
4. Где это ломается
5. Пример из backend/engineering (RFC approval)
6. Конкретный шаблон для discovery-разговора
7. Типовые ошибки
8. 10 вопросов для проверки
9. Мини-практика на 30 минут

Не пиши водно.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: stakeholder management — discovery conversations, sphere of influence, strategy kernel, positioning RFC для технических изменений.

Задавай вопросы по одному (простой → кейс → trade-off → senior-уровень).
После каждого ответа: оценка 1–10, пробелы, правильная версия, следующий вопрос.
Не хвали без причины.
```

---

## Track 13 — Networks

### Шаг 1 — Learning Curator
```
Ты мой learning curator.

Тема: Network protocols для backend engineers — TCP/IP (3-way handshake, TIME_WAIT, keepalive, TCP_NODELAY), HTTP/2 vs HTTP/1.1, gRPC frames, Kafka protocol (acks, ISR, rebalance), DNS TTL в JVM, TLS handshake.

Мой уровень:
Backend engineer, Kotlin/Java/Go, работаю с микросервисами, Kafka, ClickHouse, Postgres, Kubernetes.

Цель:
Увидеть gRPC в Wireshark (HTTP/2 frames + Protobuf payload). Замерить Kafka latency при acks=0/1/all. Диагностировать pool exhaustion в production.

Подбери ресурсы по системе (max 5–7), с порядком изучения и DoD.
```

### Шаг 4 — Self-check
```
Проверь, понял ли я тему: networking — TCP 3-way handshake, TIME_WAIT, gRPC over HTTP/2, Kafka acks и ISR, DNS TTL в JVM, connection pool exhaustion.

Задавай вопросы по одному (простой → кейс → trade-off → senior-уровень).
После каждого ответа: оценка 1–10, пробелы, правильная версия, следующий вопрос.
Не хвали без причины.
```

---

## Sprint-level промпты

### Sprint 01 — A-P End-to-End Map

#### Шаг 2 — Weekly Sprint
```
Составь мне weekly learning sprint.

Тема недели: A-P End-to-End Map — полный путь аудитории от HTTP-запроса до доставки в downstream.

Цель недели:
Нарисовать mermaid sequence diagram полного lifecycle аудитории с file:line ссылками на реальный код. Запустить A-P локально и создать тестовую аудиторию через curl.

Мой контекст:
Backend engineer, Kotlin/Java/Go, микросервисы, Kafka, ClickHouse, Postgres, K8s.

Ограничения:
- В день 60–90 минут.
- Каждый день должен заканчиваться артефактом.
- Не больше 1–2 ресурсов в день.
- Должна быть практика руками.
- В конце недели — итоговый sequence diagram.

Формат: Понедельник: Цель / Ресурсы / Практика / Артефакт / DoD. И так на 7 дней.
```

### Sprint 03 — Compute/Export/Delivery Numbers

#### Шаг 2 — Weekly Sprint
```
Составь мне weekly learning sprint.

Тема недели: Измерение времени end-to-end для audience pipeline — compute / export / delivery breakdown.

Цель недели:
Собрать реальные числа: сколько занимает каждый слой для аудиторий размером 10k / 1M / 10M. Заполнить таблицу p50/p95 с реальными данными.

Мой контекст:
Backend engineer, Kotlin/Java/Go, микросервисы, Kafka, ClickHouse, Postgres, K8s.

Ограничения: 60–90 минут в день. Артефакт каждый день. Практика руками.

Формат: Понедельник: Цель / Ресурсы / Практика / Артефакт / DoD. И так на 7 дней.
```

### Sprint 04 — Graph vs Alternatives

#### Шаг 2 — Weekly Sprint
```
Составь мне weekly learning sprint.

Тема недели: Сравнение DAG-подхода (Target TQ) vs one-SQL ClickHouse vs cache для одного конкретного use-case.

Цель недели:
Заполнить decision table реальными числами для одной конкретной аудитории. Замерить DAG baseline vs one-SQL wall clock 5 раз (p50/p95).

Мой контекст:
Backend engineer, Kotlin/Java/Go, микросервисы, Kafka, ClickHouse, Postgres, K8s.

Ограничения: 60–90 минут в день. Артефакт каждый день. Числовой вывод обязателен.

Формат: Понедельник: Цель / Ресурсы / Практика / Артефакт / DoD. И так на 7 дней.
```

### Sprint 08 — Bitmap Experiment

#### Шаг 2 — Weekly Sprint
```
Составь мне weekly learning sprint.

Тема недели: Bitmap experiment — RoaringBitmap vs CH-native groupBitmapAnd для intersection 10M client_id.

Цель недели:
Написать Kotlin benchmark: RoaringBitmap для 10M точек, 100 сегментов. Сравнить intersect time vs CH groupBitmapAnd. Числовой вывод.

Мой контекст:
Backend engineer, Kotlin/Java/Go, микросервисы, Kafka, ClickHouse, Postgres, K8s.

Ограничения: 60–90 минут в день. Без числового вывода — не считается выполненным.

Формат: Понедельник: Цель / Ресурсы / Практика / Артефакт / DoD. И так на 7 дней.
```

---

## Универсальный промпт Self-check перед RFC

```
Проверь, готов ли я писать RFC по теме: оптимизация audience building pipeline (A-P / Target / TQ).

Контекст:
- A-P (Go): orchestrates audience lifecycle, calls Target via REST, consumes Kafka results
- Target (Kotlin/Spring): builds audience via ClickHouse queries using TQ task graph  
- TQ: Kotlin Coroutines + PostgreSQL SKIP LOCKED task scheduler

Задавай вопросы по одному в порядке: Domain → Architecture → TQ internals → Performance → Rollout/Risk.

Формат каждого вопроса:
- сначала простой вопрос
- потом практический кейс
- потом вопрос на trade-offs
- потом вопрос уровня senior

После каждого моего ответа:
1. Оценка 1–10
2. Пробелы
3. Правильная версия
4. Следующий вопрос

Если я не могу ответить на 30%+ — скажи это явно и укажи, какой спринт/трек нужно пройти.
```
