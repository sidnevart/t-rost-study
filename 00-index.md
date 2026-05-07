# Learning OS — A-P / Target / TQ

> Личная система обучения backend/platform engineer. Цель — предложить архитектурное улучшение системы сборки аудиторий, опираясь на реальный код **A-P (Auditory Platform)**, **Target** и **TQ (ninja-t-queue)**.

---

## 1. Цель

```text
Понять A-P / Target / TQ end-to-end настолько глубоко, чтобы:
- самостоятельно видеть, где система справляется, а где — нет
- формулировать собственные гипотезы об узких местах
- ставить эксперименты, которые гипотезы подтверждают или опровергают
- аргументированно предлагать изменения с числами и trade-off-ами
```

Конечная точка — твоя собственная гипотеза, проверенная экспериментом, оформленная в RFC.
Что именно это будет — ты не знаешь заранее. Это и есть суть работы.

---

## 2. Как пользоваться

1. Открой `01-context/` и прочитай в порядке нумерации. Это карта системы — то, что **точно видно** в коде, и **что нужно уточнить**.
2. Открой `02-architecture/` — там разобраны архитектурные модели и альтернативы DAG-подходу.
3. Веди работу по `05-weekly-sprints/` — 60–90 минут в день, каждый день заканчивается артефактом.
4. Книги читай параллельно по плану `03-books/01-reading-plan-12-months.md`. После каждой книги — артефакт, привязанный к проектам.
5. Эксперименты — из `06-practice/01-experiment-backlog.md`. Используй шаблоны RFC и benchmark.
6. Финал — `07-final-rfc/01-final-rfc-outline.md`.

**GPT:** для каждого трека и спринта есть готовые промпты в `gpt-prompts.md` по 4-шаговой системе (Learning Curator → Weekly Sprint → Deep Explanation → Self-check). Открывай при старте трека или когда застрял.

**Принцип:** ни один день не заканчивается «прочитал». Только конкретный артефакт: бенчмарк, query, RFC-фрагмент, domain map, decision note.

---

## 3. Карта файлов

### 01-context — что есть в проектах
- [01-projects-overview.md](01-context/01-projects-overview.md) — три проекта, стек, модули, ключевые файлы
- [02-ap-domain-map.md](01-context/02-ap-domain-map.md) — A-P как платформа, не как очередь
- [03-target-domain-map.md](01-context/03-target-domain-map.md) — Target и его контракт
- [04-tq-domain-map.md](01-context/04-tq-domain-map.md) — TQ: process, task, scheduler, координация
- [05-end-to-end-audience-flow.md](01-context/05-end-to-end-audience-flow.md) — путь аудитории от запроса до доставки
- [06-compute-export-delivery-split.md](01-context/06-compute-export-delivery-split.md) — почему «аудитория за 2–3 секунды» — обманчивая метрика
- [07-open-questions-for-team.md](01-context/07-open-questions-for-team.md) — что нужно уточнить у команды

### 02-architecture — модели и альтернативы
- [01-audience-platform-model.md](02-architecture/01-audience-platform-model.md)
- [02-target-contract.md](02-architecture/02-target-contract.md)
- [03-tq-role-in-system.md](02-architecture/03-tq-role-in-system.md)
- [04-graph-vs-sql-vs-bitmap-vs-cache.md](02-architecture/04-graph-vs-sql-vs-bitmap-vs-cache.md) — **ключевой файл**
- [05-execution-router-planner.md](02-architecture/05-execution-router-planner.md)
- [06-result-registry-model.md](02-architecture/06-result-registry-model.md)
- [07-freshness-snapshot-model.md](02-architecture/07-freshness-snapshot-model.md)
- [08-observability-model.md](02-architecture/08-observability-model.md)

### 03-books — план чтения 12 месяцев
- [01-reading-plan-12-months.md](03-books/01-reading-plan-12-months.md)
- [02-engineering-books.md](03-books/02-engineering-books.md)
- [03-business-product-books.md](03-books/03-business-product-books.md)
- [04-how-to-read-books-with-project-practice.md](03-books/04-how-to-read-books-with-project-practice.md)

### 04-learning-tracks — 12 треков
- [01-audience-platform-domain.md](04-learning-tracks/01-audience-platform-domain.md)
- [02-target-integration.md](04-learning-tracks/02-target-integration.md)
- [03-tq-kotlin-coroutines.md](04-learning-tracks/03-tq-kotlin-coroutines.md)
- [04-postgresql-queues-listen-notify.md](04-learning-tracks/04-postgresql-queues-listen-notify.md)
- [05-distributed-coordination.md](04-learning-tracks/05-distributed-coordination.md)
- [06-dag-algorithms-and-critical-path.md](04-learning-tracks/06-dag-algorithms-and-critical-path.md)
- [07-clickhouse-performance.md](04-learning-tracks/07-clickhouse-performance.md)
- [08-data-locality-s3-pushdown.md](04-learning-tracks/08-data-locality-s3-pushdown.md)
- [09-bitmap-inverted-index-serving.md](04-learning-tracks/09-bitmap-inverted-index-serving.md)
- [10-cache-incremental-cdc.md](04-learning-tracks/10-cache-incremental-cdc.md)
- [11-jvm-go-performance.md](04-learning-tracks/11-jvm-go-performance.md)
- [12-business-product-sales-leadership.md](04-learning-tracks/12-business-product-sales-leadership.md)
- [13-networks.md](04-learning-tracks/13-networks.md)

