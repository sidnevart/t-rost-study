# 01. Обзор трёх проектов

> Источник фактов: `work_projects/a-p`, `work_projects/target`, `t-segment-projects/ninja-t-queue`. Если поле помечено «гипотеза» — значит, не подтверждено напрямую кодом, требует подтверждения у команды.

---

## A-P / Auditory Platform (`work_projects/a-p`)

### Что делает (по коду)
- Сервис управления **транзакционными и чековыми аудиториями** в T-Банке.
- Принимает HTTP-запросы на создание/обновление/проливку/продление аудиторий.
- Хранит метаданные аудиторий и результаты в PostgreSQL.
- Отправляет аудиторию в **T-Segment** для серверного использования (CSV в S3 + Kafka событие).
- Координируется с **Target** через REST + Kafka (Target собирает аудиторию по параметрам и возвращает результат).

### Стек (`README.md:46-55`, `go.mod`)
```text
Язык       : Go 1.23.1
Web        : Chi router (`internal/api`)
Контракты  : OpenAPI/TypeSpec → oapi-codegen + Avro (`contract/`)
БД         : PostgreSQL (Liquibase миграции в `migrations/changelog/`)
Очереди    : Kafka (Sage), Producer/Consumer в `internal/delivery/kafka/`
Хранилище  : S3 (`internal/lib/s3/`, `internal/client/s3_client.go`)
gRPC       : SegmentStorage, TimeService (`internal/client/pb/`)
Логи       : Sage / slog (`internal/lib/logger/`)
Метрики    : Prometheus (см. `application/`)
Деплой     : Docker, K8s, GitLab CI
```

### Модули верхнего уровня (`internal/`)
| Модуль | Роль |
|---|---|
| `api/` | HTTP контроллеры (auditory, gateway, tsegment, mocks) |
| `application/` | Сборка приложения: container, cron, kafka, public/private server |
| `client/` | Внешние HTTP/gRPC клиенты: anon, target, segment_storage, time, s3, tsegment |
| `db/` | postgres + transactor |
| `delivery/` | HTTP сервисы + Kafka producers/consumers |
| `domain/` | модели, dto, avro, errors |
| `lib/` | jwt, kafka, logger, s3, scheduler/locker |
| `repository/` | postgres-репозитории по сущностям |
| `service/` | бизнес-сервисы: gathering, loading, sdp, sending, sizecount, tsegment |

### Ключевые файлы
1. `internal/domain/model/auditory.go` — модель `Auditory`, статусы, типы (STATIC/DYNAMIC), `AuditoryParams` (большой DSL: client/terminal/MCC/geo/segment/finance).
2. `internal/api/auditory/controller.go` — HTTP-контроллер аудиторий.
3. `internal/repository/auditory_repository.go` — основной CRUD по таблице `auditory`.
4. `internal/service/gathering/auditory_gathering_service.go` — старт сборки (взаимодействие с Target).
5. `internal/delivery/kafka/target/auditory/consumer.go` — приём результата от Target.
6. `internal/service/sdp/auditory_params_sdp_service.go` — отправка параметров аудитории в SDP.
7. `internal/service/t_segment_exporting_service.go` — экспорт в T-Segment.
8. `internal/application/cron.go` — все cron-джобы (size count, dynamic gathering, TTL, partitioner и т.п.).
9. `migrations/changelog/` — все таблицы PG.
10. `contract/avro/auditory-export.avsc` + `docs/components/kafka/nfs-partner-platforms.internal.auditory.mdx` — Kafka-контракт с Target.

### Cron-джобы (`docs/components/job/`)
```
AuditoryParamsSdpSendingJob          — отправка параметров в SDP
DeleteAuditoryTTLJob                 — чистка просроченных аудиторий
LoadAuditoryRequestProcessingJob     — обработка запросов на проливку
RunDynamicAuditoryGathering          — повторная сборка динамических аудиторий
SegmentSizeCountJob                  — асинхронный подсчёт размера сегмента
TSegmentUploadMetricsSendingJob      — метрики выгрузки
TSegmentValueFilePartitionerJob      — разделение файлов на партиции
TSegmentValueVerificationJob         — верификация значений фильтров
```

### Открытые вопросы по A-P
- Где именно начинается сборка: cron + поллинг или kafka-event? `RunDynamicAuditoryGathering` запускает повторную сборку динамических — а старт первой?
- Как `gathering_id` связывает запрос на сборку и результат от Target?
- Какие есть дублирующие пути компиляции аудитории (быстрые vs медленные)?

---

## Target (`work_projects/target`)

### Что делает (по коду)
- Kotlin/Spring-приложение, центральная **compute-машина** для аудиторий разных каналов (PARTNER, CB platform, Extra Cashback, Offer и т.п.).
- Использует **TQ** как библиотеку для оркестрации задач сборки.
- Получает запросы из A-P (REST + Kafka), запускает сбор аудитории, выгружает CSV в S3, шлёт результат в A-P через Kafka-топик `nfs-partner-platforms.internal.auditory`.
- Параллельно работает с целым семейством Kafka-топиков для разных потребителей: `cbp.auditory`, `ms.auditory.extra-cashback`, `ms.manual-badge-fire-auditory` и т.д.

