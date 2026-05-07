# Track 08 — Data Locality / S3 / Predicate Pushdown

## Зачем учить
В нашей системе минимум **двойная упаковка S3** (Target пишет → A-P переупаковывает). Также есть HashParams как S3-locations. Понимать, как минимизировать round-trip — это прямой выигрыш по latency и cost.

## Где это есть в проектах
- A-P, `internal/lib/s3/manager.go` + `internal/client/s3_client.go` — клиент S3.
- A-P, `internal/service/s3/chunk/chunk_upload_service.go` — нарезка чанков.
- A-P, `internal/service/s3/partition/auditory_service.go` — партиционирование.
- A-P, `internal/service/s3/merge/service.go` — merge.
- A-P, `internal/service/s3/mapping/t_segment_service.go` — маппинг.
- Target, бакеты: `target-load-auditory`, `target-gather-auditory`, `target-send-auditory`, `ms-auditory`.
- A-P, бакеты: `nfs-partner-platforms-target-auditory-qa`, `...-load-...`, `...-gather-...`, `...-send-...` (`infra/.../application.yaml:152-176` Target side).

## Что изучить
```text
1. S3 model: object, key, ETag, multipart upload, copy-on-server.
2. CopyObject vs download/upload: когда какой быстрее.
3. Server-side encryption и cost.
4. Lifecycle policies / retention / IA / Glacier.
5. Listing semantics: ListObjectsV2 paging.
6. Eventual consistency (S3 теперь strong, но было важно знать историю).
7. Parquet vs CSV: предикат-пушдаун на parquet (column-store + row groups).
8. CH s3() table function: как читать parquet/csv напрямую.
9. Pre-signed URLs.
10. Throughput: ограничения по бакету / per-prefix.
```

## Практика на проекте
- Найти место, где A-P **скачивает** файл из target-bucket-а и **загружает** в свой. Замерить байты и время.
- Прототип: использовать `CopyObject` (server-side) — измерить разницу.
- Эксперимент: чтение parquet через CH `s3()` — сколько секунд занимает скачивание + парсинг.
- Прогнать выборку: «если в нашей системе 1M client_id экспортируется как parquet вместо CSV — насколько меньше байт».

## Артефакты
- Заметка «двойная упаковка S3: где, сколько и как удалить».
- Бенчмарк CSV vs parquet (объём, время чтения).
- Раздел в `02-architecture/02-target-contract.md` или новый файл, обосновывающий «shared bucket».
- Эксперимент №3 в `06-practice/01-experiment-backlog.md` с реальными числами.

## Self-check
1. Когда CopyObject невозможен (cross-region? cross-account? разные lifecycle policies?).
2. Что такое pre-signed URL и почему он удобен/опасен?
3. Чем parquet принципиально лучше CSV для нашего use-case?
4. Какие есть ограничения throughput per-prefix и стоит ли их обойти namespace-prefix-ами?
5. Где в коде A-P сейчас реальная двойная упаковка?

## DoD
```text
[ ] Найдена точка двойной упаковки.
[ ] Замер CopyObject vs download/upload.
[ ] Бенчмарк parquet vs CSV.
[ ] Запись эксперимента №3.
```
