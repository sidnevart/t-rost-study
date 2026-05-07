# 02. Engineering Books

> Под каждую — почему именно мне, как связано с A-P/Target/TQ, что сделать руками.

---

## DDIA — *Designing Data-Intensive Applications*, Martin Kleppmann

### Почему мне
Я работаю с системой, где переплетены batch (CH-запросы), pull (REST/poll) и push (Kafka). DDIA — единственный учебник, который правильно даёт всю эту мозаику в одном языке.

### Связь с проектами
- Глава 3 (Storage) — почему PG + CH у нас. Когда CH выгоден.
- Глава 5 (Replication) — почему `tq_node` с heartbeat достаточно вместо Raft.
- Глава 7 (Transactions) — `FOR UPDATE SKIP LOCKED` в TQ.
- Глава 11 (Stream Processing) — Kafka-flow A-P↔Target↔T-Segment, exactly-once vs at-least-once.

### Артефакты
- Записать в `04-learning-tracks/06-dag-algorithms-and-critical-path.md`, как DAG-резолюция в TQ соотносится с моделью dataflow из DDIA.
- В `02-architecture/02-target-contract.md` — добавить раздел «delivery semantics in DDIA terms».

---

## *SQL Performance Explained*, Markus Winand

### Почему мне
Каждый раз, когда «медленный SQL» — я хочу понимать почему, а не угадывать. Книга короткая (~200 стр), но именно те 200, которые нужны.

### Связь
- Прямо к ClickHouse / Postgres queries в A-P.
- Index-design для `tq_task` (см. миграцию `014_add_new_indices_on_database.xml`).
- Понять, почему партиальные индексы там работают как «подзеркало для FOR UPDATE SKIP LOCKED».

### Артефакты
- `06-practice/01-experiment-backlog.md` №10 — query plan для одного реального CH-запроса в Target.
- Discovery: на каких запросах в A-P сейчас нет покрывающих индексов.

---

## *Database Internals*, Alex Petrov

### Почему мне
Все мои гипотезы про PG в TQ (B-tree, MVCC, vacuum) — пока интуитивные. Эта книга превратит интуицию в модель.

### Связь
- Партиальные индексы и WHERE-clauses (Часть I, B-tree).
- MVCC и `FOR UPDATE` (Часть I).
- LSM (для понимания, почему CH иначе работает).
- Распределённая часть (Часть II) — для tq_node coordination.

### Артефакты
- `04-learning-tracks/04-postgresql-queues-listen-notify.md` — секция «как FOR UPDATE SKIP LOCKED работает на уровне MVCC».
- Замер autovacuum behavior на dump-е tq_task.

---

## *Kotlin Coroutines: Deep Dive*, Marcin Moskała

### Почему мне
Target и TQ — Kotlin, и оба используют Coroutines. Без этого я не понимаю код шедулера.

### Связь
- `TQConfig.schedulingScope = CoroutineScope(Dispatchers.IO + SupervisorJob())` — что это значит, почему IO и почему SupervisorJob.
- Структура `TQScheduler.kt` — main loop как `coroutineScope { while(...) launch { ... } }`.
- Тест coroutines — мокать ли время.

### Артефакты
- `04-learning-tracks/03-tq-kotlin-coroutines.md` — schemas + примеры из реального TQ.
- Туториал «как самому написать mini-TQ из 100 строк Kotlin».

---

## *Java Concurrency in Practice*, Brian Goetz

### Почему мне
Понимать JVM concurrency для Spring (Target) — обязательное условие staff-уровня.

### Связь
- `synchronized`, volatile, CAS — фундамент, на котором стоит Coroutines.
- Thread pools, Executors — что делает Spring под капотом.
- Memory model — почему «иногда» бывают race conditions.

### Артефакты
- В `04-learning-tracks/11-jvm-go-performance.md` — section по JVM-thread-design в Spring.
- Статья-конспект на 10 страниц, как «дискомфортная» область.

---