### Стек (`build.gradle.kts`, `application/infra/src/main/resources/application.yaml`)
```text
Язык        : Kotlin (2514 файлов; Java/Python — почти нет)
Фреймворк   : Spring Boot
Сборка      : Gradle (multi-module: 21 модуль)
Реактивщина : WebClient (Reactor)
БД          : PostgreSQL (Liquibase в `application/infra/src/main/resources/db/`)
Очереди     : Kafka, Avro (схемы — `contract/avro/`)
Хранилище   : S3 (несколько бакетов: ms-auditory, target-auditory, target-load-auditory, target-gather-auditory, target-send-auditory)
Оркестрация : TQ (`ru.tinkoff.ms.ninja:tqueue`)
gRPC        : `contract/grpc-api/proto/AuditoryLoadingService.proto`, `ExternalAuditoryGatheringService.proto`, `PartnerOfferAuditoryService.proto`
Auth A-P    : JWT HS512, `AuditoryPlatformApi`
```

### Модули (`application/`)
```
age/                          
auditory-distribution/        
auditory-loading/  (api/core) — TQ-задачи на проливку
common/                       
data-replication/             
infra/                        — Spring config, Kafka, DB, дашборды
precamp/                      
target-auditory/  (api/core)  — REST API + интеграция с A-P
```

### Типы TQ-задач Target (`auditory-loading-api/.../AuditoryLoadingTQTaskType.kt`)
```text
TARGET_AUDITORY_LOADING
TARGET_AUDITORY_EVENT_LOADING
CB_PLATFORM_AUDITORY_LOADING
EXTRA_CB_AUDITORY_LOADING
UNKNOWN_CLIENT_AVAILABILITY_SENDING
STANDARD_AUDITORY_LOADING
OFFER_AUDITORY_LOADING
MATCHING_EXCLUSION
LOADING_EXCLUSION
MATCHING_DELTA_EXCLUSION
SENDING_EXCLUSION
```

### Ключевые файлы
1. `target-auditory-core/.../entrypoint/rest/TargetAuditoryControllerV2.kt` — REST API.
2. `target-auditory-core/.../service/api/AuditoryPlatformApi.kt` — клиент к A-P (WebClient + JWT HS512, exp 1h).
3. `auditory-loading-api/.../tq/api/AuditoryLoadingTQTaskBuilder.kt` — DSL построения графа задач.
4. `auditory-loading-core/.../gathering/job/DynamicOfferAuditoryGatheringJob.kt` — динамический сбор офферов.
5. `auditory-loading-core/.../auditoryExport/AuditoryExportBuilder.kt` — формирование экспортного сообщения.
6. `infra/src/main/resources/application.yaml` — все Kafka-топики, S3, тайминги.
7. `contract/avro/auditory-export.avsc` — формат экспорта аудитории.
8. `contract/grpc-api/proto/ExternalAuditoryGatheringService.proto` — gRPC-входы для внешних сборщиков.

### Открытые вопросы по Target
- Какие именно TQ-задачи бывают «длинными» (минуты/часы)?
- Где именно решение «полный пересбор vs delta» (`MATCHING_DELTA_EXCLUSION`, `isDiff` в kafka-сообщении)?
- Сколько в среднем задач в одном процессе (DAG-ширина)?
- Какова реальная нагрузка по очередям TQ (queueLimits в config)?

---

## TQ / ninja-t-queue (`t-segment-projects/ninja-t-queue`)

### Что делает (по коду)
- **Kotlin-библиотека**, не отдельный сервис. Подключается как зависимость (`implementation("ru.tinkoff.ms.ninja:tqueue:{Version}")`).
- Реализует асинхронную очередь задач **поверх PostgreSQL**: процессы (DAG из задач), приоритеты, зависимости, retry, heartbeat, координация нод.
- Используется Target (и, по `docs/intro.md`, потенциально другими сервисами).

### Стек (`build.gradle.kts`, `docs/intro.md`)
```text
Язык       : Kotlin (Coroutines)
DI         : Spring Boot starter (`application/starter/spring`)
БД         : PostgreSQL (JDBI) — `application/engine/storage/postgresql/`
Миграции   : Liquibase (`db/changelog/001..016`)
Расширения : Recurring tasks (cron), Heartbeat, Cleanup, Continuous processes
Frontend   : `tq-frontend/` (наблюдение)
```

### Структура
```
application/
  api/common-api/        — TQ, TQConfig, TQTask, DSL (TQTaskDsl), TQRegistry, ExceptionHandler
  engine/
    common/              — TQPersister, TQApiService, TQProcessExceptionProcessor
    scheduler/
      core/              — TQScheduler (главный цикл)
      heartbeat/         — TQHeartbeatService, TQNodeHeartbeat
    storage/
      api/               — интерфейсы TQTaskRepository, TQProcessRepository, TQNodeRepository, ...
      postgresql/        — JDBI-реализации (TQJdbiTaskDao и т.п.)
  starter/
    simple/              — ручная инициализация
    spring/              — Spring-стартер (RecurringWorkerPreProcessor)
  utils/                 — SerializationUtils, GenericUtils
test/                    — модульные/интеграционные тесты
```

