# Мастер-промпт для Claude/Codex: Learning OS по A-P / Target / TQ

```md
# Задача

Ты работаешь как мой learning architect и technical curator.

У тебя есть доступ к трём проектам, связанным с системой сборки аудиторий:

1. A-P / Auditory Platform / Audience Platform
2. Target / targeting / downstream-система, которая использует аудитории
3. TQ / Task Queue / execution layer для процессов и задач

Твоя задача — изучить эти проекты по коду, документации, README, конфигам, миграциям, SQL, тестам и существующим docs, а затем создать для меня полноценную систему обучения в виде набора Markdown-файлов.

Все материалы нужно положить в папку:

```text
documents/learning-os
```

Если папки нет — создай её.

---

# Главная цель learning-os

Я хочу не просто читать книги, а через реальные проекты вырасти как backend/platform engineer.

Моя практическая цель:

```text
Понять A-P / Target / TQ end-to-end и составить программу обучения, которая поможет мне предложить архитектурное улучшение системы сборки аудиторий:
- ускорить compute
- понять, где bottleneck
- понять, нужен ли graph/DAG-подход
- понять, где лучше SQL / bitmap / cache / precompute / serving model
- понять, как Target реально потребляет аудитории
- не сломать guarantees TQ
```

Важно: не делай план только вокруг TQ.  
TQ — это execution layer, а не вся система.

Нужно смотреть шире:

```text
A-P / Audience Platform
→ audience definition
→ planning/routing
→ execution
→ result storage
→ export
→ Target delivery
→ observability
```

---

# Мой контекст

Я backend engineer.

Мой стек:

```text
Kotlin
Java
Go
Spring Boot
PostgreSQL
ClickHouse
Kafka
Kubernetes
S3
Microservices
```

Мне нужно учиться через практику на реальных проектах.

Я хочу читать минимум 2 книги в месяц:

```text
1 engineering book
1 business/product/management book
```

Но книги — это не самоцель.  
Главное — рабочие артефакты:

```text
RFC
benchmark
architecture note
domain map
experiment report
query analysis
target contract
execution model comparison
```

---

# Важное ограничение

Не придумывай архитектуру по эвристикам.

Сначала изучи проекты и явно отделяй:

```text
1. Что точно видно из кода/документов
2. Что является гипотезой
3. Что нужно проверить вопросами к команде
```

Если информации не хватает — так и пиши:

```text
"В проекте не найдено достаточно данных, нужно уточнить..."
```

Не выдумывай внутренние детали.

---

# Что нужно сделать

## Шаг 1. Изучи проекты

Посмотри:

```text
README
docs
documents
src/main
src/test
migrations
SQL
configs
application.yaml
Docker/K8s manifests
CI
packages/modules
domain classes
services
repositories
task/process models
Target integration code
ClickHouse queries
S3 usage
Kafka usage
```

Найди:

```text
- где описываются аудитории
- где создаются процессы сборки
- где создаются task-и
- где находится TQ
- как task-и исполняются
- где используются PostgreSQL / ClickHouse / S3
- как результат аудитории хранится
- как результат уходит в Target
- какие SLA/ограничения/метрики видны из кода или docs
```

---

# Ключевой архитектурный вопрос

Особое внимание удели вопросу:

```text
А что если graph/DAG-подход для сборки аудитории плох как универсальная модель?
```

Нужно не просто оптимизировать TQ, а проверить:

```text
1. Где DAG/TQ действительно нужен
2. Где DAG/TQ является overhead
3. Где можно заменить цепочку task-ов одним ClickHouse SQL
4. Где можно использовать bitmap / inverted index
5. Где нужен cache
6. Где можно precompute popular segments
7. Где compute и export нужно разделить
8. Где Target может принимать reference/result_id вместо полного client_id list
```

---

# Итоговая структура файлов

Создай примерно такую структуру:

```text
documents/learning-os/
  00-index.md

  01-context/
    01-projects-overview.md
    02-ap-domain-map.md
    03-target-domain-map.md
    04-tq-domain-map.md
    05-end-to-end-audience-flow.md
    06-compute-export-delivery-split.md
    07-open-questions-for-team.md

  02-architecture/
    01-audience-platform-model.md
    02-target-contract.md
    03-tq-role-in-system.md
    04-graph-vs-sql-vs-bitmap-vs-cache.md
    05-execution-router-planner.md
    06-result-registry-model.md
    07-freshness-snapshot-model.md
    08-observability-model.md

  03-books/
    01-reading-plan-12-months.md
    02-engineering-books.md
    03-business-product-books.md
    04-how-to-read-books-with-project-practice.md

  04-learning-tracks/
    01-audience-platform-domain.md
    02-target-integration.md
    03-tq-kotlin-coroutines.md
    04-postgresql-queues-listen-notify.md
    05-distributed-coordination.md
    06-dag-algorithms-and-critical-path.md
    07-clickhouse-performance.md
    08-data-locality-s3-pushdown.md
    09-bitmap-inverted-index-serving.md
    10-cache-incremental-cdc.md
    11-jvm-go-performance.md
    12-business-product-sales-leadership.md

  05-weekly-sprints/
    sprint-01-ap-end-to-end-map.md
    sprint-02-target-contract.md
    sprint-03-compute-export-delivery.md
    sprint-04-graph-vs-alternatives.md
    sprint-05-tq-coroutines.md
    sprint-06-postgres-queue.md
    sprint-07-clickhouse-baseline.md
    sprint-08-bitmap-experiment.md

  06-practice/
    01-experiment-backlog.md
    02-benchmark-template.md
    03-rfc-template.md
    04-self-check-questions.md
    05-artifact-checklist.md

  07-final-rfc/
    01-final-rfc-outline.md
    02-senior-initiative-dod.md
