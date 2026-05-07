# 04. TQ Domain Map

> TQ (`ninja-t-queue`) — **Kotlin-библиотека**, не сервис. Подключается как зависимость в Target. Это меняет всё: «улучшить TQ» означает изменить контракт для всех потребителей. Любые предложения по архитектуре сборки должны учитывать это.

---

## 4.1 Сущности

```text
TQProcess          — родитель DAG-а задач (id UUID, status, params, priority, type, continuous BOOL)
TQTask             — единица работы (id UUID, type, status, params/result JSONB, priority,
                     retry_count, queue_type, process_id, node_id, date_reserved, dates)
TQTaskDependency   — ребро DAG (task_id, dependency_task_id) — оба PK + FK
TQRecurringTask    — повторяющаяся задача (cron-выражение)
TQRecurringTaskRun — журнал запусков recurring
TQNode             — heartbeat ноды (id UUID, live_until)
```

### Статусы TQTask
```text
NEW           — создана, ждёт зависимостей или мощностей
IN_PROGRESS   — занята воркером (строго одной нодой через node_id)
COMPLETED     — успешно
FAILED        — упала, retry_count исчерпан или невозвратная ошибка
INTERRUPTED   — нода умерла во время IN_PROGRESS (восстанавливается шедулером)
TIMEOUT       — превысила taskExecutionTimeout или taskStaleNewTimeout
```

### Статусы TQProcess
```text
NEW          — создан, ещё не стартовал
IN_PROGRESS  — хотя бы одна задача стартовала
COMPLETED    — все задачи завершены
FAILED       — обработчик исключений «потопил» процесс
```
(точные имена статусов — в `application/api/common-api/.../TQProcessStatus`, набор `ACTIVE_PROCESS_STATUSES` используется в `TQJdbiTaskDao.kt:175`)

---

## 4.2 Граф зависимостей (DAG)

### Из миграции `002_create_tq_task_dependencies_table.xml`
```sql
CREATE TABLE tq_task_dependencies (
  task_id UUID,                  -- задача-потребитель
  dependency_task_id UUID,        -- задача-предок
  PRIMARY KEY (task_id, dependency_task_id)
);
```

### DSL построения (`docs/intro.md`, `TQTaskDsl.kt`)
```kotlin
tqueue(tq, priority = 1, continuous = true) {
    val a = task("STAT") { params["nums"] = listOf(1,2,3) }
    val b = task("STAT") { params["nums"] = listOf(4,5,6) }
    task("EMAIL") { params["to"] = "x@y" }.dependsOn(a, b)
}
```

### Резолв зависимостей (запрос `getAvailableTasks`, `TQJdbiTaskDao.kt:160-191`)
```sql
SELECT t.* FROM tq_task t
WHERE t.status = 'NEW'
  AND t.node_id IS NULL
  AND NOT EXISTS (              -- все предки завершены
    SELECT 1 FROM tq_task_dependencies d
    JOIN tq_task tt ON d.dependency_task_id = tt.id
                   AND tt.status IN (NEW, IN_PROGRESS, FAILED, INTERRUPTED, TIMEOUT)
    WHERE d.task_id = t.id
  )
  AND EXISTS (                  -- процесс ещё активен
    SELECT 1 FROM tq_process p
    WHERE p.id = t.process_id AND p.status IN (active)
  )
ORDER BY t.priority ASC NULLS LAST, t.date_created ASC
LIMIT :fetchLimit
FOR UPDATE SKIP LOCKED;
```

**Ключевой инсайт:** проверка предков — `NOT EXISTS` по «незавершённым» статусам. Это дешевле, чем «все предки COMPLETED», потому что Postgres может рано выйти из подзапроса.

---

## 4.3 Scheduler / координация нод

### Цикл (`TQScheduler.kt`, упрощённо)
```text
loop every TQConfig.interval (default 5s):
  1. Прочитать живые ноды и их текущую загрузку.
  2. С учётом maxConcurrentTasks и queueLimits — определить «сколько слотов».
  3. SELECT ... FOR UPDATE SKIP LOCKED — забрать N задач (taskFetchLimit=15).
  4. На забранных проставить node_id + date_reserved.
  5. Запустить выполнение в schedulingScope (Coroutines, Dispatchers.IO).
  6. По завершении — обновить status + дать «толчок» зависимым.
```

### Heartbeat (`TQHeartbeatService.kt`, `tq_node`)
```text
- Каждые heartbeatInterval (15s): UPSERT в tq_node, live_until = now() + 30s.
- Мёртвые ноды (live_until < now()): шедулер их видит, освобождает их задачи.
```

### Освобождение задач после смерти ноды
- `releaseStaleAssignedTasks(seconds=300)` — задачи NEW + node_id != null + date_reserved старше 5min → node_id=NULL (вернуть в очередь).
- `markTimedOutStaleNewTasks(seconds=24h)` — задачи NEW + node_id IS NULL + date_created старше 24ч → TIMEOUT.

---

## 4.4 Concurrency и FOR UPDATE SKIP LOCKED

### Зачем
- Несколько нод одновременно делают `SELECT ... FOR UPDATE SKIP LOCKED LIMIT 15`.
- Каждая нода получает свои 15 строк, не блокируясь на тех, что уже взяла другая.
- **Никаких дубликатов** — задача ровно у одной ноды (заодно ставится `node_id`).

