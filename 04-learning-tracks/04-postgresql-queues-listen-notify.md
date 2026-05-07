# Track 04 — PostgreSQL internals (для очередей, индексов, MVCC)

## Зачем учить
В нашем стеке Postgres везде: A-P (прикладная БД), TQ (queue с FOR UPDATE SKIP LOCKED), Target (своя БД). Ты не сможешь рассуждать про производительность, надёжность, миграции без понимания внутренностей PG.

«Использовал Postgres много лет» ≠ «понимаю Postgres». Нужно одно второе.

## Что учить — фундамент

### A. Storage и MVCC
```text
1. Page layout (8KB по умолчанию), tuple header
2. MVCC: xmin, xmax, ctid
3. Visibility rules: snapshot isolation, transaction id wraparound
4. HOT updates: что это, когда работает, fillfactor
5. Dead tuples и vacuum
6. Autovacuum: как настраивается, когда не справляется
7. WAL: что это, fsync, recovery
8. Logical decoding (база для CDC)
```

### B. Indexes
```text
1. B-tree (default), как структурирован, page splits
2. Hash, GIN, GiST, BRIN — когда какой
3. Multi-column index: order matters
4. Partial indexes (WHERE-clause)
5. Expression indexes (CREATE INDEX ON t ((lower(col))))
6. Covering indexes (INCLUDE)
7. Index-only scan vs index scan
8. Statistics: pg_stats, pg_class.reltuples, autovacuum_analyze
```

### C. Query planning
```text
1. EXPLAIN, EXPLAIN ANALYZE, EXPLAIN (BUFFERS)
2. Узлы плана: Seq Scan, Index Scan, Bitmap Heap Scan, Nested Loop, Hash Join, Merge Join
3. Когда planner выбирает Seq Scan несмотря на index (selectivity)
4. work_mem, shared_buffers, effective_cache_size
5. JIT (PG 11+) — для каких queries
6. Parallel query
```

### D. Concurrency и locks
```text
1. Уровни locks: AccessShare, RowShare, RowExclusive, ShareUpdateExclusive, ...
2. SELECT FOR UPDATE / FOR SHARE / FOR NO KEY UPDATE / FOR KEY SHARE
3. SKIP LOCKED, NOWAIT
4. Advisory locks (pg_advisory_lock)
5. Deadlock detection
6. Isolation levels: Read Committed, Repeatable Read, Serializable
```

### E. Очереди и patterns
```text
1. SKIP LOCKED queue pattern (что и почему делает TQ)
2. LISTEN/NOTIFY: лимиты payload, no persistence, fallback poll
3. Outbox pattern (для consistency между БД и Kafka)
4. CDC через logical replication / Debezium
5. Anti-paterns: SELECT for update без LIMIT, polling без index
```

### F. Replication, scalability
```text
1. Streaming replication (sync vs async)
2. Logical replication
3. Read replicas: lag, eventual consistency
4. Connection pooling (PgBouncer)
5. Partitioning (declarative, PG 11+)
6. Sharding — что Postgres сам не делает (и инструменты типа Citus)
```

## Что почитать

| Источник | Что брать |
|---|---|
| **Database Internals** (Petrov) | Часть I — глубже про storage engines |
| **PostgreSQL: Up and Running** (Obe & Hsu) | для общего "operational" понимания |
| **postgresql.org/docs** | главы по MVCC, WAL, indexes, planner |
| **Bruce Momjian's talks** (momjian.us/main/presentations/) | бесплатные глубокие материалы |
| **Use The Index, Luke!** (Markus Winand) | как читать query plans |
| **SQL Performance Explained** (Winand, ~200 стр) | компактное и must-read |
| 2ndquadrant blog | архив deep-dives |
| pganalyze blog | indexing, query analysis |
| Erwin Brandstetter answers на StackOverflow | золото для практических вопросов |

## Что сделать руками

```text
1. Развернуть локально PG 15+, наполнить 1M строк синтетическими tq_task.
2. На тех же данных:
   a. Прогнать `getAvailableTasks` запрос с EXPLAIN (ANALYZE, BUFFERS).
   b. DROP всех партиальных индексов, прогнать снова → сравнить план и время.
   c. CREATE INDEX каждого партиального по очереди → пометить, какой даёт прирост.

3. 4 параллельных psql-сессии: SELECT ... FOR UPDATE SKIP LOCKED.
   Замерить throughput vs число потребителей.

4. Симулировать update-heavy workload (100k UPDATE на 1M строк).
   - Замерить bloat (pg_stat_user_tables).
   - Прогнать VACUUM ANALYZE, пометить разницу.
   - Прочитать про HOT и почему это важно для queue.

5. LISTEN/NOTIFY:
   - Producer: после INSERT в queue → NOTIFY с id.
   - Consumer: LISTEN + fallback poll каждые 30 секунд (на случай пропуска).
   - Замерить latency LISTEN → consume vs poll(5s).

6. Прочитать `pg_stat_statements` после нагрузки — найти top-N медленных.
```

## Self-check

```text
1. Что такое HOT update, в каких условиях работает, почему критично для queue?
2. Чем partial index отличается от обычного, и когда planner его действительно выбирает?
3. Какова разница между FOR UPDATE и FOR UPDATE SKIP LOCKED в плане queries и locks?
4. Что такое snapshot isolation? Какие аномалии возможны при Read Committed?
5. Что показывает pg_stat_user_tables.n_dead_tup и когда стоит насторожиться?
6. Как работает autovacuum, и какие настройки имеет смысл тюнить для queue table?
7. Когда LISTEN/NOTIFY достаточно, а когда нужен полноценный broker (Kafka/NATS)?
8. Что такое xid wraparound и почему это критично?
9. Чем logical decoding отличается от streaming replication?
10. Что такое CTE OPTIMIZATION FENCE (PG до 12) и почему она была проблемой?
```

## DoD

```text
[ ] Я могу прочесть EXPLAIN ANALYZE и не угадывать.
[ ] У меня есть СВОИ замеры: throughput SKIP LOCKED, prefix scan vs partial index.
[ ] Я знаю, какие autovacuum настройки нужны для queue-table-а.
[ ] Я могу spot-ить anti-patterns в чужом SQL за 30 секунд.
[ ] Я могу аргументированно сравнить queue в PG vs Kafka vs NATS для конкретного use-case.
```