### 05-weekly-sprints — 8 недель
- [sprint-01-ap-end-to-end-map.md](05-weekly-sprints/sprint-01-ap-end-to-end-map.md) — система целиком
- [sprint-02-target-contract.md](05-weekly-sprints/sprint-02-target-contract.md)
- [sprint-03-compute-export-delivery.md](05-weekly-sprints/sprint-03-compute-export-delivery.md)
- [sprint-04-graph-vs-alternatives.md](05-weekly-sprints/sprint-04-graph-vs-alternatives.md)
- [sprint-05-tq-coroutines.md](05-weekly-sprints/sprint-05-tq-coroutines.md)
- [sprint-06-postgres-queue.md](05-weekly-sprints/sprint-06-postgres-queue.md)
- [sprint-07-clickhouse-baseline.md](05-weekly-sprints/sprint-07-clickhouse-baseline.md)
- [sprint-08-bitmap-experiment.md](05-weekly-sprints/sprint-08-bitmap-experiment.md)

### 06-practice
- [01-experiment-backlog.md](06-practice/01-experiment-backlog.md)
- [02-benchmark-template.md](06-practice/02-benchmark-template.md)
- [03-rfc-template.md](06-practice/03-rfc-template.md)
- [04-self-check-questions.md](06-practice/04-self-check-questions.md)
- [05-artifact-checklist.md](06-practice/05-artifact-checklist.md)

### 07-final-rfc
- [01-final-rfc-outline.md](07-final-rfc/01-final-rfc-outline.md)
- [02-senior-initiative-dod.md](07-final-rfc/02-senior-initiative-dod.md)

---

## 4. Откуда начать

Нет предписанного порядка. Есть логичная последовательность:

```text
1. 01-context/ — прочитай факты о системе, которые видны в коде.
   Цель: понять, что вообще существует. Не делать выводов — только факты.

2. 02-architecture/ — прочти frameworks мышления.
   Цель: получить язык, чтобы думать об архитектуре. Не искать ответы — задать вопросы.

3. 04-learning-tracks/ — учись параллельно с погружением.
   Треки 01-05 — фундамент (домен, контракт, TQ, PG, distributed).
   Треки 06-13 — инструменты и глубина.
   Выбирай трек, который отвечает на вопрос, который у тебя прямо сейчас.

4. 05-weekly-sprints/ — структурированное погружение.
   8 недель практики. Каждый спринт заканчивается артефактом из реального кода.

5. 06-practice/ — ставь эксперименты, когда появятся наблюдения.
   Не раньше. Эксперимент без наблюдения — «просто попробуем».

6. 07-final-rfc/ — оформи гипотезу в RFC, когда есть числа.
```

Читай `05-weekly-sprints/sprint-01` как точку входа, если не знаешь откуда начать.

---

## 5. Артефакты, которые ты будешь создавать

Ты сам заполняешь эти файлы числами и наблюдениями из реального кода.

```text
1. 01-context/02-09-*.md         — твои личные наблюдения из кода (заметки в каждом файле)
2. 06-practice/01-experiment-backlog.md — твои эксперименты, по одному шаблону
3. 06-practice/02-benchmark-template.md — твои числа из профилирования
4. 07-final-rfc/                 — твоя гипотеза, когда она появится из данных
```

Что именно это будет — появится из наблюдений. Если ты уже знаешь ответ до экспериментов — ты работаешь с confirmation bias, не с инженерным процессом.

---

## 6. Senior DoD

См. подробно [07-final-rfc/02-senior-initiative-dod.md](07-final-rfc/02-senior-initiative-dod.md).

Кратко: финал готов, когда:

```text
1. Я могу за 10 минут на доске нарисовать end-to-end путь любой аудитории по реальному коду.
2. У меня есть бенчмарк compute/export/delivery с разбивкой времени.
3. Есть decision table: «когда DAG, когда SQL, когда bitmap, когда cache».
4. Есть RFC с явным rollout/rollback и метриками успеха.
5. Все гипотезы либо подтверждены кодом, либо вынесены в `01-context/07-open-questions-for-team.md`.
```

---

## 7. Принципы стиля

- Просто. Без воды.
- Каждый файл = ответ на конкретный вопрос инженера.
- Нет фактов — пиши явно: «гипотеза» или «не найдено в коде, нужно уточнить».
- Каждый шаг заканчивается артефактом, не «прочитал».