```

Если после изучения проектов поймёшь, что структура должна быть чуть другой — можно адаптировать, но обязательно сохрани смысловые разделы:

```text
context
architecture
books
learning tracks
weekly sprints
practice
final RFC
```

---

# Что должно быть в 00-index.md

Сделай главную навигационную страницу.

В ней должны быть:

```text
1. Цель learning-os
2. Как пользоваться
3. Карта файлов
4. Первые 4 недели
5. Главные артефакты
6. Senior DoD
7. Ссылки на все остальные md-файлы
```

Формат ссылок — относительные markdown-ссылки.

Пример:

```md
- [A-P Domain Map](01-context/02-ap-domain-map.md)
- [Target Contract](02-architecture/02-target-contract.md)
```

---

# Что должно быть в context-файлах

## 01-projects-overview.md

Опиши три проекта:

```text
- что делает проект
- какую роль играет в системе
- какие основные модули нашёл
- какие технологии используются
- какие важные классы/таблицы/конфиги нашёл
- какие вопросы остались
```

## 02-ap-domain-map.md

Опиши A-P не как TQ, а как платформу:

```text
- audience definition
- audience lifecycle
- planner/routing
- execution
- result
- export
- observability
```

## 03-target-domain-map.md

Опиши Target:

```text
- что Target принимает
- как получает аудитории
- client_id list vs segment_id/result_id/reference
- SLA
- freshness
- retry/idempotency
- delivery confirmation
```

Если этого нет в коде — напиши, что нужно уточнить.

## 04-tq-domain-map.md

Опиши TQ:

```text
- process
- task
- dependency graph
- scheduler
- worker
- statuses
- retries
- failure handling
- concurrency
- node coordination
```

## 05-end-to-end-audience-flow.md

Нарисуй путь аудитории:

```text
business request
→ audience definition
→ process/task creation
→ execution
→ intermediate results
→ final result
→ export
→ Target delivery
→ completion/reporting
```

Для каждого шага укажи:

```text
- где код
- какие данные
- какая система
- потенциальный bottleneck
- что непонятно
```

## 06-compute-export-delivery-split.md

Раздели:

```text
compute time
export time
delivery time
total lead time
```

Объясни, почему "аудитория за 2–3 секунды" может значить разные вещи.

---

# Что должно быть в architecture-файлах

## 04-graph-vs-sql-vs-bitmap-vs-cache.md

Обязательно разложи:

```text
DAG/TQ path
One ClickHouse SQL path
Bitmap/index path
Cache path
Precompute path
Incremental/CDC path
```

Для каждого:

```text
- когда подходит
- когда не подходит
- какие плюсы
- какие риски
- какие данные нужны
- какие эксперименты провести
```

Сделай decision table:

```md
| Use-case | DAG/TQ | One SQL | Bitmap | Cache | Precompute | Incremental |
|---|---|---|---|---|---|---|
```

## 05-execution-router-planner.md

Спроектируй гипотетический Planner/Router:

```text
Audience Definition
        ↓
