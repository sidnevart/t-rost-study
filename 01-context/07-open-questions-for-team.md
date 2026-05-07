# 07. Открытые вопросы команде

> Честный список того, что **не виден из кода** или вызывает сомнение. Перед тем, как делать архитектурное предложение, его нужно проверить здесь.

Формат: `[категория] вопрос. что выясняем. кому задать (роль).`

---

## A) Контракт A-P ↔ Target

```text
A1. [Sync vs async] Когда A-P стартует Target — это синхронный REST с ожиданием
    process_id, или fire-and-forget? Что происходит при таймауте?
    → A-P backend lead

A2. [gathering_id] Уникальность и lifecycle: новый id на каждый перезапуск
    или один на запрос? Кто его генерирует — A-P или Target?
    → A-P backend lead

A3. [Idempotency] Гарантия дедупа `nfs-partner-platforms.internal.auditory`:
    по messageId, gathering_id, или вообще не гарантируется?
    Что если Target отправил один и тот же gathering_id дважды?
    → Target backend lead

A4. [Direction] Топик `nfs-partner-platforms.auditory` — вход в Target.
    Кто его пишет и зачем (refresh? подтверждения? cancel?)?
    → Target / A-P

A5. [Error semantics] Если `error != null` в kafka-сообщении —
    можно ли retry, или это терминальное состояние?
    → A-P + Target

A6. [Timeout] Сколько A-P ждёт результата от Target до того, как считать,
    что "пропало"? Где это конфигурируется?
    → A-P backend
```

---

## B) Размеры и нагрузка

```text
B1. p50/p95/max — число client_id в одной аудитории.
    → analytics + DB stats

B2. Сколько аудиторий в день создаётся / пересобирается?
    Из них: STATIC vs DYNAMIC, экспортируется в T-Segment vs нет.
    → product + analytics

B3. Какова реальная глубина и ширина DAG в Target?
    SELECT process_id, count(*) FROM tq_task GROUP BY process_id ORDER BY count(*) DESC LIMIT 20;
    → Target backend (или DB-доступ)

B4. Распределение таймингов по типам TQ-задач?
    SELECT type, percentile_cont(0.5)... FROM tq_task WHERE status='COMPLETED' ...
    → Target / DB

B5. Какие S3 retention настроены для каждого бакета (target-*, ms-auditory-*)?
    Может ли downstream читать reference без риска "файла уже нет"?
    → Infra / Target backend
```

---

## C) Источники данных и compute

```text
C1. Где compute реально считает client_id-ы — ClickHouse, Postgres, S3 файлы?
    Доминирующий источник?
    → Target backend

C2. Сколько разных источников трогает один gathering — 1, 5, 10?
    → Target backend

C3. Есть ли уже where-pushdown оптимизации к ClickHouse, или часть фильтров
    делается в Kotlin in-memory?
    → Target backend

C4. Есть ли кэш на уровне Target для popular-фильтров (например, "клиенты с подпиской PRO")?
    → Target backend

C5. Используется ли incremental compute (delta) для DYNAMIC-аудиторий, или каждый раз
    полный пересбор? `MATCHING_DELTA_EXCLUSION` — что именно использует?
    → Target backend
```

---

## D) SLA и ожидания заказчика

```text
D1. Какие реальные SLA у разных типов аудиторий (STATIC, DYNAMIC, push-кампания, отчётность)?
    → product + business stakeholders

D2. Что именно бизнес считает «готовая аудитория»: момент status=GATHERED,
    или момент, когда канал уже её разослал?
    → product

D3. Для каких use-cases допустимо stale (10 мин/1 час), а где надо real-time?
    → product

D4. Где сейчас violations SLA замечают раньше всего — какой канал жалуется?
    → product / on-call

D5. Поддержка cancel и pause: бизнес иногда хочет «не запускать,
    собирай только если важно» — есть ли такие сценарии?
    → product
```

---

## E) Хранение результатов

