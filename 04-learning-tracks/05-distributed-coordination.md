# Track 05 — Distributed systems theory + coordination

## Зачем учить
A-P, Target и TQ — распределённая система. Несколько подов A-P, несколько подов Target, несколько нод TQ через одну Postgres. Если ты не знаешь теорию (CAP, consistency models, time, ordering, consensus, failure modes) — ты будешь принимать решения по интуиции, что в распределёнке всегда плохо.

## Что учить — фундамент

### A. Failure modes — реальная карта проблем
```text
1. Crash failure (нода умерла, не отвечает)
2. Omission failure (некоторые сообщения теряются)
3. Timing failure (медленная нода — выглядит как сбой)
4. Byzantine (узел врёт; обычно не проблема внутри своей инфры)
5. Partition (сетевое разбиение)
6. Slow leader / slow node (хуже crash — не очевидно)
7. Clock skew (часы рассинхронизированы)
8. GC pauses, JIT compilation pauses, kernel-level stalls
```

«Network is reliable» — главное заблуждение. Прочти fallacies of distributed computing (Peter Deutsch).

### B. Consistency models
```text
1. Linearizability (strongest)
2. Sequential consistency
3. Causal consistency
4. Read-your-writes
5. Eventual consistency
6. Session guarantees
```

Каждая model = trade-off latency / availability. CAP theorem: при partition можно выбрать либо C, либо A. PACELC расширяет: even WITHOUT partition есть trade-off latency vs consistency.

### C. Time in distributed systems
```text
1. Why physical clocks нельзя доверять (NTP drift, leap seconds)
2. Lamport timestamps — logical ordering без физического времени
3. Vector clocks — partial order, может выявить concurrency
4. Hybrid logical clocks (HLC) — physical + logical
5. TrueTime (Spanner) — физика с bounded uncertainty
```

Lamport's «Time, Clocks, and Ordering of Events» (1978) — обязательное чтение, 11 страниц.

### D. Coordination primitives
```text
1. Heartbeat: liveness signal, TTL
2. Lease: тот же heartbeat + ownership
3. Fencing token: монотонная versions для защиты от split-brain
4. Lock service (etcd, Zookeeper, Redis Redlock — последний под вопросом)
5. Leader election: Raft / Paxos / Zab — алгоритмы консенсуса
6. Quorum: majority writes/reads
7. Consensus: agreement on a single value across N nodes
```

В TQ — heartbeat через `tq_node` + 5min reservation timeout. Это упрощённая coordination (нет fencing, нет consensus). Это работает в нашей нагрузке, но имеет границы.

### E. Replication
```text
1. Single-leader (master-slave)
2. Multi-leader (с конфликтами и их разрешением)
3. Leaderless (Cassandra, Dynamo) — кворумы, hinted handoff, anti-entropy
4. Sync vs async replication trade-offs
5. Replication lag и monotonic-read проблемы
```

### F. Saga и distributed transactions
```text
1. Two-phase commit (2PC) — блокирующий, не used в реальности часто
2. Saga: цепочка локальных транзакций + compensating actions
3. Outbox pattern: гарантированная публикация event-а после commit
4. Idempotency receivers — работают только с at-least-once
5. CRDTs — eventually consistent merging
```

## Что почитать

| Источник | Что брать |
|---|---|
| **DDIA** (Kleppmann) — гл. 5, 7, 8, 9 | replication, transactions, troubles, consistency |
| **Designing Distributed Systems** (Burns) | патерны, контейнерные сервисы |
| **Lamport's papers** | "Time, Clocks..." и Paxos Made Simple |
| **The CAP FAQ** (Henry Robinson) | устранение распространённых заблуждений |
| **"Notes on Distributed Systems for Young Bloods"** (Jeff Hodges) | reality-check, must-read |
| **Aphyr's Jepsen blog** (jepsen.io) | как реальные системы ломаются |
| **"There Is No Now"** (Justin Sheehy, ACM Queue) | про время в системах |
| **Brendan Burns talks** | container patterns |
| **etcd docs** + **Raft paper** (Ongaro & Ousterhout) | consensus в practice |
| **Streaming Systems** (Akidau) | consistency в потоковом мире |
| **Honest Apology series** (Aphyr) | разборы systems-level багов |