Planner / Router
        ↓
Fast path / SQL path / DAG path / Cache path
        ↓
Result Registry
        ↓
Target Delivery
```

Важно: это может быть гипотеза. Если в коде такого нет — явно напиши, что это proposal.

---

# Что должно быть в books

Составь 12-месячный reading plan.

Каждый месяц:

```text
- engineering book
- business/product/management book
- зачем эта книга
- какие главы приоритетны
- какой практический артефакт сделать после чтения
```

Обязательно включи или проверь уместность книг:

Engineering:

```text
Kotlin Coroutines Deep Dive — Marcin Moskała
Designing Data-Intensive Applications — Martin Kleppmann
PostgreSQL: Up and Running — Regina Obe, Leo Hsu
Database Internals — Alex Petrov
Systems Performance — Brendan Gregg
SQL Performance Explained — Markus Winand
Software Architecture: The Hard Parts — Neal Ford et al.
Enterprise Integration Patterns — Hohpe, Woolf
Streaming Systems — Akidau, Chernyak, Lax
Introduction to Algorithms — CLRS, Graph Algorithms
Java Concurrency in Practice — Brian Goetz
The Staff Engineer’s Path — Tanya Reilly
```

Business/Product:

```text
The Mom Test — Rob Fitzpatrick
Continuous Discovery Habits — Teresa Torres
Inspired — Marty Cagan
Obviously Awesome — April Dunford
The Sales Acceleration Formula — Mark Roberge
Good Strategy / Bad Strategy — Richard Rumelt
High Output Management — Andrew Grove
Crossing the Chasm — Geoffrey Moore
The Startup Owner’s Manual — Steve Blank, Bob Dorf
Team Topologies — Skelton, Pais
Empowered — Marty Cagan
The Hard Thing About Hard Things — Ben Horowitz
```

Не просто списком.  
Для каждой книги объясни:

```text
1. Почему она нужна именно мне
2. Как связана с A-P / Target / TQ
3. Что сделать руками после чтения
```

---

# Что должно быть в learning tracks

Каждый learning track должен иметь одинаковую структуру:

```md
# Название темы

## Зачем учить

## Где это есть в проектах

Если найдено в коде:
- файл/класс/модуль
- что делает
- почему важно

Если не найдено:
- что нужно уточнить

## Что изучить

## Практика на проекте

## Артефакты

## Self-check questions

## DoD
```

Темы обязательно:

```text
Audience Platform domain
Target integration
TQ / Kotlin Coroutines
PostgreSQL queues / LISTEN NOTIFY / SKIP LOCKED
Distributed coordination
DAG / topological sort / critical path
ClickHouse profiling
Predicate pushdown / S3 roundtrip
Bitmap / inverted index / audience serving
Cache / incremental / CDC
JVM / Go performance
Business / product / sales / leadership
```

---

# Что должно быть в weekly sprints

Сделай минимум 8 weekly sprint-файлов.

Каждый sprint:

```md
# Sprint N — Название

