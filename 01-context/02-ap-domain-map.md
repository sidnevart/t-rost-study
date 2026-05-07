# 02. A-P Domain Map

> A-P нельзя смотреть как «обёртку над TQ». Это **платформа** с собственным доменом и lifecycle. TQ в самом A-P не используется — TQ живёт в Target.

---

## 2.1 Audience Definition

### Структура `Auditory` (`internal/domain/model/auditory.go:16-37`)

```text
ID              int64?
Name            string
Type            STATIC | DYNAMIC
Initiator       string
IsHidden        bool
DateCreated     time.Time
DateUpdated     time.Time?
ExpiresAt       time.Time
DateHidden      time.Time?
S3Key           string?
Size            int64
Systems         []string
Params          *AuditoryParams      ← главный «DSL»
Version         int64
Status          DRAFT | GATHERING | REGATHERING | GATHERED | ERROR | ARCHIVED
ExportSegment   bool
LastModified    time.Time
GatheringId     uuid.UUID?           ← связь с конкретным запуском сборки
SdpDateSent     time.Time?
Source          string?
```

### `AuditoryParams` (`auditory.go:71-99`)

```text
Showcase              VYGODA | EXTRA_CASHBACK
TargetType            PARTNER_BASE | CLIENTS | PARTNER_AND_RECEIPT
ClientParams          { sex, age, income, regions, cities, cardType,
                        subscriptions{include/exclude},
                        paymentSystems, clientStatuses, loyaltyPrograms }
TerminalParams        IncludeExclude<TerminalParam{S3Location, dateRange}>
PartnerTerminalParams IncludeExclude<{partnerId, dateRange}>
BrandTerminalParams   IncludeExclude<{brandIds, dateRange}>
GeoParams             []GeoParam{lat, lon, radiusKm, daysThreshold}
HashParams            IncludeExclude<HashParam{S3Locations, EMAIL/PHONE_*, MD5/SHA512/NO_HASH, salt}>
MccParams             {codes, MONTHLY|GENERAL, dateRange, count, amount}
ReceiptParams         IncludeExclude<{dateRange, brandIds, categoryIds, ReceiptCategories tree}>
OfferActivationParams {offersForExclusion}
SegmentParams         []SegmentParam
```

**Это богатый DSL** — фактически набор предикатов, плюс include/exclude конструкции, плюс ссылки на S3-локации (HashParams, TerminalParams).

### Что из этого следует
- Аудитория — это **набор фильтров над клиентами/чеками/терминалами/гео/сегментами**, а не SQL.
- Часть параметров (HashParams, TerminalParams) — **внешние данные в S3**, которые нужно загрузить и применить.
- Возможна композиция: SegmentParams (готовые сегменты) + дополнительные фильтры.
- `Showcase` различает каналы: «Выгода» vs «Extra Cashback» — потенциально разные пайплайны.

---

## 2.2 Audience Lifecycle

```text
DRAFT
  ↓ (start gathering / loading request)
GATHERING ─────→ ERROR (на неудаче)
  ↓ (Target прислал результат)
GATHERED
  ↓ (params изменились / TTL — для DYNAMIC)
REGATHERING → GATHERED
  ↓ (TTL истёк / явное удаление)
ARCHIVED (через DeleteAuditoryTTLJob, либо isHidden=true)
```

### Триггеры переходов (что видно из кода и docs)
- `DRAFT → GATHERING`: HTTP `POST /auditory/{id}/loading` (`docs/components/rest/auditory/start-auditory-loading.mdx`) → `LoadAuditoryRequestProcessingJob`.
- `GATHERING → GATHERED`: kafka-consumer `delivery/kafka/target/auditory/consumer.go` обрабатывает сообщение из `nfs-partner-platforms.internal.auditory`, заполняет `s3Key` и `size`.
- `GATHERED → REGATHERING`: `RunDynamicAuditoryGathering` — крон находит DYNAMIC-аудитории с истёкшим интервалом.
- `* → ERROR`: при `error != null` в kafka-сообщении или ошибке экспорта в T-Segment (`docs/components/kafka/nfs-partner-platforms.internal.auditory.mdx`).
- `* → ARCHIVED/hidden`: `DeleteAuditoryTTLJob`.

### Важно
- **A-P сам не считает** аудиторию. Он отдаёт запрос в Target и ждёт результат.
- `GatheringId` (UUID) — связь между request и result. См. миграцию `2025-09-12-TGT-55898-add-gathering_id.xml`.
- Status как enum в Postgres (хранится строкой), нет типизированного state-machine — переходы валидируются в коде сервисов.

---

## 2.3 Planner / Routing — есть ли он?

### Что найдено
- В `internal/service/gathering/auditory_gathering_service.go` (6 символов) есть AuditoryDynamicGatheringService — он маппит запрос (`request_mapper.go`) и вызывает Target.
- В `internal/client/target/apiclient.gen.go` сгенерирован клиент Target API — REST-вызов на сборку.
- Альтернативные пути сборки **не найдены**: всё уходит в Target.

### Что отсюда следует
- Сейчас в A-P **нет planner-а / router-а** — всегда один путь: «делегируй Target».
- Это и есть точка для архитектурного предложения: ввести Planner, который выберет
  - быстрый путь (один SQL в ClickHouse, если параметры — простые),
  - средний путь (Target + TQ-DAG),
  - кэш / precomputed (если такая аудитория уже есть).
