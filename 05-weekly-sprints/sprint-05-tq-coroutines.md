# Sprint 05 — TQ / Kotlin Coroutines

## Цель недели
Понять TQScheduler.kt построчно. Научиться менять параметры TQ осознанно.

## Почему это важно
RFC будет затрагивать TQ. Без deep-dive — изменения будут «вслепую».

---

## GPT-ускоритель

**Перед началом спринта** → вставь из `gpt-prompts.md` → Track 03 → Шаг 2 (Weekly Sprint).
**Если застрял на suspend state machine** → вставь из `gpt-prompts.md` → Track 03 → Шаг 3 (Deep Explanation).
**В конце спринта** → вставь из `gpt-prompts.md` → Track 03 → Шаг 4 (Self-check).

---

## День 1 — DSL и API
- `application/api/common-api/.../main/TQ.kt`
- `application/api/common-api/.../main/TQTaskDsl.kt`
- `application/api/common-api/.../configuration/TQConfig.kt`

**Практика:** написать минимальный pet-project на TQ из 3 задач с зависимостями.

**Артефакт:** работающий pet-project (~200 строк Kotlin).

**DoD:** pet-project запускается, видны статусы в БД.

---

## День 2 — Scheduler core
- `application/engine/scheduler/core/.../TQScheduler.kt`
- Прочитать построчно. Понять, какая часть — Coroutines, какая — pure logic.

**Практика:** на pet-project поменять `interval` с 5s на 1s, замерить, что меняется.

**Артефакт:** аннотированная копия TQScheduler.kt в `practice/sprint-05/`.

**DoD:** могу за 10 минут пересказать главный цикл.

---

## День 3 — Heartbeat и recovery
- `application/engine/scheduler/heartbeat/.../TQHeartbeatService.kt`
- `releaseStaleAssignedTasks`, `markTimedOutStaleNewTasks`.

**Практика:** на pet-project запустить 3 ноды (3 процесса), убить одну в середине задачи, замерить, через сколько задача вернётся.

**Артефакт:** реальное число «recovery time после kill -9».

**DoD:** понимаю верхнюю границу recovery time.

---

## День 4 — Continuous processes
- `docs/API/core-processes/continuous-processes.md`
- Реализация cascade priority -1 (поиск в коде).

**Практика:** на pet-project включить `continuous=true`, замерить, насколько быстрее DAG-цепочка.

**Артефакт:** числа «с continuous vs без» для DAG depth=5.

**DoD:** имею количественный ответ.

---

## День 5 — Coroutines глубже
- Прочитать главу 1-3 *Kotlin Coroutines: Deep Dive* (Moskała).
- Структурная concurrency, SupervisorJob, CancellationException.

**Практика:** написать тест на TQ с virtual time (`runTest` из `kotlinx.coroutines.test`).

**Артефакт:** конспект по Coroutines + аннотированный тест.

**DoD:** уверенно ориентируюсь в Coroutines basics.

---

## Итог недели

```text
- Pet-project на TQ.
- Аннотированный TQScheduler.kt.
- Замеры (continuous, recovery time).
- Конспект по Coroutines.
```

## Финальный артефакт
`practice/sprint-05/` папка с pet-project + аннотациями.

## Self-check
1. Что произойдёт, если `interval` = 100ms? Какие риски?
2. Почему `SupervisorJob`, а не обычный `Job`?
3. Зачем нужен `taskReservationTimeout` в добавок к `taskExecutionTimeout`?
4. Каков верхний bound на recovery time после смерти ноды?
5. Continuous=true — когда не помогает (или мешает)?
