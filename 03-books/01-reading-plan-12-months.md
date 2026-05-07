# 01. 12-месячный план чтения

> 1 engineering + 1 business/product в месяц. Книги — не самоцель. Каждая привязана к артефакту по A-P/Target/TQ.

---

## Принципы

```text
- Глубина > широта: лучше прочитать 70% и сделать что-то руками, чем «пройти все страницы».
- Каждая книга → 1 артефакт в `01-context/` или `02-architecture/` или эксперимент в `06-practice/`.
- Если книга «не пошла» через 1 неделю — поменять её на эту же тему другую (не упорствовать).
- Параллельно engineering и business: чтобы не выгорать.
- Выходные = 1 час книги, будни = 30 минут книги вечером.
```

---

## Распределение по месяцам

> Привязка к спринтам — ориентировочная. Сдвигай по своему темпу.

### Месяц 1 — Audience Platform fundamentals (Sprints 1-2)
- **Engineering:** *Designing Data-Intensive Applications* (DDIA), Kleppmann — главы 1, 3 (Data Models, Storage), 11 (Stream Processing).  
  → артефакт: переписать `01-context/01-projects-overview.md` своими словами, наложив терминологию DDIA.
- **Business:** *The Mom Test*, Fitzpatrick.  
  → артефакт: 5 «правильных» вопросов для команды + продакта (положить в `01-context/07-open-questions-for-team.md`).

### Месяц 2 — Storage & SQL performance (Sprints 3-4)
- **Engineering:** *SQL Performance Explained*, Markus Winand. Полностью.  
  → артефакт: `06-practice/01-experiment-backlog.md` №10 — query plan + EXPLAIN ANALYZE для 1 реального CH-запроса.
- **Business:** *Continuous Discovery Habits*, Torres.  
  → артефакт: схема discovery-интервью с PM «как меряется ценность аудиторий».

### Месяц 3 — Database internals (Sprint 5)
- **Engineering:** *Database Internals*, Petrov — главы по B-tree, LSM, MVCC.  
  → артефакт: записать в `04-learning-tracks/04-postgresql-queues-listen-notify.md`, как в TQ FOR UPDATE SKIP LOCKED работает на уровне MVCC.
- **Business:** *Inspired*, Cagan. Чтение медленное.  
  → артефакт: сделать «product brief» одной аудиторной фичи.

### Месяц 4 — Concurrency (Sprint 5)
- **Engineering:** *Kotlin Coroutines: Deep Dive*, Moskała. Главы 1-7 + Flow.  
  → артефакт: разобрать `TQScheduler.kt` — где Coroutines, как Job/SupervisorJob, какие риски.
- **Business:** *High Output Management*, Grove. Главы 1-5 (managerial leverage, meetings).  
  → артефакт: формулировка собственного «1:1 шаблона».

### Месяц 5 — Distributed coordination (Sprint 6)
- **Engineering:** DDIA — главы 5 (Replication), 7 (Transactions), 8 (Trouble), 9 (Consistency & Consensus).  
  → артефакт: дополнить `01-context/04-tq-domain-map.md` failure mode-ами в духе Chaos Engineering.
- **Business:** *Obviously Awesome*, Dunford.  
  → артефакт: позиционирование A-P внутри компании (для презентации RFC).

### Месяц 6 — Streaming & data flow (Sprint 6)
- **Engineering:** *Streaming Systems*, Akidau et al. — главы 1, 3, 4 (Beam Model).  
  → артефакт: пометить, какие наши процессы — batch, какие — micro-batch, какие — потенциально stream.
- **Business:** *Crossing the Chasm*, Moore.  
  → артефакт: классификация потребителей аудиторий (early adopters vs mainstream).

### Месяц 7 — ClickHouse / OLAP (Sprint 7)
- **Engineering:** *ClickHouse: The Definitive Guide* (Altinity / docs) + DDIA глава 3 (column-oriented).  
  → артефакт: `06-practice/01-experiment-backlog.md` №10 — baseline + materialized view предложение.
- **Business:** *Good Strategy / Bad Strategy*, Rumelt.  
  → артефакт: «strategy kernel» для нашего RFC: diagnosis / guiding policy / coherent actions.

### Месяц 8 — Performance & profiling (Sprint 7)
- **Engineering:** *Systems Performance*, Brendan Gregg — главы 5 (Applications), 6 (CPU), 8 (File Systems), 10 (Network).  
  → артефакт: профилирование одной TQ-задачи (CPU/IO breakdown).
- **Business:** *Sales Acceleration Formula*, Roberge.  
  → артефакт: «как продавать архитектурное предложение внутри компании» — короткий план.

### Месяц 9 — Algorithms / DAG (Sprint 8)
- **Engineering:** *Introduction to Algorithms* (CLRS), главы по graph algorithms (BFS/DFS, topological sort, critical path).  
  → артефакт: переписать `getAvailableTasks` алгоритмически: найти critical-path в DAG, оценить стоимость.
- **Business:** *Team Topologies*, Skelton & Pais.  
  → артефакт: топология команд вокруг A-P/Target/TQ — где сейчас, где должно быть.

### Месяц 10 — Concurrency hardcore (Sprint 8)
- **Engineering:** *Java Concurrency in Practice*, Goetz — главы 5, 6, 10, 11.  
  → артефакт: разбор thread pool в Target (Spring) + проверка на безопасность для DAG-задач.
- **Business:** *Empowered*, Cagan.  
  → артефакт: «empowered team» self-assessment — где у нас autonomy, где — нет.

### Месяц 11 — Architecture as code (financial RFC)
- **Engineering:** *Software Architecture: The Hard Parts*, Ford et al. — главы 1, 4, 9.  
  → артефакт: trade-off table в финальный RFC.
- **Business:** *The Hard Thing About Hard Things*, Horowitz.  
  → артефакт: «жёсткие решения» — какие части RFC технически правильны, но политически тяжелы.

### Месяц 12 — Senior path
- **Engineering:** *The Staff Engineer's Path*, Reilly. Полностью.  
  → артефакт: «sphere of influence map» — на какие команды/системы я хочу/могу влиять.
- **Business:** *Enterprise Integration Patterns*, Hohpe & Woolf.  
  → артефакт: обобщить наши Kafka/REST/S3 паттерны в EIP-нотации, добавить в final RFC.

---

## Итого артефактов

```text
12 engineering-артефактов + 12 business-артефактов = 24.
Из них минимум 8 — кладутся в RFC и презентацию.
```

---

## Как читать

```text
1. Купи / возьми бумажную книгу или e-reader. Не читай с ноутбука — отвлекаешься.
2. Конспект — короткий: 1 страница итог на главу. Не «всё подряд».
3. После каждой главы — 5 минут «как это применимо к A-P/Target/TQ?».
4. Артефакт — в течение недели после окончания главы, иначе забудешь.
5. Раз в 2 недели — пересмотр: что из прочитанного реально используешь?
```

---

## Бэкап-список (если книга «не зашла»)

```text
Engineering (replacements):
- Database Internals — заменить на CMU 15-445 lectures.
- DDIA — заменить на CMU 15-721 advanced.
- Kotlin Coroutines — заменить на доку JetBrains + блог Moskała.
- Streaming Systems — заменить на Flink documentation + блог Akidau.

Business (replacements):
- Mom Test — заменить на блог сайта talkingtohumans.com.
- Inspired — заменить на блог Cagan (svpg.com).
- Good Strategy — заменить на эссе самого Rumelt.
```