## *Streaming Systems*, Akidau, Chernyak, Lax

### Почему мне
Если когда-то перейдём на CDC / streaming refresh DYNAMIC аудиторий — без этой книги не разберусь.

### Связь
- Beam model: event time vs processing time.
- Watermark — это `data_as_of` в нашей freshness-модели.
- Triggers — какой analog для refresh DYNAMIC аудиторий.

### Артефакты
- `02-architecture/07-freshness-snapshot-model.md` — section «event time vs processing time в нашей системе».

---

## *Introduction to Algorithms* (CLRS), графовая часть

### Почему мне
TQ-DAG — это в чистом виде топологическая сортировка с приоритетами. Хочу понимать алгоритмически, почему `NOT EXISTS` в `getAvailableTasks` корректен.

### Связь
- Topological sort, BFS/DFS — для понимания резолюции зависимостей.
- Critical path method — для оценки минимального времени DAG.
- Single-source shortest path — для приоритизации.

### Артефакты
- `04-learning-tracks/06-dag-algorithms-and-critical-path.md` — formal definition и применение к TQ.
- Маленький Go-скрипт: построить топ-20 «самых критичных путей» из real `tq_task` dump.

---

## *Systems Performance*, Brendan Gregg

### Почему мне
Если предложу архитектурное улучшение, должен уметь профилировать «до» и «после».

### Связь
- USE method (Utilization/Saturation/Errors) — для PG, S3, Kafka, JVM.
- perf, eBPF, off-CPU profiling — практические инструменты.
- I/O subsystem — критично для S3 throughput оценок.

### Артефакты
- 1 профилировка реальной долгой TQ-задачи.
- Предложение: какие 3 метрики оffer-ит USE-метод нам в дашборде.

---

## *Software Architecture: The Hard Parts*, Ford et al.

### Почему мне
Книга про trade-off-ы (синхронность, координация, distributed transactions). Это и есть язык RFC.

### Связь
- Распределённые транзакции — почему мы их избегаем (saga и event-driven).
- Modularity vs autonomy — про микросервисную границу A-P↔Target.
- Architecture decision records — формат для нашего RFC.

### Артефакты
- ADR-формат для decisions в RFC (см. `06-practice/03-rfc-template.md`).

---

## *Enterprise Integration Patterns*, Hohpe & Woolf

### Почему мне
У нас зоопарк интеграций: REST, Kafka, S3, gRPC, Avro, JSON. Книга даёт общий язык.

### Связь
- Message Channel, Message Translator, Content-Based Router — все это про наш Planner.
- Idempotent Receiver — что нужно для T-Segment.
- Message Bus vs ETL — наш Kafka path vs A-P repackaging.

### Артефакты
- В final RFC — секция «integration patterns map» с EIP-нотацией.

---

## *The Staff Engineer's Path*, Tanya Reilly

### Почему мне
Цель — расти как platform engineer и предлагать улучшения, которые внедряются. Это книга ровно об этом.

### Связь
- «Big picture thinking» — для всего этого learning-os.
- Politics, sponsorship — как RFC реально пройдёт.
- Glue work — как обеспечить, чтобы предложения не остались на бумаге.

### Артефакты
- «Sphere of influence map» — куда мне нужно проникнуть, чтобы RFC взлетел.
- Список 5 stakeholder-ов, без которых RFC не пойдёт. План разговора с каждым.

---

## Бонусы (опционально, в свободные недели)

```text
- Designing Distributed Systems, Brendan Burns       — паттерны k8s, для общего бэкграунда.
- Site Reliability Engineering (Google book)         — про SLO/SLA discipline.
- Database Reliability Engineering, Campbell & Majors — про operational PG.
- Cloud Native Patterns, Davis                       — для будущих миграций.
- High Performance Browser Networking, Grigorik      — для понимания HTTP/2/3 в наших REST-вызовах.
```

Не цель — прочитать всё. Цель — освоить так, чтобы при code review / RFC мне не приходилось «гуглить базу».
