# Track 07 — ClickHouse Performance

## Зачем учить
ClickHouse — основной кандидат на «one-SQL fast path» в Planner. Чтобы предложить это серьёзно — нужно уметь читать query plan, понимать MergeTree, projections, materialized views, partition pruning, sample_by. Без этого предложение «считать аудиторию через CH» останется уровнем powerpoint.

## Где это есть в проектах
- В A-P / Target прямые CH-запросы — нужно найти. Поиск: `grep -rn "clickhouse\|jdbc:clickhouse" /Users/a.i.sidnev/Documents/work_projects/target` (зафиксировать).
- T-Segment — потребитель аудиторий, тоже на CH (предположение, проверить).
- DLH-процессы (`a-p/docs/processes/dlh/`) — поток в Data Lake House.

## Что изучить
```text
1. MergeTree storage: parts, marks, granules, primary key vs sorting key.
2. Index types: minmax, set, bloom_filter (data skipping indexes).
3. Materialized views: reading + writing, AggregatingMergeTree, SimpleAggregateFunction.
4. Projections (CH 21.3+) — альтернатива MV.
5. Partition pruning: PARTITION BY и query plan.
6. Predicate pushdown: WHERE-фильтр на mark-уровень.
7. Distributed engines vs Replicated, replication semantics.
8. JOIN в CH: типы (hash, merge, partial_merge), реальная стоимость, GLOBAL JOIN.
9. SAMPLE clause — как и когда применим.
10. SETTINGS максимально полезные:
    max_threads, max_memory_usage, optimize_read_in_order,
    join_algorithm, max_block_size.
11. Анти-паттерны: SELECT ... ORDER BY without LIMIT, full scan, не-prefix предикат на ключе.
```

## Практика на проекте
- Снять список всех таблиц CH, к которым обращается A-P/Target.
- Для 1 типичного запроса — `EXPLAIN PIPELINE`, `EXPLAIN ESTIMATE`, `EXPLAIN PLAN` — читать вместе.
- Сравнить два варианта одного запроса: «in-Kotlin filter» vs «push down to CH».
- Предложить 1 материализованное представление для повторяющегося запроса.

## Артефакты
- Карта CH-таблиц проекта (имя, размер, partition_by, primary key).
- Один real-life query с EXPLAIN ANALYZE / EXPLAIN PIPELINE.
- Бенчмарк: 1 SQL vs текущий DAG-путь для одной аудитории. Числа в `06-practice/01-experiment-backlog.md` №10.
- Предложение MV / projection с обоснованием.

## Self-check
1. Чем primary key в CH отличается от primary key в Postgres?
2. Что такое granule и почему он важен для производительности?
3. Когда materialized view действительно ускоряет, а когда — заставляет писать дважды?
4. Что такое `optimize_read_in_order` и в каких случаях оно даёт большой выигрыш?
5. Как diagnose «почему запрос медленный» в CH — в каком порядке открываешь инструменты?

## DoD
```text
[ ] Список CH-таблиц проекта.
[ ] EXPLAIN ANALYZE для 1 типичного запроса.
[ ] Бенчмарк one-SQL vs DAG.
[ ] Понимание, какой запрос станет fast-path кандидатом в Planner.
```