### Таблицы PG (`db/changelog/`)
```
tq_task               — задача (id UUID, type, status, params/result JSONB, priority, retry_count, dates, queue_type, process_id, node_id, date_reserved)
tq_task_dependencies  — DAG (task_id, dependency_task_id, оба PK + FK на tq_task)
tq_process            — процесс (id, status, params, priority, dates, type, continuous BOOL)
tq_recurring_task     — повторяющиеся задачи (cron)
tq_recurring_task_run — журнал запусков recurring
tq_node               — heartbeat нод (id UUID, live_until)
```

### Ключевые механики
- **Резервация задач**: `SELECT ... FROM tq_task WHERE status='NEW' AND node_id IS NULL ORDER BY priority, date_created LIMIT N FOR UPDATE SKIP LOCKED` (`TQJdbiTaskDao.kt:181-182`).
- **Continuous processes** (`013_add_continuous_to_tq_process.xml`, `docs/API/core-processes/continuous-processes.md`): после старта первой задачи остальные NEW-задачи получают priority=-1 — чтобы DAG не «зависал» между шагами.
- **Stale recovery**: `markTimedOutStaleNewTasks` (24h NEW), `releaseStaleAssignedTasks` (5min reservation).
- **Heartbeat**: каждая нода каждые 15с пишет `live_until = now() + 30s` в `tq_node`. Мёртвые ноды → задачи освобождаются.
- **Партиальные индексы** (`014_add_new_indices_on_database.xml`): отдельные индексы под `status='NEW' AND node_id IS NULL`, `status='IN_PROGRESS'`, `node_id IS NOT NULL` и т.д. — оптимизировано под FOR UPDATE SKIP LOCKED.
- **TQConfig defaults** (`docs/API/configuration/TQConfig.md`):
```text
interval = 5s        — итерация шедулера
maxConcurrentTasks   = 5
taskFetchLimit       = 15
taskExecutionTimeout = 10min  → IN_PROGRESS дольше → TIMEOUT
taskStaleNewTimeout  = 24h    → NEW без ноды дольше → TIMEOUT
taskReservationTimeout = 5min → NEW + node_id, не стало IN_PROGRESS → возврат в очередь
maxRetryCount        = 1
heartbeatInterval/TTL = 15s/30s
```

### Ключевые файлы
1. `application/api/common-api/.../main/TQ.kt` — главный entry point.
2. `application/api/common-api/.../main/TQTaskDsl.kt` — `tqueue { task("X") {...} }` DSL.
3. `application/api/common-api/.../configuration/TQConfig.kt` — все настройки.
4. `application/engine/scheduler/core/.../TQScheduler.kt` — главный цикл шедулинга.
5. `application/engine/storage/postgresql/.../task/TQJdbiTaskDao.kt` — все SQL по задачам, в т.ч. `getAvailableTasks` с SKIP LOCKED.
6. `application/engine/scheduler/heartbeat/.../TQHeartbeatService.kt` — heartbeat / детект мёртвых нод.
7. `application/engine/common/.../TQPersister.kt` — запись результатов.
8. `application/engine/common/.../TQProcessExceptionProcessor.kt` — обработка ошибок процесса.
9. `db/changelog/014_add_new_indices_on_database.xml` — индексы.
10. `docs/API/core-processes/continuous-processes.md` — описание continuous-режима.

### Открытые вопросы по TQ
- Что происходит, если задача отвалилась после `IN_PROGRESS` без update в БД, но нода жива? (только `taskExecutionTimeout` вернёт?)
- Как ведёт себя retry при `maxRetryCount=1`? Это ровно одна попытка повтора, или одна попытка вообще?
- Какова латентность одного hop между задачами в DAG? (interval=5s, значит минимум ~5с межшаговая задержка — кроме `continuous=true`).
- Есть ли priority inversion риск, если queueLimits заполнены долгими задачами?

---

## Что в каждом проекте остаётся неясно (общее)

| Область | Вопрос |
|---|---|
| Контракт A-P↔Target | Все ли интеграции через REST+Kafka, или есть gRPC? Точный набор полей в `nfs-partner-platforms.internal.auditory`. |
| Размер аудиторий | Какие p50/p95/max по числу client_id в одной аудитории? |
| Тайминги | Сколько компонент времени уходит на compute, export, S3 upload, t-segment.offline-upload.upload, доставку? |
| SLA | Видны ли где-то цифры SLA (timeout, deadline, max_age) кроме TQ-defaults? |
| Хранение результата | Где «единый» registry результатов? `auditory.s3Key` + `t_segment_auditory.s3_key` — это два места, насколько они согласованы? |
| Графы DAG | Реальная глубина и ширина DAG в продакшене Target? |
| Recurring | Какие recurring-задачи реально живут в `tq_recurring_task`? |
| Идемпотентность | Где гарантируется идемпотентность: `messageId` в `auto_segment_loading_message`, или ниже? |

→ см. подробнее [07-open-questions-for-team.md](07-open-questions-for-team.md).
