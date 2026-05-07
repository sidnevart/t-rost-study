# Sprint 07 — ClickHouse Baseline

## Цель недели
Снять реальные таймы CH-запросов под нашу аудитория-логику. Сделать материалы для предложения «one-SQL fast path» в Planner.

## Почему это важно
Без CH-чисел — RFC «давайте делать SQL» останется на уровне идеи.

---

## GPT-ускоритель

**Перед началом спринта** → вставь из `gpt-prompts.md` → Track 07 → Шаг 1 (Learning Curator).
**Если застрял на MergeTree** → вставь из `gpt-prompts.md` → Track 07 → Шаг 3 (Deep Explanation).
**В конце спринта** → вставь из `gpt-prompts.md` → Track 07 → Шаг 4 (Self-check).

---

## День 1 — Карта CH-таблиц
- `grep -rn "clickhouse" work_projects/target/application` — найти все обращения.
- Извлечь имена таблиц.
- Узнать у DBA partition_by и primary key.

**Практика:** заполнить файл «map of CH tables used by Target» (имя, объём, ключи).

**Артефакт:** `practice/sprint-07/ch-tables-map.md`.

**DoD:** известны 5-10 главных таблиц.

---

## День 2 — Один реальный запрос
- Выбрать 1 типичный compute-запрос (например, для use-case из sprint-04).
- Снять `EXPLAIN ESTIMATE`, `EXPLAIN PIPELINE`, `EXPLAIN PLAN actions=1`.
- Прочитать каждый план.

**Практика:** прогнать запрос на staging-CH, замерить wall clock и `query_log`.

**Артефакт:** аннотированный план + времена.

**DoD:** понимаю, на чём время уходит.

---

## День 3 — Materialized view
- Понять, можно ли заменить часто повторяющийся запрос на MV.
- Прочитать SimpleAggregateFunction, AggregatingMergeTree.

**Практика:** на тестовом сегменте создать MV и сравнить wall clock.

**Артефакт:** «MV vs raw query» бенчмарк.

**DoD:** есть число «во сколько раз быстрее с MV».

---

## День 4 — Partition pruning и predicate pushdown
- Проверить, что наши запросы используют partition_by.
- Если нет — почему (предикаты не на ключе).

**Практика:** взять «плохой» запрос, переписать с предикатом на partition key, замерить.

**Артефакт:** «партиции — до и после» сравнение.

**DoD:** знаю, какие наши запросы — partition-friendly.

---

## День 5 — Settings и tuning
- Прочитать про `optimize_read_in_order`, `max_threads`, `join_algorithm`.
- Применить разные SETTINGS к одному запросу.

**Практика:** замер на 5 разных конфигурациях.

**Артефакт:** «what worked, what didn't» заметка.

**DoD:** понимаю, какие SETTINGS критичны для нашего workload.

---

## Итог недели

```text
- Карта CH-таблиц.
- Один аннотированный реальный запрос.
- MV-эксперимент.
- Settings-эксперимент.
- Понимание, какой класс аудиторий — кандидат на one-SQL fast path.
```

## Финальный артефакт
`practice/sprint-07-clickhouse-baseline.md` — твои числа: query plan, wall clock, partition pruning. Что из них следует — твой вывод.

## Self-check
1. Какой % наших CH-запросов используют partition pruning?
2. Чем `optimize_read_in_order` помогает, и в каких запросах?
3. MV ускоряет — но что усложняет?
4. Какой `join_algorithm` дефолтный и стоит ли его менять?
5. Где именно у нас «full scan» в типичном запросе, и можно ли его убрать?