- См. [02-architecture/05-execution-router-planner.md](../02-architecture/05-execution-router-planner.md).

---

## 2.4 Хранение результата

### Что видно в коде

| Где | Что |
|---|---|
| `auditory.s3_key` (PG) | ссылка на S3-объект с client_id-ами для одной аудитории |
| `auditory.size` | размер аудитории |
| `t_segment_auditory.s3_key` | дополнительная ссылка на S3 для t-segment-выгрузки |
| `t_segment_filter_value` | значения фильтров (загружаемые из CSV в S3) |
| S3 buckets `nfs-partner-platforms-target-send-auditory/<auditory_id>/...` | разбиение на партиции (`SegDataS3URLs` в kafka-сообщении) |

### Признаки отсутствия «Result Registry»
- Один `s3_key` — один результат. Нет версионирования.
- Reference-передача в downstream возможна (`s3Key` известен), но реально A-P **физически отгружает CSV** в `t-segment.offline-upload.upload`. Reference-only пока не используется.
- Нет таблицы вроде `audience_result_version (audience_id, version, snapshot_s3_key, computed_at, params_hash)` — хотя `Version int64` в модели есть, она монотонно инкрементируется при изменении params.

### Что предложить
- См. [02-architecture/06-result-registry-model.md](../02-architecture/06-result-registry-model.md).

---

## 2.5 Export / Delivery

### Куда уходит аудитория
1. **T-Segment** через Kafka `t-segment.offline-upload.upload` (`docs/components/kafka/nfs-partner-platforms.internal.auditory.mdx`):
   - CSV-чанки по 500k записей в S3, ссылки + checksum + rowCount в сообщении.
   - Поля: `messageId, dataSource=TARGET, segId, isDiff, segDataS3URLs[], dataDateTime, model{format=CSV, keys[clientId, action, recency]}`.
   - Это и есть «доставка в downstream» с точки зрения A-P.

2. **SDP** (Smart Data Platform) — параметры аудитории через `auditory_params_sdp_service.go` и Avro-сообщения (`contract/avro/auditory_*_params.avsc`).
   - Это передача **самих фильтров**, а не результата. Используется аналитикой / DLH.

3. **Уведомления** в Time / Pechkin — нотификации ответственным.

### Что A-P **не** делает
- Не отправляет client_id-ы напрямую в push/email/sms каналы. Это делает Target и потребители T-Segment.
- Не хранит client-data долго (S3 retention управляется отдельно — нужно уточнить TTL бакетов).

---

## 2.6 Observability

| Где | Что видно |
|---|---|
| `internal/lib/logger/` | structured slog с context-handler |
| `tracing.go` | OTel tracing setup |
| Prometheus | через стандартный `application/` (точно есть, но конкретные метрики надо смотреть в коде сервисов) |
| `t_segment_upload_metrics` (PG) | таблица с таймингами выгрузки |
| `TSegmentUploadMetricsSendingJob` | джоба, отправляющая эти метрики наружу |

### Открытые вопросы по observability
- Какие SLO заведены? Где?
- Видны ли в Grafana разбивка `compute / export / delivery`?
- Есть ли correlation_id, который сквозит от A-P → Target → A-P → T-Segment?

---

## 2.7 Карта таблиц (PG)

```text
auditory                          — главная сущность
auditory_changes                  — журнал изменений
auditory_metadata                 — описание/счётчик/аналитик
load_auditory_request             — очередь запросов на проливку
t_segment_auditory                — связь auditory ↔ t-segment
t_segment_filter_value            — значения фильтров
t_segment_filter_value_part_task  — задачи разделения файлов значений
t_segment_filter_value_task       — задачи обработки значений
t_segment_filter_string_value     — текстовые значения фильтров
t_segment_task_status             — статус t-segment-задачи
t_segment_upload_metrics          — метрики
mcc_category                      — справочник MCC
cron_job_table                    — locker для крон-джоб (single-leader)
```

### Важная деталь
- `cron_job_table` (`2025-08-26-TGT-55904-create-cron-job-table.xml`) — таблица-локер. Используется `internal/lib/scheduler/locker.go`. Это типичный паттерн «один лидер на крон через INSERT/UPDATE с TTL» — а не CronJob от K8s.

---

## 2.8 Что обязательно проверить руками (DoD на этот файл)

```text
[ ] Запустить локально A-P (`init_dev_environment.sh`, `make build-openapi`, `run auditory-platform`).
[ ] Создать одну STATIC и одну DYNAMIC аудиторию через REST.
[ ] Посмотреть в БД, как меняется `status`, `gathering_id`, `s3_key` у конкретной строки `auditory`.
[ ] Сделать tail логов вокруг старта проливки и поймать момент `GATHERING → GATHERED`.
[ ] Записать наблюдения сюда же, в этот файл, в раздел «Личные заметки».
```

### Личные заметки (заполнить самому)
```text
1. Тип аудитории, которую тестировал: ...
2. Сколько прошло от DRAFT до GATHERED: ...
3. Какие kafka-топики реально дёргались (по логам): ...
4. Что осталось непонятно: ...
```
