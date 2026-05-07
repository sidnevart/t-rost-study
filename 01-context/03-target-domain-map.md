# 03. Target Domain Map

> Target — Kotlin/Spring multi-module приложение (21 модуль). Главный наблюдаемый факт: Target действительно **исполняет сборку** и **использует TQ как библиотеку**. Это отличает его от роли «потребителя аудиторий», которая часто угадывается по названию.

---

## 3.1 Что Target принимает на вход

### REST (`target-auditory-core/.../entrypoint/rest/`)
- `TargetAuditoryController` (V1) и `TargetAuditoryControllerV2` — admin-эндпоинты для аудиторий.
- Принимают команды на запуск проливки, изменения, исключения, отгрузки.

### Kafka (`infra/src/main/resources/application.yaml`)
Входящие топики:
```text
ms.auditory                              — события платформы
ms.external-auditory.response            — ответы внешних сборщиков
ms.auditory.extra-cashback               — extra-cashback-аудитории
cbp.auditory                             — cb-platform-аудитории
ms.manual-badge-fire-auditory            — мануальные триггеры
ms.auditory-exclusion-events             — события исключений
ms.auditory-modification-events          — модификации
```

### gRPC (`contract/grpc-api/proto/`)
- `AuditoryLoadingService.proto` — загрузка/доставка.
- `ExternalAuditoryGatheringService.proto` — внешний триггер сборки.
- `PartnerOfferAuditoryService.proto` — партнёрские офферы.

---

## 3.2 Что Target делает (high-level)

```text
Запрос на сборку (REST/Kafka/gRPC)
   ↓
Сервис создаёт TQ-процесс (TQProcessType) с DAG задач (TQTaskType)
   ↓
TQ-задачи бегут шагами:
   - GATHER (параметры → SQL/CH запрос или внешний вызов)
   - PROCESS (intersect/exclude, matching, delta)
   - EXPORT (S3-чанки)
   - SEND (Kafka сообщение в A-P / в специфичные кампании)
   ↓
Финал: producer пишет в `nfs-partner-platforms.internal.auditory` (для A-P)
       и/или в каналы аудиторий (cbp.auditory, ms.auditory.extra-cashback и т.п.)
```

Подтверждается:
- `auditory-loading-api/.../tq/api/AuditoryLoadingTQTaskBuilder.kt` — DSL построения графа.
- `auditory-loading-api/.../tq/model/task/AuditoryLoadingTQTaskType.kt` — типы задач (см. ниже).
- `auditory-loading-core/.../auditoryExport/AuditoryExportBuilder.kt` — формирование выходного сообщения.

---

## 3.3 Типы TQ-задач Target

`AuditoryLoadingTQTaskType.kt`:
```text
TARGET_AUDITORY_LOADING               — стандартная проливка в Target
TARGET_AUDITORY_EVENT_LOADING         — событийная проливка
CB_PLATFORM_AUDITORY_LOADING          — cb-платформа
EXTRA_CB_AUDITORY_LOADING             — extra cashback
UNKNOWN_CLIENT_AVAILABILITY_SENDING   — отдельный поток для «неизвестных клиентов»
STANDARD_AUDITORY_LOADING             — общий путь
OFFER_AUDITORY_LOADING                — оффер-аудитории (партнёрские)
MATCHING_EXCLUSION                    — матчинг исключений
LOADING_EXCLUSION                     — проливка исключений
MATCHING_DELTA_EXCLUSION              — delta-матчинг (incremental)
SENDING_EXCLUSION                     — отправка исключений
```

### Что отсюда читается
- Есть **разные типы аудиторий** с разной логикой: standard/offer/cb-platform/extra-cb.
- Есть отдельный путь под **delta** (`MATCHING_DELTA_EXCLUSION`) — то есть incremental уже частично существует.
- Есть отдельная очередь под Kafka-bound задачи: `TQQueueType.KAFKA` (см. `AuditoryLoadingTQTaskBuilder.kt:18, 23, 30`).

---

## 3.4 Контракт Target → A-P (что A-P принимает)

### Kafka topic `nfs-partner-platforms.internal.auditory`
(`a-p/docs/components/kafka/nfs-partner-platforms.internal.auditory.mdx`)

```text
Avro-сообщение со следующей семантикой:
  - error: nullable. Если != null → A-P ставит статус FAILED и шлёт в Time.
  - gathering_id: id запуска сборки.
  - результат сборки: ссылки на S3-партиции (CSV) с client_id.
  - дополнительные поля: isDiff, dataDateTime.

A-P, получая это:
  1. Если error != null → status=FAILED, нотификация.
  2. Иначе:
     - сохраняет s3 ссылки;
     - разделяет CSV на партиции по 500k;
     - заливает в свой S3 (nfs-partner-platforms-target-send-auditory);
     - пишет в `t-segment.offline-upload.upload` (Kafka) сообщение AutoSegmentLoadingMessage.
```