```text
E1. Что значит `auditory.s3_key` vs `t_segment_auditory.s3_key` —
    они хранят одно и то же или разные представления?
    → A-P backend

E2. Есть ли версионирование результатов? Если параметры аудитории не менялись,
    но был пересбор — мы пишем поверх или храним обе версии?
    → A-P backend

E3. Кому может пригодиться result_id reference (downstream, который сам читает)?
    Какие downstream-ы это поддержат, а какие нет?
    → Target / A-P / consumers (cb-platform, t-segment)

E4. Есть ли инструменты analytics поверх результатов
    (например, "сколько уникальных client_id попадают в обе аудитории X и Y")?
    → analytics
```

---

## F) Observability и SLO

```text
F1. Какие SLO заведены сейчас (compute time, delivery time, error rate)?
    Где они мониторятся?
    → SRE

F2. Есть ли correlation_id, который сквозит A-P → Target → A-P → T-Segment?
    Если нет — это первая вещь, которую стоит ввести.
    → A-P / Target backend

F3. Какие алерты сейчас работают и на что реально срабатывают
    (на компании-критичные кейсы или на инфраструктурные)?
    → SRE / on-call

F4. Где Grafana / Kibana дашборды на пайплайн аудиторий?
    → SRE / dev experience
```

---

## G) TQ specific

```text
G1. `maxRetryCount=1` — это «1 повтор» (=2 попытки) или «ровно 1 попытка»?
    Что в коде `TQApiService.kt` / `TQProcessExceptionProcessor.kt`?
    → ninja-team (TQ owners)

G2. Используется ли continuous=true в реальных процессах Target?
    Где его польза реально проверена?
    → Target backend

G3. Какова реальная нагрузка на tq_task в продакшене (rows, qps на SELECT)?
    Хватает ли индексов? Был ли тюнинг autovacuum?
    → SRE / DB

G4. Есть ли потребность в priority-deadline (а не статической priority)?
    Бывает, что задача низкого приоритета должна "проснуться" к дедлайну.
    → product / business

G5. Recurring tasks: какие конкретно реально живут в проде?
    SELECT type, cron FROM tq_recurring_task;
    → Target backend
```

---

## H) Архитектурное предложение

```text
H1. Если ввести Planner / Router (быстрый/средний/медленный путь),
    кто его «владеет»: A-P или новый сервис?
    → archtech / A-P lead

H2. Компетенции команды: знают ли все Kotlin Coroutines, ClickHouse,
    bitmap-структуры? Каков обучающий бюджет?
    → tech lead

H3. Какие части системы НЕЛЬЗЯ менять без больших обсуждений
    (контракты с T-Segment, Avro-схемы, downstream-каналы)?
    → archtech / Target lead

H4. Есть ли мораторий на breaking changes публичных контрактов?
    → archtech

H5. Что важнее на ближайший квартал: ускорить compute, ускорить delivery,
    или снизить error rate?
    → product / archtech
```

---

## Как пользоваться этим списком

```text
1. Когда какой-то вопрос ответили — записать ответ прямо в этот же файл,
   с датой и источником ("спросил X 2026-05-10").
2. Если ответ обнаруживается в коде — заменить на цитату file:line.
3. Если ответ переводит вопрос в гипотезу — оставить как открытое поле,
   но пометить «требует валидации».
4. В RFC ссылаться на конкретный пункт ("разбирали в open-questions H3").
```

---

## Скрипт «спросить по 30 минут в неделю»

Берём 5–7 вопросов в неделю, формируем список с приоритетом:

| Pri | Вопрос | Кому | Когда спросить |
|---|---|---|---|
| P0 | A1 — sync/async A-P→Target | A-P lead | первая 1:1 |
| P0 | D1 — SLA по типам | product | sync с PM |
| P0 | C1 — где compute считает | Target lead | sync |
| P1 | B3 — реальная DAG-структура | Target lead | по доступу к staging БД |
| P1 | E1 — двойной s3_key | A-P lead | в код-ревью пингануть |
| P2 | G2 — continuous реально используется? | ninja team | slack-вопрос |

Следующая неделя — следующая партия.
