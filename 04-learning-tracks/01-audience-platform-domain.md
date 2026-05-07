# Track 01 — Audience Platform Domain

## Зачем учить
Без модели домена все технические улучшения промахиваются. Я должен уметь нарисовать на доске «что такое аудитория, какой её lifecycle и какие типы фильтров бывают» без подсказки.

## Где это есть в проектах
- A-P, `internal/domain/model/auditory.go:16-209` — Auditory + AuditoryParams (DSL).
- A-P, миграции `migrations/changelog/*.xml` — таблицы домена.
- A-P, `internal/api/auditory/controller.go` — публичные операции над доменом.
- A-P, `docs/processes/` — процессы DLH, SDP, t-segment upload.
- Target, `application/auditory-loading/auditory-loading-api/.../tq/model/process/AuditoryLoadingTQProcessType.kt` — типы процессов сборки.

## Что изучить
```text
1. Структуру AuditoryParams и каждый из 9+ типов фильтров.
2. Audience lifecycle (DRAFT/GATHERING/REGATHERING/GATHERED/ERROR/ARCHIVED).
3. Различие STATIC vs DYNAMIC.
4. Виды Showcase (VYGODA, EXTRA_CASHBACK) и TargetType.
5. Связку auditory_id ↔ gathering_id ↔ результат в S3.
6. Точку «начало проливки»: load_auditory_request + LoadAuditoryRequestProcessingJob.
7. Точку «возврат результата»: kafka consumer на nfs-partner-platforms.internal.auditory.
```

## GPT-ускоритель

**Шаг 1 — Learning Curator** → вставь из `gpt-prompts.md` → Track 01 → Шаг 1.
**Шаг 4 — Self-check** → вставь из `gpt-prompts.md` → Track 01 → Шаг 4.

---

## Практика на проекте

### Задача 1 — REST + БД (обязательно)
- Создать локально 1 STATIC аудиторию через curl: `POST /auditory` с минимальным телом.
- Создать 1 DYNAMIC аудиторию с фильтром по городу + возрасту.
- В psql проследить переход статусов: `SELECT id, status, gathering_id, updated_at FROM auditory ORDER BY updated_at DESC LIMIT 5;`
- Записать, в каком файле/строке меняется `status = GATHERING`.

### Задача 2 — Lifecycle state machine (код + схема)
- Написать state machine на бумаге/Mermaid: все возможные переходы статусов Auditory.
- Для каждого перехода — найти строку кода в A-P, которая его вызывает.
- Найти переход, который не очевиден из кода (нужно читать тест или доку).

### Задача 3 — AuditoryParams DSL → SQL трансляция
- Найти в коде A-P место, где `AuditoryParams` транслируется в условие запроса к ClickHouse или Postgres.
- Написать 3 примера: простой predicate (city), составной (city + age), с S3-reference (segment file).
- Для каждого — нарисовать, как он превращается в SQL/CH WHERE-условие.
- Фиксируй: что случится если добавить новый тип фильтра — где нужно изменить код?

### Задача 4 — Kafka consumer: trace от сообщения до БД
- Найти Kafka consumer для `nfs-partner-platforms.internal.auditory`.
- Через код пройти путь: сообщение → Avro-десериализация → обновление статуса → запись в S3.
- Поставить `log.Info` в ключевые точки и прогнать тестовый сценарий.
- Найти: есть ли dedup-логика? Если нет — что произойдёт при повторном сообщении?

### Задача 5 — REGATHERING: когда и почему (исследование)
- Найти все места в коде, где аудитория переходит в `REGATHERING`.
- Написать список условий: «переходит в REGATHERING если...»
- Проверить гипотезу: DYNAMIC всегда может уйти в REGATHERING, STATIC — нет?

## Артефакты
- `practice/track-01-anatomy.md` — 1 страница: «Anatomy of an Auditory» (модель + lifecycle mermaid + flow).
- `practice/track-01-state-machine.md` — таблица переходов с file:line для каждого.
- Заполнить раздел «Личные заметки» в `01-context/02-ap-domain-map.md`.

## Self-check
1. Чем STATIC отличается от DYNAMIC по переходу статусов?
2. Кто пишет в `nfs-partner-platforms.internal.auditory`?
3. Почему `gathering_id` отдельно от `auditory_id`?
4. Что происходит с CSV в S3 при пересборке DYNAMIC аудитории — переписывается или версионируется?
5. Какие из AuditoryParams — predicates, какие — references на S3?
6. В каком файле строка `status = GATHERING`? Что вызывает этот переход?
7. Что произойдёт, если Kafka consumer получит одно и то же сообщение дважды?
8. Как добавить новый тип фильтра в AuditoryParams — где изменить код?

## DoD
```text
[ ] Могу нарисовать модель домена и lifecycle на доске за 5 минут.
[ ] Знаю file:line каждого перехода статусов.
[ ] Нашёл или исключил dedup-механизм (точка в коде).
[ ] Написал 3 примера трансляции AuditoryParams → SQL/CH.
[ ] Заметки в 01-context/02-ap-domain-map.md заполнены.
```