### Index design (`014_add_new_indices_on_database.xml`)
- Партиальные индексы под именно те предикаты, которые шедулер использует:
  - `(priority ASC NULLS LAST, date_created ASC) WHERE status='NEW' AND node_id IS NULL` — для очереди.
  - `(queue_type, date_started) WHERE status='IN_PROGRESS'` — для лимитов очереди.
  - `(date_reserved) WHERE status='NEW' AND node_id IS NOT NULL` — для recovery.
  - `(node_id) WHERE status IN ('IN_PROGRESS','NEW')` — для action-on-node-death.
  - `(id) WHERE status IN ('NEW','IN_PROGRESS','FAILED','INTERRUPTED')` — для обхода активных задач.
  - `(type, status, date_created)` — для observability.

### Что это даёт
- Шедулер забирает «top-N» из NEW-задач без full scan.
- Ноды между собой **не конфликтуют** при одновременном poll.

---

## 4.5 Continuous processes

### Проблема (из `docs/API/core-processes/continuous-processes.md`)
> Промежуточные данные имеют TTL. Если задача в DAG ждёт в очереди слишком долго, данные протухают.

### Решение
- При создании процесса с `continuous=true`:
  - Все задачи имеют изначальный `priority` процесса (например, 2).
  - Когда **первая задача** перешла в `IN_PROGRESS`, все оставшиеся NEW-задачи этого процесса получают `priority=-1`.
- На следующем шедулинг-цикле они подхватываются первыми.

### Trade-off
- Решает «зависание DAG», но не решает **самой первой задержки** (до старта первой задачи).
- Может вызвать priority inversion: continuous-задачи блокируют другие важные.

---

## 4.6 Retry / failure handling

### `maxRetryCount = 1` (default)
- При ошибке: `retry_count++`, status может вернуться в NEW для повторного забора (см. `TQApiService.kt`, `TQProcessExceptionProcessor.kt`).
- При `retry_count >= maxRetryCount` → FAILED.

### Что происходит с зависимыми
- Если предок FAILED — потомок остаётся в NEW навсегда (зависимости не выполнятся), пока кто-то явно не решит, как поступить.
- В коде есть `TQProcessExceptionHandler` — кастомный обработчик процесса. Команда могла подключить свою логику восстановления.

### Открытый вопрос
- Что значит «1 retry»? Это «1 повтор после первой неудачи» (т.е. до 2 попыток) или «вообще одна попытка»?
- → проверить в `TQApiService.kt` и тестах.

---

## 4.7 Failure modes (что может пойти не так)

```text
1. Нода умерла во время IN_PROGRESS:
   - Heartbeat ttl истечёт (30s).
   - Шедулер пометит задачи INTERRUPTED → возможно вернёт в NEW.
   - taskExecutionTimeout (10min) — страховочный механизм поверх.

2. Нода забрала NEW (node_id, date_reserved) и не стартовала:
   - taskReservationTimeout=5min → node_id=NULL, обратно в очередь.

3. Задача застряла в IN_PROGRESS дольше 10min:
   - taskExecutionTimeout → TIMEOUT.

4. Задача NEW > 24h без ноды:
   - taskStaleNewTimeout → TIMEOUT.

5. Зависимость FAILED:
   - Потомок не запустится (NOT EXISTS в getAvailableTasks вернёт false при FAILED предке —
     потому что FAILED входит в notCompletedTaskStatus).
   - Это — design feature: цепочка останавливается. Команда должна вмешаться.
```

---

## 4.8 Что TQ **не** делает

```text
- Не делает дистрибьюшен задач по нодам активно: ноды pull-моделью забирают сами.
- Не делает SLA-tracking. Только timeouts.
- Не делает приоритизацию по deadline (только static priority).
- Не делает batching: одна задача = один воркер-коллбек.
- Не делает dataflow между задачами автоматически: данные передаются через task.params/result в JSONB.
- Не делает типизированное API без registry — нужно регистрировать TQWorker.
- Не делает streaming: задачи дискретные.
- Не имеет cron-выражений уровня K8s — есть только TQRecurringTask.
```

---

## 4.9 Где это в коде (top-10)

```
1. application/api/common-api/.../main/TQ.kt                          — entry point
2. application/api/common-api/.../main/TQTaskDsl.kt                   — DSL
3. application/api/common-api/.../configuration/TQConfig.kt           — все настройки
4. application/engine/scheduler/core/.../TQScheduler.kt               — главный цикл
5. application/engine/storage/postgresql/.../task/TQJdbiTaskDao.kt    — все SQL по задачам
6. application/engine/scheduler/heartbeat/.../TQHeartbeatService.kt   — координация нод
7. application/engine/common/.../TQPersister.kt                       — запись результатов
8. application/engine/common/.../TQProcessExceptionProcessor.kt       — обработка ошибок
9. application/engine/common/.../TQApiService.kt                      — API для воркеров
10. db/changelog/014_add_new_indices_on_database.xml                  — индексы
```

---

## 4.10 DoD на этот файл

```text
[ ] Запустить тестовый Spring-проект с TQ и одним TQ-процессом из 3 задач (см. docs/intro.md «Пример настройки на Spring'е»).
[ ] Локально выполнить getAvailableTasks с EXPLAIN ANALYZE на dump-е реальной БД (или fake данных) — увидеть план запроса.
[ ] Замерить latency между шагами в DAG (от COMPLETED предка до IN_PROGRESS потомка) при interval=5s.
[ ] Записать результат сюда + в `06-practice/01-experiment-backlog.md` (Эксперимент 4 / 5).
```
