# Track 06 — DAG алгоритмы и Critical Path

## Зачем учить
TQ-DAG — это в чистом виде граф. «Когда задача может начаться» — это «когда все предки COMPLETED». Время DAG-а = critical path. Без формальной модели — трудно говорить про «можно ли это распараллелить».

## Где это есть в проектах
- TQ, `db/changelog/002_create_tq_task_dependencies_table.xml` — структура графа.
- TQ, `application/engine/storage/postgresql/.../task/TQJdbiTaskDao.kt:160-191` — `getAvailableTasks` (BFS-like).
- Target, `application/auditory-loading/auditory-loading-api/.../tq/api/AuditoryLoadingTQTaskBuilder.kt` — DSL для построения DAG.
- TQ, `docs/API/core-processes/continuous-processes.md` — тот же DAG, но с приоритет-cascade.

## Что изучить
```text
1. BFS / DFS — базовые обходы.
2. Topological sort (Kahn's algorithm, DFS-based).
3. Critical Path Method (CPM) — для оценки минимального времени DAG.
4. Single-source shortest path (если ввести «вес» = ожидаемое время задачи).
5. SCC (Strongly Connected Components) — для детектирования циклов в зависимостях.
6. Levelization: разбить DAG на «уровни», уровни параллелятся.
7. Параметры: width (число задач на уровне), depth (число уровней),
   width × time_per_task vs depth × time_per_task.
```

## GPT-ускоритель

**Шаг 1 — Learning Curator** → вставь из `gpt-prompts.md` → Track 06 → Шаг 1.
**Шаг 3 — Deep Explanation** (если застрял на CPM) → вставь из `gpt-prompts.md` → Track 06 → Шаг 3.
**Шаг 4 — Self-check** → вставь из `gpt-prompts.md` → Track 06 → Шаг 4.

---

## Практика на проекте

### Задача 1 — Critical path скрипт (обязательно)
- Снять реальный dump `tq_task` + `tq_task_dependencies` со staging:
```sql
COPY (SELECT * FROM tq_task WHERE created_at > now() - interval '1 day') TO '/tmp/tasks.csv' CSV HEADER;
COPY (SELECT * FROM tq_task_dependencies) TO '/tmp/deps.csv' CSV HEADER;
```
- Написать Go/Kotlin скрипт (~150 строк): загрузить граф, запустить topological sort, найти critical path.
- Результат: top-10 самых длинных критических путей по типам процессов.

### Задача 2 — Cycle detection
- В том же скрипте — добавить обнаружение циклов (DFS + color marking).
- Написать: «что произошло бы в TQ, если бы в граф попал цикл?»
- Найти в коде TQ — есть ли защита от circular dependencies?

### Задача 3 — Depth/width статистика
- На dump за 1 день: посчитать depth/width для всех process_id.
- SQL + скрипт: `SELECT process_type, avg(depth), max(depth), avg(width), count(*) FROM ...`
- Ответить: для каких типов процессов `depth × interval` = bottleneck?

### Задача 4 — Idle time vs compute time
- Для выборки из 100 процессов: посчитать что занимает больше времени — выполнение задач или ожидание между ними (interval + queue time).
- SQL:
```sql
SELECT process_id,
  sum(completed_at - started_at) as compute_time,
  sum(started_at - (lag(completed_at) OVER (PARTITION BY process_id ORDER BY started_at))) as idle_time
FROM tq_task WHERE status = 'COMPLETED'
GROUP BY process_id;
```
- Вывод: если idle_time > compute_time — DAG overhead реальный, не воображаемый.

## Артефакты
- Скрипт + результат: top-N критических путей.
- Распределение depth/width по типам процессов.
- Раздел в `02-architecture/04-graph-vs-sql-vs-bitmap-vs-cache.md` пополнить наблюдениями: «когда DAG избыточен».
- Конспект CPM на 1 страницу.

## Self-check
1. Если DAG имеет depth=10 и interval=5s, какова минимальная latency без continuous?
2. Что такое critical path и почему ширину нет смысла увеличивать сверх неё?
3. Как detect-ить cycle в DAG, и что произошло бы, если он попадёт в TQ?
4. Levelization: всегда ли N задач на одном уровне = N-кратный speedup?
5. Когда parallelism в DAG ограничен queue-limits, а не depth?

## DoD
```text
[ ] Скрипт + числа.
[ ] CPM-формальное определение записано.
[ ] Распределение depth/width известно для топ-3 типов TQ-процессов.
```