## Цель недели

## Почему это важно

## День 1
- Цель
- Что прочитать
- Что найти в коде
- Практика
- Артефакт
- DoD

## День 2
...

## Итог недели

## Финальный артефакт

## Self-check
```

Ограничения:

```text
- 60–90 минут в день
- не больше 1–2 ресурсов в день
- каждый день заканчивается артефактом
- практика должна быть связана с реальными проектами
```

Первые 4 sprint должны быть не про TQ, а про A-P/Target/system model:

```text
Sprint 1 — A-P end-to-end map
Sprint 2 — Target contract
Sprint 3 — Compute vs Export vs Delivery
Sprint 4 — Graph vs SQL vs Bitmap vs Cache
```

Потом уже:

```text
Sprint 5 — TQ / Coroutines
Sprint 6 — PostgreSQL queue / LISTEN NOTIFY
Sprint 7 — ClickHouse baseline
Sprint 8 — Bitmap experiment
```

---

# Что должно быть в practice

## experiment-backlog.md

Сделай backlog экспериментов.

Формат:

```md
# Experiment: Название

## Гипотеза

## Где в проекте

## Что сделать

## Метрики

## Как проверить correctness

## Риски

## Ожидаемый результат

## DoD
```

Эксперименты:

```text
1. DAG/TQ vs one ClickHouse SQL
2. current pipeline vs bitmap expression
3. S3 roundtrip removal
4. LISTEN/NOTIFY push + poll fallback
5. parallel DAG execution
6. cache by audience definition hash
7. precomputed popular segments
8. compute/export/delivery breakdown
9. Target reference/result_id possibility
10. ClickHouse query baseline
```

## rfc-template.md

Сделай шаблон RFC для итоговой инициативы:

```text
RFC: Как ускорить сборку аудиторий без поломки guarantees
```

Включи:

```text
- problem
- current architecture
- baseline
- bottlenecks
- alternatives
- decision table
- recommended path
- rollout plan
- rollback plan
- metrics
- open questions
```

---

# Финальный результат

После выполнения у меня должна появиться папка:

```text
documents/learning-os
```

В ней должен быть полноценный markdown learning system.

Я должен открыть:

```text
documents/learning-os/00-index.md
```

И оттуда попасть во все материалы.

---

# Критерии готовности

Работа готова, если:

```text
1. Создана папка documents/learning-os
2. Есть 00-index.md с навигацией
3. Есть context по A-P, Target, TQ
4. Есть честное разделение known facts / hypotheses / open questions
5. Есть 12-месячный план книг
6. Есть learning tracks
7. Есть минимум 8 weekly sprints
8. Есть experiment backlog
9. Есть RFC template
10. Главный фокус не только TQ, а вся A-P end-to-end
11. Отдельно разобран вопрос: может ли graph/DAG-подход быть плохим
12. Отдельно разобран Target contract
13. Все файлы связаны относительными markdown-ссылками
```

---

# Стиль

Пиши на русском.

Стиль:

```text
- просто
- понятно
- без воды
- как для backend engineer, который хочет применить это на работе
- с конкретными практическими действиями
- с DoD
- с артефактами
```

Не надо академической воды.

Плохой стиль:

```text
"Изучить распределённые системы и повысить надёжность."
```

Хороший стиль:

```text
"Найти, где task переходит в RESERVED/RUNNING, проверить транзакцию, понять, может ли две ноды взять одну task, написать tq-delivery-guarantees.md."
```

---

# Важный финальный шаг

После создания файлов выведи в ответ:

```text
1. Какие проекты были изучены
2. Какие ключевые файлы/модули найдены
3. Что создано в documents/learning-os
4. Какие гипотезы появились
5. Какие вопросы нужно задать команде
6. С какого sprint мне начать
```

Не ограничивайся общими словами.  
Дай конкретный summary по результатам исследования.
```
