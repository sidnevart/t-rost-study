# 05. End-to-End Audience Flow

> Один проход: от бизнес-запроса до доставки в downstream. Каждый шаг привязан к конкретному коду или открытому вопросу.

---

## Сценарий: STATIC аудитория с фильтрами по терминалам и MCC

### Шаг 1. Создание (HTTP)

```text
Клиент (UI/админка) → A-P REST POST /auditory
  Тело: CreateAuditoryRequest {name, type=STATIC, params: {...}, exportSegment: true}
```

| Где | Что происходит |
|---|---|
| `internal/api/auditory/controller.go` | приём запроса, валидация |
| `internal/delivery/http/validator/auditory_validator.go` | бизнес-валидация params |
| `internal/service/auditory_service.go` | создание `auditory` row, status = DRAFT |

**Вопрос-bottleneck:** валидация HashParams проверяет S3-локации? Если да — это IO-нагрузка на синхронный путь.

---

### Шаг 2. Старт проливки (HTTP)

```text
Клиент → A-P POST /auditory/{id}/loading
```

| Где | Что |
|---|---|
| `internal/api/auditory/controller.go` | принимает |
| `internal/repository/load_auditory_request_repository.go` | вставка в `load_auditory_request` |
| `auditory.status` → GATHERING | (через service) |

`load_auditory_request` — это **очередь запросов на проливку внутри A-P** (отдельная от TQ).

---

### Шаг 3. Обработка запроса A-P-стороны

| Где | Что |
|---|---|
| Cron `LoadAuditoryRequestProcessingJob` (`docs/components/job/...`) | поллит `load_auditory_request` |
| `internal/service/loading/load_auditory_service.go` | для каждого запроса — вызывает Target REST |
| `internal/client/target/apiclient.gen.go` | сгенерированный oapi-codegen клиент |

**Bottleneck-кандидат:** poll-интервал крона = задержка старта. Альтернатива — LISTEN/NOTIFY или Kafka.

---

### Шаг 4. Compute в Target

```text
Target REST принял запрос → создал TQProcess
  ↓
TQ-DAG (упрощённо для STANDARD_AUDITORY_LOADING):
  [GATHER]  ─→ [PROCESS] ─→ [EXPORT]  ─→ [SEND]
   |            |             |            |
   v            v             v            v
   читает      применяет      пишет        пишет
   источники   include/exclude чанки в S3   Kafka
```

| Где | Что |
|---|---|
| `auditory-loading-core/.../gathering/job/*GatheringJob.kt` | стартует процесс |
| `auditory-loading-api/.../tq/api/AuditoryLoadingTQTaskBuilder.kt` | строит граф задач |
| TQ scheduler внутри Target | забирает task с FOR UPDATE SKIP LOCKED |
| `auditory-loading-core/.../auditoryExport/AuditoryExportBuilder.kt` | финальная сборка экспортного сообщения |
| `auditory-loading-core/.../loading/producer/AuditoryExportProducer*.kt` | пишет Kafka |

**Bottleneck-кандидаты:**
- `interval=5s` шедулера TQ → минимум 5с между шагами DAG (если не continuous=true).
- `taskExecutionTimeout=10min` на задачу. Длинные ClickHouse-запросы могут резать `TIMEOUT`.
- Размер аудитории → число партиций → IO в S3.
- Network round-trip к источникам данных (где они? нужно уточнить).

---

### Шаг 5. Возврат в A-P через Kafka

```text
Target → Kafka topic `nfs-partner-platforms.internal.auditory`
  Avro-сообщение: {gathering_id, error?, S3-ссылки на CSV-партиции, isDiff, dataDateTime}
```

| Где | Что |
|---|---|
| `internal/delivery/kafka/target/auditory/consumer.go` | A-P consumer |
| Если `error != null`: status=FAILED + Pechkin (Time) нотификация | |
| Иначе: A-P **переразбивает** на партиции по 500k и **загружает в свой S3** | |
| `internal/service/s3/chunk/chunk_upload_service.go` | разбивка |
| `internal/service/s3/partition/auditory_service.go` | партиционирование |
| Затем: `t-segment.offline-upload.upload` Kafka-сообщение | |

**Bottleneck-кандидат:** A-P **повторно** читает CSV из target-bucket и переписывает в свой. Это лишний S3 round-trip. Если бакет общий — можно sub-second оптимизировать.