## GPT-ускоритель

**Шаг 1 — Learning Curator** → вставь из `gpt-prompts.md` → Track 05 → Шаг 1.
**Шаг 3 — Deep Explanation** (если застрял на Saga pattern) → вставь из `gpt-prompts.md` → Track 05 → Шаг 3.
**Шаг 4 — Self-check** → вставь из `gpt-prompts.md` → Track 05 → Шаг 4.

---

## Что сделать руками

### Задача 1 — Failure injection: kill -9 в IN_PROGRESS (обязательно)
```bash
# 1. Запустить 3 процесса TQ с одним PG
# 2. Создать задачу, дать уйти в IN_PROGRESS
# 3. kill -9 <pid_node_with_task>
# 4. Замерить через sql: когда задача вернётся в PENDING?
SELECT id, status, node_id, updated_at 
FROM tq_task 
WHERE status = 'IN_PROGRESS'
ORDER BY updated_at;
```
- Артефакт: «Recovery time = X секунд. Верхняя граница = heartbeat_ttl + interval».
- Найти в коде, где именно происходит возврат задачи (file:line).

### Задача 2 — Split-brain симуляция
- Запустить 2 ноды TQ, у одной — заблокировать UPDATE в `tq_node` (через `iptables` или mock).
- Что произойдёт с задачами этой ноды?
- Есть ли fencing token в TQ? Найти в коде или подтвердить отсутствие.
- Вывод: при каком сценарии одна задача может выполниться дважды?

### Задача 3 — Saga прототип (coding, ~100 строк Go/Kotlin)
Написать минимальный Saga orchestrator для сценария «audience loading»:
```
Step 1: создать TQ-process (compensating: cancel process)
Step 2: дождаться Kafka-результата (compensating: noop, результат уже отправлен)
Step 3: обновить статус в A-P (compensating: rollback status)
```
Прогнать: что происходит если Step 2 таймаутится?
- Артефакт: код + описание «где неявная Saga в текущем коде A-P/Target».

### Задача 4 — Leader election: audit текущего решения
- Найти в A-P код leader election для cron-задач.
- Написать list: «что гарантирует / не гарантирует текущее решение».
- Написать сценарий при котором две A-P-ноды одновременно запустят одну cron-задачу.
- Есть ли защита? Что произойдёт?

### Задача 5 — At-least-once + idempotency audit
- Найти все места в коде, где используется at-least-once delivery (Kafka consumer).
- Для каждого — проверить: есть ли idempotency защита?
- Написать: «для каких операций дублирование безопасно, для каких — нет».

### Задача 6 — Lamport paper (чтение с применением)
- Прочитать «Time, Clocks, and the Ordering of Events» (11 страниц).
- Применить к `tq_task` events: как Lamport timestamp помог бы в debugging?
- Написать: «где в нашей системе ordering важен, где — нет».

## Self-check

```text
1. CAP — что выбирает Postgres-based queue в TQ под partition?
2. Чем sequential consistency отличается от linearizability?
3. Что такое fencing token и где у нас он отсутствует?
4. Почему Lamport timestamp недостаточно для concurrent writes detection?
5. Что произойдёт, если две ноды одновременно станут leader-ом cron-задачи?
6. Saga vs 2PC — в чём принципиальная разница?
7. Что такое at-least-once + idempotent receiver = exactly-once? Где эта эмуляция используется у нас?
8. Чем eventually consistent отличается от strong eventual consistency (CRDTs)?
9. Что значит "monotonic read" guarantee и где она нам важна?
10. Какова стоимость consensus (Raft) — почему мы его не используем?
```

## DoD

```text
[ ] Я могу прочитать любой post-mortem про распределённый сбой и говорить про него на одном языке.
[ ] У меня есть свой эксперимент с failure injection в локальном TQ.
[ ] Я понимаю, что упрощено в нашей системе по сравнению с full-fledged distributed system.
[ ] Я могу аргументированно сказать, нужен ли нам Raft/etcd для какой-то задачи.
[ ] Прочитан Lamport paper.
```
