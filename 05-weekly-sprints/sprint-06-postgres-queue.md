# Sprint 06 — Postgres Queue / SKIP LOCKED / LISTEN-NOTIFY

## Цель недели
Понять, как именно PG обеспечивает очередь в TQ. Уметь оценить пропускную способность и узкие места.

## Почему это важно
Без понимания PG-механики все рассуждения про «увеличить maxConcurrentTasks» — поверхностные.

---

## GPT-ускоритель

**Перед началом спринта** → вставь из `gpt-prompts.md` → Track 04 → Шаг 2 (Weekly Sprint).
**Если застрял на MVCC + SKIP LOCKED** → вставь из `gpt-prompts.md` → Track 04 → Шаг 3 (Deep Explanation).
**В конце спринта** → вставь из `gpt-prompts.md` → Track 04 → Шаг 4 (Self-check).

---

## День 1 — FOR UPDATE SKIP LOCKED семантика
- Прочитать главу про MVCC из *Database Internals*.
- `TQJdbiTaskDao.kt:160-191` — `getAvailableTasks` с SKIP LOCKED.
- Запустить 4 параллельных psql, протестировать `SELECT … FOR UPDATE SKIP LOCKED`.

**Практика:** написать минимальный Go/Kotlin pull-pattern с SKIP LOCKED, замерить throughput.

**Артефакт:** числа throughput vs число потребителей.

**DoD:** понимаю, как масштабируется SKIP LOCKED.

---

## День 2 — Партиальные индексы
- `db/changelog/014_add_new_indices_on_database.xml`.
- На локальной PG: DROP INDEX → EXPLAIN → CREATE INDEX → EXPLAIN.

**Практика:** замер до/после для `getAvailableTasks` запроса на 1M tq_task строк.

**Артефакт:** EXPLAIN ANALYZE до/после в `practice/sprint-06/`.

**DoD:** понимаю, какой index выбран и почему.

---

## День 3 — Autovacuum и dead tuples
- Queue-pattern = много UPDATE → много dead tuples.
- Прочитать про HOT updates, fillfactor.

**Практика:** на 1M строк прогнать 100k UPDATE, замерить bloat. Запустить VACUUM ANALYZE.

**Артефакт:** замер «bloat и его рост».

**DoD:** есть гипотеза про autovacuum tuning.

---

## День 4 — LISTEN/NOTIFY
- Демо-приложение: producer пишет + NOTIFY, consumer слушает + fallback poll.
- Понять лимиты: 8000 байт payload, no persistence.

**Практика:** написать demo на 150 строк (Go/Kotlin).

**Артефакт:** репозиторий + замер latency LISTEN vs poll(5s).

**DoD:** имею альтернативу cron-poll-у в A-P.

---

## День 5 — Сравнение и выводы
- LISTEN/NOTIFY vs SKIP LOCKED vs обычный poll — где какой уместен.
- Применить к `LoadAuditoryRequestProcessingJob` в A-P (cron-poll сейчас).

**Практика:** прототип «LISTEN/NOTIFY вместо cron-poll» для A-P (не для production, для понимания).

**Артефакт:** заметка с рекомендацией + риски.

**DoD:** в RFC можно осознанно говорить про push-event vs poll.

---

## Итог недели

```text
- Демо SKIP LOCKED + замеры.
- EXPLAIN до/после партиальных индексов.
- LISTEN/NOTIFY демо.
- Заметка по queue-pattern в PG.
```

## Финальный артефакт
`practice/sprint-06-pg-queue/` — все эксперименты в одном месте.

## Self-check
1. Когда HOT update применяется и почему он критичен для queue?
2. Чем NOTIFY отличается от Kafka topic в плане durability?
3. Партиальный индекс — выигрывает только в скорости, или ещё в размере?
4. SKIP LOCKED + 100 нод — где упирается в потолок?
5. autovacuum для tq_task — какие настройки имеет смысл тюнить?