### REST (Target → A-P)
- `AuditoryPlatformApi.kt` — WebClient + JWT HS512, exp 1h, claim `login`.
- Используется для синхронных операций (например, валидация или enrichment метаданных).

### REST (A-P → Target)
- `internal/client/target/apiclient.gen.go` — сгенерированный oapi-codegen клиент.
- A-P дёргает Target REST для запуска проливки.

### S3 (общая шина)
Бакеты Target:
```text
ms-auditory               (общие)
ms-auditory-tmp           (промежуточные)
target-auditory           (фасад)
target-load-auditory      (входящие проливки)
target-gather-auditory    (промежуточные результаты сборки)
target-send-auditory      (исходящие, перед отгрузкой)
```

A-P читает `target-send-auditory` (по сути, partition-CSV) и переразбивает.

---

## 3.5 SLA / freshness — что видно

### Из defaults TQ (Target использует TQ)
```text
taskExecutionTimeout = 10 minutes (per task)
taskFetchLimit       = 15 (одна итерация шедулера)
interval             = 5 seconds (минимальная задержка между шагами DAG)
```

### Из docs A-P
- На `t-segment.offline-upload.upload` партиции по **500k записей**.
- TTL аудиторий определяется `expiresAt` + `DeleteAuditoryTTLJob` (период не виден без чтения cron-конфига).

### Что **не** видно явно
- p95/p99 времени сборки одной аудитории.
- Дедлайн от заказчика (бизнес-SLA).
- Refresh interval динамических аудиторий — только то, что `RunDynamicAuditoryGathering` запускается по крону, но какой именно cron — нужно смотреть в `internal/application/cron.go`.

---

## 3.6 client_id list vs reference — что передаётся

### Сейчас (по коду)
- Target → A-P: **ссылки на S3 + checksums + rowCount**, не сам list.
- A-P → T-Segment: **CSV-файлы в S3** + ссылки в Kafka-сообщении.
- Target → каналы (`cbp.auditory`, `ms.auditory.extra-cashback`): **полное Avro-сообщение с метаданными**, плюс вероятно ссылки на S3 — нужно подтвердить чтением `auditory-export.avsc` и producer-кода.

### Ключевой вывод
Передача **по reference** (через S3 + checksum) уже фактически работает. Это **не блокер** для предложения «передавать reference вместо list» — наоборот, реальная физика именно такая.

Что отсутствует — единый **`result_id`**, по которому downstream может дозапросить нужные данные. Сейчас downstream должен сам читать S3.

---

## 3.7 Ключевые файлы для глубокого чтения

```text
1. application/auditory-loading/auditory-loading-api/.../AuditoryLoadingTQTaskType.kt
2. application/auditory-loading/auditory-loading-api/.../AuditoryLoadingTQTaskBuilder.kt
3. application/auditory-loading/auditory-loading-core/.../auditoryExport/AuditoryExportBuilder.kt
4. application/auditory-loading/auditory-loading-core/.../gathering/job/DynamicOfferAuditoryGatheringJob.kt
5. application/auditory-loading/auditory-loading-core/.../gathering/job/PartnerOfferAuditoryGatheringJob.kt
6. application/target-auditory/target-auditory-core/.../entrypoint/rest/TargetAuditoryControllerV2.kt
7. application/target-auditory/target-auditory-core/.../service/api/AuditoryPlatformApi.kt
8. application/infra/src/main/resources/application.yaml
9. contract/avro/auditory-export.avsc
10. contract/grpc-api/proto/AuditoryLoadingService.proto
```

---

## 3.8 Открытые вопросы по Target

```text
1. Какова реальная структура DAG-а самой долгой проливки (типы задач, ширина, глубина)?
2. Где код, который читает входной топик `nfs-partner-platforms.auditory` (т.е. что именно A-P шлёт обратно в Target — refresh? подтверждения?)
3. Сколько типов TQQueueType реально используется (KAFKA — точно, что ещё)?
4. Используется ли continuous=true в реальных процессах? Где?
5. Как делается dedup при повторной отправке одной и той же аудитории (idempotency на уровне messageId / gathering_id)?
6. Какие S3 retention настроены для каждого бакета? Это влияет на возможность переиспользовать результат.
7. Есть ли в Target прямые ClickHouse-запросы для compute, или весь compute делается через стандартные сервисы?
   — поиск по коду на `clickhouse|jdbc:clickhouse` в Target подтвердит.
```

→ см. [01-context/07-open-questions-for-team.md](07-open-questions-for-team.md).
