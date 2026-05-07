# Track 02 — Target Integration

## Зачем учить
Контракт A-P↔Target — самая хрупкая часть системы. Любое архитектурное изменение проходит через него. Без полного понимания — RFC сломается на code review.

## Где это есть в проектах
- A-P, `internal/client/target/apiclient.gen.go` (oapi-codegen) — REST клиент.
- A-P, `internal/delivery/kafka/target/auditory/consumer.go` — consumer результата.
- Target, `application/target-auditory/target-auditory-core/.../service/api/AuditoryPlatformApi.kt` — клиент к A-P (WebClient + JWT HS512).
- Target, `application/auditory-loading/auditory-loading-api/.../tq/api/AuditoryLoadingTQTaskBuilder.kt` — DSL построения графа задач.
- Target, `contract/avro/auditory-export.avsc` — Avro контракт.
- A-P, `docs/components/kafka/nfs-partner-platforms.internal.auditory.mdx` — описание интеграции.

## Что изучить
```text
1. Все Avro-схемы: auditory.avsc, auditory-event.avsc, auditory-export.avsc, target-auditory.avsc.
2. JWT-механизм между A-P и Target (claim "login", HS512, exp=1h).
3. REST-эндпоинты в обе стороны.
4. Все Kafka-топики: in/out, key/value, partitioning.
5. Семантика error в kafka сообщении (что именно вызывает FAILED).
6. Переходы isDiff=true/false и кто их триггерит.
7. Idempotency expectations (явные и неявные).
```

## GPT-ускоритель

**Шаг 1 — Learning Curator** → вставь из `gpt-prompts.md` → Track 02 → Шаг 1.
**Шаг 3 — Deep Explanation** (если застрял на Avro schema evolution) → вставь из `gpt-prompts.md` → Track 02 → Шаг 3.
**Шаг 4 — Self-check** → вставь из `gpt-prompts.md` → Track 02 → Шаг 4.

---

## Практика на проекте

### Задача 1 — REST round-trip: A-P → Target (обязательно)
- Локально поднять Kafka + A-P + Target (через docker-compose или testcontainers).
- Создать аудиторию через curl → POST `/auditory/{id}/loading`.
- В логах найти: A-P делает HTTP вызов в Target → что именно передаёт.
- Зафиксировать: какой endpoint Target, какое тело, какой JWT header.

### Задача 2 — Avro-схема: inspect + BACKWARD check
- Через `avro tools` или python `fastavro` выгрузить `auditory-export.avsc`.
- Сделать копию схемы, добавить новое поле с default — проверить BACKWARD совместимость.
- Попробовать добавить поле БЕЗ default — убедиться что это breaking change.
- Написать: «можно / нельзя» для 5 типов изменений схемы.

### Задача 3 — Kafka topics: карта producer/consumer
- Найти все Kafka топики между A-P и Target (grep по кодовой базе).
- Для каждого топика — найти: кто producer, кто consumer, формат (Avro/JSON), партиционирование, acks.
- Заполнить таблицу: `topic | producer | consumer | format | key | acks | semantics`.

### Задача 4 — 5 провокационных сценариев (fault injection)
Для каждого — трассировать по коду, что произойдёт:
1. Target не ответил на HTTP за 60 секунд
2. Target вернул ошибку (HTTP 500)
3. Kafka consumer A-P получил то же `gathering_id` дважды
4. `error != null` в Avro сообщении — как A-P обрабатывает?
5. Target отправил результат с `gathering_id`, которого нет в `load_auditory_request`

Для каждого: найти обработку в коде (file:line) или зафиксировать «не обрабатывается».

### Задача 5 — Dedup: есть или нет?
- Найти в A-P consumer код, который обрабатывает повторное Kafka сообщение.
- Написать SQL: «что произойдёт в БД если consumer обработает одно сообщение дважды».
- Вывод: идемпотентен ли consumer? Если нет — что надо добавить?

### Задача 6 — JWT декодирование
- Найти в коде Target, где генерируется JWT для вызовов к A-P.
- Декодировать токен вручную (jwt.io или `base64 -d`): увидеть claims, exp, алгоритм.
- Ответить: что будет если `exp` прошёл во время долгой обработки аудитории?

## Артефакты
- Заполнить `02-architecture/02-target-contract.md` своими наблюдениями.
- `practice/track-02-sequence.md` — sequence diagram (mermaid) одной полной интеграции с file:line.
- `practice/track-02-fault-scenarios.md` — 5 сценариев с трассировкой по коду.
- `practice/track-02-kafka-topics.md` — таблица топиков.

## Self-check
1. Если Target не отвечает 60 секунд — что делает A-P?
2. Как dedup-ить kafka сообщение в `nfs-partner-platforms.internal.auditory`?
3. Где собственно код, разбирающий Avro в A-P?
4. Кто и как генерирует `gathering_id`, который связывает request и result?
5. Почему JWT, а не mTLS / OAuth?
6. Что произойдёт если добавить поле без default в `auditory-export.avsc`?
7. Идемпотентен ли A-P consumer при повторном сообщении?
8. Какие 3 самых критичных инварианта контракта, нарушение которых ломает pipeline?

## DoD
```text
[ ] Могу за 3 минуты объяснить контракт «A-P → Target → A-P → T-Segment».
[ ] Воспроизвёл локально один полный round-trip (или описал блокер).
[ ] Нашёл / исключил dedup-механизм (точка в коде).
[ ] Заполнил таблицу Kafka-топиков.
[ ] 5 fault-сценариев трассированы по коду.
```
