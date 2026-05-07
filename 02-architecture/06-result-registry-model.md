# 06. Storage thinking — как думать про данные

> **Цель файла:** дать тебе **рамку**, чтобы оценить ЛЮБУЮ систему хранения. Не «вот моё предложение Result Registry» — тебе нужно сначала освоить общую теорию хранения, потом ты сам решишь, что нужно нашей системе.

---

## 6.1 Уровни хранения — от storage engine до domain

```text
1. Physical storage      — байты на диске / в памяти
2. Storage engine        — как байты структурированы (B-tree, LSM, columnar)
3. Database              — query language, transactions, replication
4. Application schema    — таблицы, индексы, foreign keys
5. Domain model          — как сущности связаны в бизнес-смысле
6. Result lifecycle      — сколько живут результаты, версии, snapshot
```

Большинство «архитектурных» обсуждений тонут на уровне 4 («давайте добавим колонку»). Senior-engineer держит все 6.

---

## 6.2 Главные оси хранения

```text
A. Read pattern
   - Point lookup      (по id)
   - Range scan        (по диапазону)
   - Aggregation       (sum, count, group by)
   - Full scan         (все строки)

B. Write pattern
   - Insert-only       (append, log)
   - Mutable           (UPDATE по pk)
   - Bulk load         (миллионы строк за раз)
   - Streaming         (поступление в реальном времени)

C. Consistency model
   - Strong (linearizable, serializable)
   - Eventual
   - Causal
   - Snapshot isolation

D. Durability
   - Synchronous write (commit = на диске)
   - Async / batched
   - Replication-based (committed когда N реплик подтвердили)

E. Schema
   - Strict (заранее определён)
   - Schema-on-read (JSONB, Avro в Kafka)
   - Hybrid (Postgres + JSONB)

F. Lifecycle
   - Permanent
   - TTL-based
   - Versioned (track history)
   - Soft-delete vs hard-delete
```

Когда смотришь на любую таблицу/бакет/topic — отметь, где она по 6 осям.

---

## 6.3 OLTP vs OLAP

```text
OLTP (Postgres у нас):
- Row-oriented
- Many small reads/writes
- ACID transactions
- Indexes optimize point lookups
- Per-query overhead высокий, но одно тело запроса дешёвое

OLAP (ClickHouse у нас):
- Column-oriented
- Few but huge reads
- Typically batched writes
- Compression dominates IO
- Per-column scan + filter эффективен
```

Промежуточные:
```text
- HTAP (TiDB, Postgres + Citus + columnar)
- Snowflake, BigQuery — pure analytical, eventually consistent на новые данные
- Kafka — log storage, не «база», но используется как persistence
- S3 — object store; для нас — артефакт-stage
```

Когда выбираешь storage — спроси: какой read pattern доминирует? Если разные read patterns на одной таблице — это запах.

---

## 6.4 Snapshot и versioning

Любая система с переиспользуемыми результатами рано или поздно встаёт на эти вопросы:

```text
- Можно ли воспроизвести результат через 3 дня?
- Если данные обновились — старый результат мёртв или допустим?
- Параллельный compute двух версий — допустим? как соединить?
- Кто чистит старые версии?
- Как выразить "as of T" семантику?
```

Это выходит за пределы storage engine. Это уровень 6 в иерархии 6.1. Если ты предлагаешь improvement — ты должен **явно** сказать, как меняешь lifecycle, а не просто «добавим колонку version».

---

## 6.5 Как читать наши таблицы

Не верь моим выводам. Сам пройди по таблицам:

### Шаг 1. Собрать карту
```text
A-P (Postgres):
  auditory, auditory_changes, auditory_metadata,
  load_auditory_request, t_segment_auditory,
  t_segment_filter_value, t_segment_filter_value_part_task,
  t_segment_filter_value_task, t_segment_filter_string_value,
  t_segment_task_status, t_segment_upload_metrics,
  mcc_category, cron_job_table

TQ (Postgres):
  tq_task, tq_task_dependencies, tq_process,
  tq_recurring_task, tq_recurring_task_run, tq_node

Target (Postgres):
  свой набор — найди в migrations.

Bucket-ы S3:
  ms-auditory, ms-auditory-tmp, target-auditory,
  target-load-auditory, target-gather-auditory, target-send-auditory.

Kafka topics: с обеих сторон.
```

### Шаг 2. Для каждой таблицы — заполни оси 6.2
Сделай это сам, в виде таблицы. Это упражнение само по себе многому учит.

### Шаг 3. Найди необычное
```text
- Где schema-on-read в основной системе (JSONB)?
- Где TTL отсутствует, но судя по росту строк должен быть?
- Где есть денормализация (одни данные в 2+ местах)?
- Где данные хранятся как CSV в S3 — почему не parquet?
```

Эти вопросы — материал для гипотез **тебе**, не моих.

---

## 6.6 Что изучить

| Тема | Что даёт | Источник |
|---|---|---|
| **DDIA гл. 3** | storage engines, B-tree vs LSM, columnar | book |
| **Database Internals** Petrov | глубже по deep storage | book |
| Postgres internals | MVCC, vacuum, indexes, partitions | postgresql.org docs + Bruce Momjian's talks |
| ClickHouse internals | MergeTree, parts, marks, projections | docs + Altinity blog |
| S3 semantics | consistency, multipart, lifecycle | AWS docs |
| Avro / Parquet | schema-on-read formats | их specs |
| Logical replication / CDC | Debezium, wal2json | docs |
| Lakehouse (Iceberg, Delta) | versioning + time travel в больших данных | их docs |

---

## 6.7 Упражнения

```text
УПРАЖНЕНИЕ A. Сделай таблицу: таблица | read pattern | write pattern | consistency | TTL.
              Заполни для всех таблиц A-P (15+).

УПРАЖНЕНИЕ B. Найди ту, у которой read и write pattern не совпадают с типом storage.
              (например, OLAP-style query на OLTP table.)
              Запиши, почему это сейчас работает / когда сломается.

УПРАЖНЕНИЕ C. Прочти EXPLAIN ANALYZE 3 разных запросов. Для каждого определи:
              какой index выбран и почему. Где не выбран — что мешает.

УПРАЖНЕНИЕ D. Сформулируй СВОЮ гипотезу про storage:
              «У нас сейчас X, потому что Y. Это работает в условиях Z. Сломается, если W».
              Без моих подсказок.
```

---

## DoD

```text
[ ] Карта всех таблиц с осями 6.2 заполнена.
[ ] Я понимаю разницу между OLTP и OLAP storage и могу её применить к нашему стеку.
[ ] Я могу читать EXPLAIN ANALYZE и не угадывать.
[ ] У меня есть СВОЯ гипотеза про storage, не моя.
```