---

### Шаг 6. Доставка в T-Segment

```text
A-P → Kafka `t-segment.offline-upload.upload`
  AutoSegmentLoadingMessage {messageId, dataSource=TARGET, segId, isDiff,
                              segDataS3URLs[], dataDateTime, model{format=CSV, keys}}
```

| Где | Что |
|---|---|
| `contract/json/auto_segment_loading_message.go` | формат сообщения |
| T-Segment | принимает, делает свои операции |
| Возврат через `t-segment.offline-upload.upload-info` | статус выгрузки |
| `internal/delivery/kafka/tsegment/status/consumer.go` | A-P consumer статусов |
| Обновляет `t_segment_auditory.status` и `t_segment_upload_metrics` | |

---

### Шаг 7. Доставка в каналы

```text
T-Segment / Target → продакшен-каналы (push, email, ads, cb-platform)
```

A-P **не участвует напрямую**. Каналы потребляют через свои контракты (`cbp.auditory`, `ms.auditory.extra-cashback`, `ms.manual-badge-fire-auditory` и т.п.).

---

## Тайминги (грубая декомпозиция)

| Этап | Что доминирует | Кто хост | Где замерить |
|---|---|---|---|
| HTTP create | сеть + PG insert | A-P | request log |
| Loading request → start compute | poll-интервал крона A-P | A-P | timestamp в `load_auditory_request` |
| Compute (Target TQ-DAG) | размер аудитории, число шагов, interval=5s между шагами | Target | `tq_task.date_started/date_finished` |
| Export Target → S3 | partition count + S3 throughput | Target | `tq_task` для EXPORT |
| Возврат Kafka → A-P | партиция/батчинг consumer | A-P | log + offset lag |
| A-P переупаковка S3 | количество строк / partition size | A-P | `chunk_upload_service` |
| `t-segment.offline-upload.upload` | T-Segment side | T-Segment | вне зоны видимости |

> «Аудитория за N секунд» — почти всегда смешивает разные слои. См. [06-compute-export-delivery-split.md](06-compute-export-delivery-split.md).

---

## Где **точно** есть лишние round-trip-ы (по коду)

```text
1. Target пишет в свой S3 → A-P читает и перезаписывает в свой S3.
   - Если оба бакета внутри одного S3-кластера, можно делать `CopyObject` без download/upload.
   - Лучше: договориться о **shared bucket**, и в A-P только перепаковка metadata.

2. Cron-полл `load_auditory_request` в A-P даёт лаг старта.
   - Альтернатива: Kafka-event на момент создания запроса, или PG LISTEN/NOTIFY.

3. TQ interval=5s между шагами DAG, если не continuous=true.
   - Для длинных DAG это +N*5s к общему времени.
   - Альтернатива: continuous=true для critical DAG, или меньший interval, или event-driven trigger.

4. Передача CSV в downstream — full payload каждый раз.
   - Альтернатива: result_id + downstream сам читает (когда нужен).
   - Где это применимо vs где нужен push — в [02-architecture/02-target-contract.md](../02-architecture/02-target-contract.md).
```

---

## Что построить руками для проверки

```text
[ ] Включить debug log на Target и A-P, прогнать одну STATIC аудиторию.
[ ] По таймстемпам собрать таблицу: t_HTTP_create, t_loading_start, t_first_task_started,
    t_last_task_finished, t_kafka_msg_to_AP, t_csv_to_S3, t_kafka_to_TSegment.
[ ] Перенести в `01-context/06-compute-export-delivery-split.md` как реальный baseline.
[ ] Если есть доступ к staging — воспроизвести на 3 разных размерах аудитории (10k, 1M, 10M).
```

---

## Открытые вопросы по этому flow

```text
1. Как именно A-P стартует Target — REST sync (request → process_id обратно)? Или просто fire-and-forget?
2. Что A-P делает, если Target не ответил вовремя? (timeout, retry, dead letter)
3. У STATIC и DYNAMIC аудиторий — разные пайплайны в Target, или один с флагом?
4. Кто гарантирует exactly-once в `nfs-partner-platforms.internal.auditory`?
   (Kafka headers, dedup по messageId/gathering_id, idempotent consumer?)
5. Что происходит, если A-P получил повторное сообщение после уже COMPLETED state?
```
