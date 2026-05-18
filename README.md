# Course Roadmap: A-P / Target / TQ Platform Engineering

> Большой учебный маршрут, который объединяет спринты, learning tracks, книги, research-артефакты, pet-проекты и реальные эксперименты в коде.  
> Цель — не "прочитать материалы", а вырасти до уровня, где ты можешь предложить и защитить архитектурное улучшение пайплайна аудиторий с фактами, числами и понятным rollout.

---

## 0. Что это за курс

Это не календарный план и не список файлов. Это учебный курс вокруг реальной системы:

```text
A-P / Auditory Platform
Target
TQ / ninja-t-queue
Postgres
ClickHouse
Kafka
S3
Kotlin/JVM
Go
distributed coordination
performance engineering
product / RFC / influence
```

У курса есть примерная последовательность, но нет жёстких дедлайнов. Ты можешь идти медленнее или быстрее. Важен не темп, а качество артефактов.

Ориентир по длительности:

```text
Минимальный интенсивный проход: 8-12 недель.
Нормальный глубокий проход: 4-6 месяцев.
Сильный senior-level проход с книгами, pet-проектами и RFC: 9-12 месяцев.
```

Каждый модуль устроен одинаково:

```text
1. Понять вопрос.
2. Прочитать нужную теорию.
3. Найти это в реальном коде.
4. Сделать pet-проект или маленький эксперимент.
5. Применить к A-P / Target / TQ.
6. Замерить или доказать наблюдение.
7. Выпустить артефакт.
8. Решить: это просто знание, open question или кандидат в RFC.
```

Главный принцип:

```text
Не "учусь вообще".
А "каждый блок делает меня способным принять более сильное инженерное решение".
```

---

## 1. Как работать с реальным кодом

Да, если ты что-то придумал, нормальный путь — менять код проекта, пробовать и замерять. Но не как "сразу делать фичу", а как инженерный эксперимент.

Правильный цикл:

```text
Observation
  В коде, логах, метриках или разговоре увидел проблему/возможность.

Question
  Сформулировал вопрос: "а правда ли X занимает большую часть времени?"

Hypothesis
  Если сделать Y, то метрика Z улучшится, потому что ...

Experiment branch / pet-project
  Проверяешь либо в отдельном pet-проекте, либо в ветке реального проекта.

Measurement
  Снимаешь baseline до изменения и numbers после.

Correctness
  Проверяешь, что результат не стал неправильным.

Decision
  Выводишь: бросить / оставить как research / оформить RFC / сделать production task.
```

### 1.1 Когда делать pet-проект

Pet-проект нужен, когда ты изучаешь механизм без шума production-кода.

Примеры:

- mini-TQ engine на Kotlin, чтобы понять DAG и Coroutines;
- PG queue demo с `FOR UPDATE SKIP LOCKED`;
- LISTEN/NOTIFY demo;
- RoaringBitmap benchmark;
- маленький Go benchmark для hashing/caching;
- WebClient pool exhaustion test;
- Kafka producer/consumer latency playground.

Pet-проект отвечает на вопрос:

```text
"Как работает механизм в чистом виде?"
```

### 1.2 Когда менять код реального проекта

Реальный проект трогаешь, когда нужно проверить поведение в настоящем контексте:

- где реально уходит время;
- какой query plan на настоящей схеме;
- где стоит лог/метрика;
- как ведёт себя current code path;
- можно ли встроить fast path;
- что ломается на integration boundary.

Правила:

```text
1. Делать в отдельной ветке или локально.
2. Сначала baseline, потом изменение.
3. Менять одну вещь за раз.
4. Сохранять команды, конфиги, входные данные.
5. Не смешивать research patch и production-ready implementation.
6. Если патч удачный — отдельно решить, нужен ли RFC/task/MR.
```

### 1.3 Типы изменений в проекте

Безопасные research changes:

- временные debug-логи;
- локальные metrics/timing hooks;
- benchmark-only code;
- SQL scripts;
- feature flag вокруг экспериментального пути;
- маленький alternate implementation за интерфейсом;
- local-only config.

Опасные changes, которые нельзя "просто попробовать" без отдельного решения:

- смена контракта REST/Kafka/Avro;
- миграции production-таблиц;
- изменение retry/idempotency semantics;
- изменение security/JWT;
- изменение S3 layout;
- изменение scheduler defaults;
- удаление старого пути без rollback.

---

## 2. Как использовать ИИ в этом курсе

ИИ в этом курсе — это не преподаватель, которому нужно верить, и не исполнитель, который "сделает за тебя". Это рабочий инструмент для ускорения:

```text
понимания -> поиска гипотез -> проектирования экспериментов -> проверки рассуждений -> оформления артефактов
```

Главная установка:

```text
ИИ может ускорять обучение, но не должен заменять контакт с кодом, числами и реальными фактами.
```

Если ИИ сказал "скорее всего", "обычно", "в таких системах" — это гипотеза. Она не становится фактом, пока ты не нашёл подтверждение в коде, docs, логах, метриках, тестах или разговоре с командой.

### 2.1 Роли ИИ

Используй ИИ в разных ролях и явно называй роль в запросе.

#### Tutor

Когда нужно понять тему:

```text
Объясни FOR UPDATE SKIP LOCKED через MVCC и locks.
Сначала дай модель, потом пример на 2 транзакциях, потом типичные ошибки.
В конце дай 5 self-check questions.
```

Хорошо для:

- новой темы;
- повторения;
- объяснения "на пальцах";
- подготовки к чтению документации;
- генерации self-check questions.

Ограничение:

```text
Tutor не знает, как именно это сделано в нашем коде, пока ты не дал ему конкретные файлы.
```

#### Code reading partner

Когда читаешь конкретный файл:

```text
Я читаю TQScheduler.kt. Помоги построить карту файла:
- какие основные ответственности;
- где state transitions;
- где coroutine boundaries;
- где transaction boundaries;
- какие failure modes искать.
Не делай выводы без ссылок на фрагменты кода.
```

Хорошо для:

- составления карты файла;
- поиска entry points;
- объяснения сложного control flow;
- выделения open questions;
- подготовки аннотации.

Обязательное правило:

```text
Ответ ИИ должен быть проверен глазами в коде.
Если он ссылается на несуществующий метод/класс — это сигнал снизить доверие.
```

#### Research designer

Когда появилась идея:

```text
Помоги превратить идею "one-SQL fast path" в проверяемую гипотезу.
Нужны: observation, hypothesis, minimal experiment, metrics, correctness check,
caveats, decision rule.
```

Хорошо для:

- формулировки гипотезы;
- выбора минимального эксперимента;
- определения метрик;
- поиска correctness check;
- выявления рисков.

Нельзя:

```text
Нельзя позволять ИИ превращать непроверенную идею сразу в production design.
Сначала experiment.
```

#### Experiment reviewer

Перед запуском эксперимента:

```text
Проверь дизайн эксперимента.
Где bias? Что я не контролирую? Какие метрики не хватает?
Как может сломаться correctness check?
```

Хорошо для:

- поиска confounders;
- проверки baseline;
- проверки repeatability;
- проверки caveats;
- борьбы с confirmation bias.

#### Debugging assistant

Когда что-то не работает:

```text
Вот симптом, команда, output и что я уже проверил.
Помоги построить debugging tree: самые вероятные причины, как отличить одну от другой,
какие команды выполнить следующими.
```

Хорошо для:

- структурирования отладки;
- генерации проверок;
- чтения stack traces;
- поиска missing assumptions.

Нельзя:

```text
Не проси "почини" без симптомов, команды и expected behavior.
ИИ должен работать с evidence, а не угадывать.
```

#### Code generator

Когда пишешь pet-проект, benchmark или локальный research patch:

```text
Сгенерируй минимальный Kotlin пример SKIP LOCKED worker-а.
Требования:
- только demo code;
- comments where behavior matters;
- no production abstractions;
- include how to run;
- include expected output.
```

Хорошо для:

- boilerplate;
- pet-project skeleton;
- benchmark harness;
- SQL scripts;
- small utilities;
- markdown templates.

Ограничение:

```text
Сгенерированный код не считается правильным, пока ты его не запустил,
не понял и не проверил output.
```

#### Reviewer / critic

Когда артефакт уже написан:

```text
Проведи review этой заметки как senior engineer.
Ищи:
- неподтверждённые утверждения;
- missing metrics;
- слабый correctness check;
- неявные риски;
- места, где вывод не следует из данных.
```

Хорошо для:

- RFC review;
- experiment report review;
- architecture trade-off review;
- подготовки к обсуждению с командой.

#### Editor

Когда нужно привести мысли в понятный документ:

```text
Перепиши этот research note в структуру:
Problem / Evidence / Alternatives / Recommendation / Risks / Next steps.
Не добавляй новых фактов. Пометь слабые места как TODO/evidence missing.
```

Хорошо для:

- упаковки заметок;
- сокращения воды;
- превращения raw notes в RFC section;
- подготовки презентации.

Нельзя:

```text
Нельзя разрешать ИИ добавлять "убедительные" факты, которых нет в твоих данных.
```

### 2.2 Когда ИИ нужно использовать

ИИ желательно использовать всегда, когда он снижает стоимость мышления, но не снижает качество проверки.

#### Перед чтением новой темы

Цель: получить mental model.

```text
Мне нужно изучить ClickHouse MergeTree для audience queries.
Дай карту темы:
- какие понятия обязательны;
- какие вопросы задать к нашим таблицам;
- какие команды/EXPLAIN использовать;
- какие ошибки новичка.
```

#### Перед чтением большого файла

Цель: не утонуть.

```text
Я сейчас буду читать файл <path>. По названию и контексту предположи,
какие ответственности в нём искать, какие side effects, какие tests найти рядом.
Потом я дам тебе фрагменты кода для проверки.
```

#### После чтения файла

Цель: проверить понимание.

```text
Я понял файл так: ...
Проверь рассуждение. Где я мог перепутать control flow?
Какие вопросы остались без доказательств?
```

#### Перед экспериментом

Цель: не сделать бессмысленный benchmark.

```text
Вот мой experiment design. Проверь:
- достаточно ли baseline;
- какие переменные нужно зафиксировать;
- как проверить correctness;
- какие caveats записать;
- какой decision rule поставить до запуска.
```

#### После эксперимента

Цель: не натянуть вывод на желаемое решение.

```text
Вот raw numbers. Помоги интерпретировать.
Не делай recommendation, пока не перечислишь caveats.
Отдельно проверь, следует ли вывод из данных.
```

#### Перед разговором с командой

Цель: задать fact-based вопросы.

```text
Мне нужно поговорить с PM о задержках аудиторий.
Перепиши мои вопросы в стиле Mom Test:
- без leading questions;
- про факты из прошлого;
- чтобы узнать impact и workarounds.
```

#### Перед RFC review

Цель: подготовиться к возражениям.

```text
Вот draft RFC. Сыграй skeptical reviewer из Target/A-P/SRE.
Дай top-10 возражений, какие evidence нужны для ответа,
и какие части RFC слабые.
```

### 2.3 Когда ИИ можно использовать

Можно, если результат не является final authority.

#### Можно для генерации черновиков

Подходит:

- первый draft diagram;
- первый RFC outline;
- список альтернатив;
- markdown templates;
- glossary;
- learning plan for topic.

Условие:

```text
Ты редактируешь и проверяешь черновик.
```

#### Можно для boilerplate code

Подходит:

- pet-project skeleton;
- benchmark harness;
- SQL generator;
- parsing script;
- small local utilities.

Условие:

```text
Ты запускаешь код, читаешь его и понимаешь каждую существенную часть.
```

#### Можно для объяснения ошибок

Подходит:

- stack trace;
- query plan;
- compiler error;
- failing test;
- profiler output.

Условие:

```text
Ты даёшь полный контекст: command, output, expected behavior, what changed.
```

#### Можно для сравнения подходов

Подходит:

- DAG vs SQL;
- PG queue vs Kafka;
- CopyObject vs download/upload;
- cache vs incremental;
- bitmap service vs CH bitmap.

Условие:

```text
ИИ сравнивает conceptual trade-offs.
Фактическое решение принимается только после local/project-specific evidence.
```

### 2.4 Когда ИИ нельзя использовать как источник истины

ИИ нельзя считать источником истины для:

- текущего поведения нашего кода без предоставленного кода;
- production incidents;
- точных SLA/SLO;
- схем БД, которых он не видел;
- Kafka/Avro contracts, которых он не видел;
- security decisions;
- performance claims без benchmark;
- "лучших практик" без привязки к контексту;
- политических/организационных выводов без разговоров с людьми.

Запрещённый паттерн:

```text
ИИ сказал, что ClickHouse будет быстрее, значит делаем fast path.
```

Правильный паттерн:

```text
ИИ помог сформулировать гипотезу.
Я выбрал use-case.
Я написал query.
Я сравнил с DAG.
Я проверил correctness.
Я записал caveats.
Теперь можно обсуждать fast path.
```

### 2.5 Когда ИИ вреден

ИИ вреден, если он:

- создаёт иллюзию понимания;
- заменяет чтение кода;
- подталкивает к premature architecture;
- пишет слишком убедительный RFC без evidence;
- генерирует сложный код, который ты не можешь объяснить;
- маскирует unknowns красивыми словами;
- увеличивает scope;
- превращает learning в consumption.

Красные флаги:

```text
[ ] Я не могу объяснить сгенерированный код.
[ ] Я не запускал пример.
[ ] Я не нашёл подтверждение в проекте.
[ ] Я скопировал explanation в артефакт без проверки.
[ ] В тексте появились факты, которых нет в моих notes.
[ ] После разговора с ИИ мне кажется, что я всё понял, но нет diagram/SQL/benchmark.
```

Если появился красный флаг — остановиться и вернуться к коду/эксперименту.

### 2.6 Правило доверия

Используй такую шкалу:

| Тип ответа ИИ | Доверие | Что нужно сделать |
|---|---:|---|
| Объяснение общей концепции | Medium | сверить с docs/book |
| Объяснение конкретного кода, который ты дал | Medium | проверить руками в IDE/tests |
| Утверждение о нашем проекте без кода | Low | найти code evidence |
| Performance claim | Very low | benchmark |
| Security/contract claim | Very low | проверить docs/code/team |
| RFC wording | Medium | проверить, что нет новых фактов |
| Generated pet-code | Low | запустить, прочитать, протестировать |
| Generated production code | Very low | review, tests, benchmarks, rollback |

Базовое правило:

```text
ИИ производит hypotheses and drafts.
Факты производят код, тесты, метрики, логи, документация и люди.
```

### 2.7 Как задавать хорошие запросы

Плохой запрос:

```text
Объясни TQ.
```

Хороший:

```text
Я изучаю TQ scheduler в контексте audience loading.
Моя цель — понять, где DAG добавляет latency.
Вот что я уже знаю: ...
Вот файл/фрагмент: ...
Помоги:
1. построить control-flow map;
2. выделить state transitions;
3. найти места для timing instrumentation;
4. сформулировать 5 open questions.
Не делай выводов без ссылки на код.
```

Структура хорошего запроса:

```text
Context:
  Что изучаю и зачем.

Known facts:
  Что уже подтверждено.

Input:
  Код, output, SQL, numbers, draft.

Task:
  Что именно нужно сделать.

Constraints:
  Не добавлять факты, не писать production code, давать caveats.

Output format:
  Table / checklist / experiment design / review findings.
```

### 2.8 Сценарии по модулям курса

#### Module 1. Domain

Использовать ИИ для:

- объяснения domain model;
- составления lifecycle diagram;
- генерации self-check questions;
- проверки твоего пересказа.

Нельзя:

- принимать на веру lifecycle, если он не найден в коде;
- придумывать статусы/поля, которых нет в модели.

#### Module 2. Integration

Использовать ИИ для:

- contract table;
- edge-case list;
- Avro evolution questions;
- idempotency review.

Нельзя:

- доверять ИИ про реальные Kafka topics без схем/кода;
- менять контракт по AI-рекомендации без team review.

#### Module 3. Baseline

Использовать ИИ для:

- дизайна benchmark;
- поиска missing metrics;
- интерпретации raw numbers;
- caveats.

Нельзя:

- принимать performance claims без замера;
- сравнивать local synthetic и prod как одно и то же.

#### Module 4. TQ/DAG/Coroutines

Использовать ИИ для:

- объяснения coroutines;
- аннотации scheduler-а;
- critical path model;
- failure mode brainstorming.

Нельзя:

- менять scheduler defaults на основе общего совета;
- доверять AI-generated concurrent code без тестов.

#### Module 5. Postgres

Использовать ИИ для:

- разбора EXPLAIN;
- планирования SKIP LOCKED experiment;
- объяснения MVCC;
- проверки SQL scripts.

Нельзя:

- применять AI-generated index в реальной БД без EXPLAIN и review;
- делать выводы по planner без реальной статистики.

#### Module 6. ClickHouse/S3

Использовать ИИ для:

- чтения EXPLAIN PIPELINE;
- составления CH table map;
- S3 CopyObject experiment design;
- CSV vs Parquet comparison plan.

Нельзя:

- считать, что generic CH best practice применима к нашим таблицам;
- менять S3 layout без security/ownership review.

#### Module 7. Alternatives

Использовать ИИ для:

- decision table;
- альтернатив;
- risk review;
- формулировки candidate RFC hypothesis.

Нельзя:

- позволять ИИ выбрать архитектуру без evidence;
- увеличивать scope до "нового сервиса", пока не доказан минимальный case.

#### Module 8. Production Engineering

Использовать ИИ для:

- SLI/SLO draft;
- failure mode list;
- load test plan;
- rollback checklist;
- profiling interpretation.

Нельзя:

- принимать security/reliability guarantees без проверки;
- игнорировать p95/p99;
- делать production readiness из одного локального прогона.

#### Module 9. RFC

Использовать ИИ для:

- структуры RFC;
- review objections;
- улучшения wording;
- stakeholder-specific explanations.

Нельзя:

- позволять ИИ добавить evidence;
- писать "убедительно", если facts weak;
- скрывать risks.

### 2.9 AI-assisted learning loop

Для каждой темы используй такой цикл:

```text
1. Ask AI for map
   Получить карту темы и ключевые вопросы.

2. Read primary source
   Код, docs, книга, schema, EXPLAIN.

3. Ask AI to quiz you
   Проверить понимание.

4. Build something
   Pet-project, SQL, benchmark, diagram.

5. Ask AI to review artifact
   Найти gaps, caveats, missing evidence.

6. Verify manually
   Запустить, измерить, сверить с кодом.

7. Write final note yourself
   ИИ может редактировать, но вывод формулируешь ты.
```

### 2.10 AI usage log

Для важных решений фиксируй, как использовался ИИ.

В experiment/RFC notes добавляй:

```markdown
## AI assistance

- Used for:
- Inputs provided:
- Outputs accepted:
- Outputs rejected:
- Manual verification:
- Remaining uncertainty:
```

Зачем:

- дисциплинирует;
- показывает, где facts, а где AI-generated hypotheses;
- помогает не забыть проверить слабые места.

### 2.11 Готовые промпты

#### Topic map

```text
Я изучаю <topic> для A-P / Target / TQ.
Цель: <why>.
Дай карту темы:
- key concepts;
- how it appears in backend/platform systems;
- what to look for in our code;
- hands-on exercise;
- common mistakes;
- self-check questions.
```

#### Code reading

```text
Вот файл/фрагмент кода: <paste>.
Контекст: <flow/module>.
Помоги:
- выделить responsibilities;
- построить control flow;
- найти state changes;
- найти external boundaries;
- найти failure modes;
- предложить instrumentation points.
Не делай выводов без evidence из кода.
```

#### Experiment design

```text
Идея: <idea>.
Observation: <what I saw>.
Помоги оформить experiment:
- hypothesis;
- minimal setup;
- baseline;
- change;
- metrics;
- correctness check;
- caveats;
- decision rule.
```

#### Benchmark review

```text
Вот benchmark design / raw numbers: <data>.
Проверь:
- хватает ли прогонов;
- есть ли warmup/cache bias;
- какие переменные не контролируются;
- можно ли делать вывод;
- какой следующий эксперимент.
```

#### RFC critic

```text
Вот RFC draft: <text>.
Проведи senior review:
- unsupported claims;
- missing metrics;
- hidden risks;
- weak rollout/rollback;
- likely objections by A-P/Target/SRE/PM;
- what evidence would make it stronger.
```

### 2.12 Final rule

ИИ полезен, если после него ты ближе к одному из артефактов:

```text
diagram
SQL
EXPLAIN
benchmark
profile
patch
experiment note
decision table
RFC section
```

ИИ вреден, если после него у тебя только ощущение, что стало понятнее.

---

## 3. Ритм курса

Жёстких сроков нет, но нужен ритм. Иначе курс превратится в набор красивых файлов.

### 3.1 Одна учебная сессия

Ориентир: 60-90 минут.

```text
10 мин  — выбрать один вопрос.
25 мин  — читать код/доку/книгу.
30 мин  — сделать руками: SQL, curl, profiling, pet-код, diagram.
15 мин  — записать артефакт и следующий вопрос.
```

Сессия засчитывается, если есть output:

- заметка;
- diagram;
- SQL;
- EXPLAIN;
- benchmark result;
- code annotation;
- open question;
- маленький patch;
- pet-project commit;
- RFC paragraph.

### 3.2 Один учебный блок

Ориентир: от нескольких дней до 2 недель.

```text
1. Theory
   Книга / docs / learning track.

2. Code archaeology
   Где это живёт в A-P / Target / TQ.

3. Hands-on
   Pet-project или локальное воспроизведение.

4. Measurement
   Числа, планы запросов, профили, timings.

5. Artifact
   Документ, который можно показать другому инженеру.
```

### 3.3 Один большой этап

Ориентир: 1-2 месяца.

В конце этапа у тебя должен быть не "я прошёл темы", а пакет артефактов:

```text
domain map
contract map
baseline numbers
experiment report
decision table
pet-project
open questions
RFC draft section
```

---

## 4. Карта курса

Курс состоит из 9 больших модулей.

```text
Module 0. Setup: личная система артефактов и research workflow
Module 1. Domain: что такое аудитория и как она живёт
Module 2. Integration: контракт A-P / Target / Kafka / S3
Module 3. Baseline: compute / export / delivery numbers
Module 4. Orchestration: TQ, DAG, Kotlin, Coroutines
Module 5. Storage & queues: Postgres internals, SKIP LOCKED, LISTEN/NOTIFY
Module 6. Data execution: ClickHouse, S3 locality, predicate pushdown
Module 7. Alternatives: one-SQL, cache, incremental, bitmap
Module 8. Production engineering: performance, reliability, observability, network
Module 9. RFC & influence: как превратить findings в принятую инициативу
```

Каждый следующий модуль опирается на предыдущий. Например, нельзя честно сравнивать bitmap/SQL/cache, пока ты не знаешь домен, контракт и baseline.

---

## Module 0. Setup: research workflow

### Зачем

Перед техническими темами нужно настроить способ работы. Иначе ты будешь читать, забывать и терять выводы.

### Что сделать

Создай рабочую структуру:

```text
practice/
  reading-log.md
  discovery-log.md
  experiments/
  pet-projects/
  sprint-artifacts/
  rfc/
```

Если не хочешь плодить папки сразу, можно начать с одного `practice/README.md`, но важно фиксировать output.

### Базовые шаблоны

Experiment note:

```markdown
# Experiment: <name>

## Observation

## Question

## Hypothesis

## Setup

## Method

## Baseline

## Change

## Results

## Correctness

## Caveats

## Decision
```

Reading log:

```markdown
# Reading Log

## <date> — <book/chapter/article>

- Идея:
- Где применимо в A-P / Target / TQ:
- Что проверю руками:
- Артефакт:
```

Discovery log:

```markdown
# Discovery Log

## <date> — <person/team>

- Гипотеза перед разговором:
- Факты:
- Боль:
- Что подтвердилось:
- Что опроверглось:
- Следующий вопрос:
```

### Gate

Модуль 0 завершён, когда:

```text
[ ] Есть место для research notes.
[ ] Есть reading log.
[ ] Есть experiment template.
[ ] Есть discovery log.
[ ] Ты понимаешь, куда складывать результаты каждого блока.
```

---

## Module 1. Domain: аудитория как продуктовая и техническая сущность

### Главный вопрос

```text
Что такое аудитория в нашей системе, какой у неё lifecycle,
какие бывают параметры, кто её создаёт, кто считает, кто потребляет?
```

### Что изучить

Из текущих материалов:

- [01-context/01-projects-overview.md](01-context/01-projects-overview.md)
- [01-context/02-ap-domain-map.md](01-context/02-ap-domain-map.md)
- [01-context/05-end-to-end-audience-flow.md](01-context/05-end-to-end-audience-flow.md)
- [04-learning-tracks/01-audience-platform-domain.md](04-learning-tracks/01-audience-platform-domain.md)
- [05-weekly-sprints/sprint-01-ap-end-to-end-map.md](05-weekly-sprints/sprint-01-ap-end-to-end-map.md)

Теория:

- DDIA, chapter 1: reliable, scalable, maintainable systems.
- DDIA, chapter 3: data models and storage.
- Product angle: The Mom Test, первые главы про вопросы о фактах.

Код:

```text
A-P:
- internal/domain/model/auditory.go
- internal/api/auditory/controller.go
- migrations/changelog/*.xml
- docs/processes/

Target:
- AuditoryLoadingTQProcessType.kt
- loading process model
```

### Что сделать руками

1. Нарисовать "Anatomy of an Auditory":

```text
Auditory
  id
  status
  type STATIC/DYNAMIC
  params
  gathering_id
  target_type
  showcase
  result reference
  downstream delivery
```

2. Проследить один lifecycle:

```text
DRAFT -> GATHERING -> GATHERED / ERROR -> ARCHIVED
```

3. Найти, где появляются:

- `auditory_id`;
- `gathering_id`;
- loading request;
- S3 result;
- Kafka result.

4. Если локальное окружение поднимается — создать одну тестовую аудиторию. Если нет — пройти flow по коду и тестам.

### Pet-project

Мини-проект необязателен, но полезен:

```text
audience-domain-playground
```

Смысл:

- описать `Audience`, `AudienceParams`, `Lifecycle`;
- реализовать state machine;
- запретить invalid transitions;
- написать тесты на transitions.

Это не production-код. Это способ руками почувствовать домен.

### Research artifact

Создай:

```text
practice/sprint-artifacts/01-audience-domain.md
```

Содержание:

- 1 diagram lifecycle;
- 1 diagram end-to-end flow;
- таблица основных сущностей;
- список "точно знаю из кода";
- список "гипотеза";
- список вопросов команде.

### Gate

Модуль завершён, когда ты можешь за 5 минут объяснить:

```text
[ ] Что такое Auditory.
[ ] Чем STATIC отличается от DYNAMIC.
[ ] Зачем gathering_id отдельно от auditory_id.
[ ] Кто инициирует сборку.
[ ] Как A-P узнаёт, что результат готов.
[ ] Где результат хранится.
[ ] Какие open questions остались.
```

---

## Module 2. Integration: контракт A-P / Target / Kafka / S3

### Главный вопрос

```text
Что A-P обещает Target, что Target обещает A-P,
и какие guarantees реально есть на границах REST/Kafka/Avro/S3?
```

### Что изучить

Из текущих материалов:

- [01-context/03-target-domain-map.md](01-context/03-target-domain-map.md)
- [02-architecture/02-target-contract.md](02-architecture/02-target-contract.md)
- [04-learning-tracks/02-target-integration.md](04-learning-tracks/02-target-integration.md)
- [05-weekly-sprints/sprint-02-target-contract.md](05-weekly-sprints/sprint-02-target-contract.md)
- [01-context/07-open-questions-for-team.md](01-context/07-open-questions-for-team.md)

Теория:

- DDIA, chapter 11: stream processing.
- Enterprise Integration Patterns: Message Channel, Message Translator, Idempotent Receiver.
- Software Architecture: The Hard Parts: distributed communication trade-offs.

Код и контракты:

```text
A-P:
- internal/client/target/apiclient.gen.go
- internal/client/target_http_client.go
- internal/delivery/kafka/target/auditory/consumer.go
- contract/avro/*.avsc
- docs/components/kafka/

Target:
- TargetAuditoryControllerV2.kt
- AuditoryPlatformApi.kt
- AuditoryLoadingTQTaskBuilder.kt
- contract/avro/auditory-export.avsc
```

### Что сделать руками

1. Собрать contract table:

| Direction | Protocol | Endpoint/topic | Producer | Consumer | Format | Retry | Idempotency | Ordering |
|---|---|---|---|---|---|---|---|---|
| A-P -> Target | REST |  |  |  | JSON |  |  |  |
| Target -> A-P | REST |  |  |  | JSON/JWT |  |  |  |
| Target -> A-P | Kafka |  |  |  | Avro |  |  |  |
| A-P/Target -> S3 | S3 |  |  |  | CSV/Parquet/ref |  |  |  |

2. Сделать sequence diagram:

```text
A-P create
  -> A-P loading request
  -> Target REST
  -> TQ process
  -> Target export
  -> Kafka result
  -> A-P consumer
  -> S3 / downstream
```

3. Проверить edge cases по коду:

- повторное Kafka-сообщение;
- out-of-order result;
- error payload;
- Target timeout;
- invalid JWT;
- несовместимое Avro-поле;
- missing S3 object.

4. Записать, где guarantees явные, а где предполагаемые.

### Pet-project

```text
contract-roundtrip-playground
```

Вариант минимальный:

- producer пишет Avro-like result;
- consumer читает;
- есть dedup по `gathering_id`;
- есть error case;
- есть schema evolution simulation.

Если хочешь ближе к реальности:

- локальный Kafka;
- Avro tools/schema registry;
- один REST stub.

### Real-code experiment

В реальном проекте можно попробовать:

- добавить временный trace-id across A-P -> Target -> Kafka -> A-P;
- добавить timing logs на границах;
- написать integration test на duplicate message;
- проверить schema compatibility локально.

Не production change, а research patch.

### Research artifact

```text
practice/sprint-artifacts/02-contract-map.md
```

Содержание:

- contract table;
- sequence diagram;
- список invariants;
- список "можно расширять";
- список "нельзя ломать";
- edge cases;
- open questions для команды.

### Gate

Модуль завершён, когда:

```text
[ ] Ты можешь объяснить весь round-trip A-P -> Target -> A-P.
[ ] Для каждого topic/endpoint знаешь producer и consumer.
[ ] Понимаешь, где idempotency есть или отсутствует.
[ ] Понимаешь Avro evolution constraints.
[ ] Видишь, какие изменения потребуют RFC/coordination.
```

---

## Module 3. Baseline: compute / export / delivery numbers

### Главный вопрос

```text
Где реально уходит время: compute, export, delivery, orchestration, S3, Kafka или downstream?
```

Без этого любые идеи про CH, bitmap, cache или TQ tuning — догадки.

### Что изучить

Из текущих материалов:

- [01-context/06-compute-export-delivery-split.md](01-context/06-compute-export-delivery-split.md)
- [02-architecture/05-execution-router-planner.md](02-architecture/05-execution-router-planner.md)
- [06-practice/02-benchmark-template.md](06-practice/02-benchmark-template.md)
- [05-weekly-sprints/sprint-03-compute-export-delivery.md](05-weekly-sprints/sprint-03-compute-export-delivery.md)

Теория:

- Systems Performance: USE method.
- Brendan Gregg materials on latency and profiling.
- DDIA: latency, throughput, reliability framing.
- Little's Law basics.

### Что измерять

Раздели пайплайн:

```text
Request
  HTTP request / user action

Queueing
  loading request waits before picked

Compute
  Target/TQ tasks calculate audience

Export
  write result to S3 / Kafka / A-P receive / repack

Delivery
  A-P sends to downstream / T-Segment / channel

Total lead time
  from user action to usable downstream result
```

Таблица baseline:

| Audience size | Request | Queue wait | Compute | Export | Delivery | Total | p50 | p95 | Notes |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---|
| 10k |  |  |  |  |  |  |  |  |  |
| 1M |  |  |  |  |  |  |  |  |  |
| 10M |  |  |  |  |  |  |  |  |  |

### Что сделать руками

1. SQL timeline по `tq_task`:

```text
process_id
task_type
status
date_started
date_finished
duration
dependency depth
```

2. A-P timing points:

- loading request created;
- Target REST called;
- Kafka result consumed;
- S3 object read/written;
- downstream produce done.

3. Снять минимум несколько прогонов одного сценария.

4. Для каждого числа написать caveat:

- local/staging/prod;
- synthetic/real data;
- warm/cold cache;
- approximate/complete.

### Pet-project

Не обязателен, но полезен:

```text
pipeline-timeline-analyzer
```

Идея:

- на вход CSV/JSON из таблиц/logs;
- на выход timeline и breakdown;
- можно строить Gantt-like representation в markdown.

### Real-code experiment

Можно временно добавить:

- timing logs;
- trace-id propagation;
- structured event per stage;
- local metrics around S3/Kafka/Target call.

Важно: это research instrumentation. Если окажется полезным — потом отдельно оформить production observability task.

### Research artifact

```text
practice/sprint-artifacts/03-baseline-numbers.md
```

Содержание:

- схема stages;
- SQL/commands;
- raw numbers;
- aggregated p50/p95;
- bottleneck candidates;
- "если ускорить X в 10 раз, total изменится на Y";
- open questions.

### Gate

Модуль завершён, когда:

```text
[ ] Есть baseline хотя бы для одного реального или близкого к реальному сценария.
[ ] Есть разделение compute/export/delivery.
[ ] Есть p50/p95 или честное объяснение, почему пока нет.
[ ] Есть главный bottleneck candidate.
[ ] Есть список данных, которых не хватает.
```

---

## Module 4. Orchestration: TQ, DAG, Kotlin, Coroutines

### Главный вопрос

```text
Как TQ планирует задачи, как работает DAG,
какой recovery model после падения ноды,
и где границы параллелизма?
```

### Что изучить

Из текущих материалов:

- [01-context/04-tq-domain-map.md](01-context/04-tq-domain-map.md)
- [02-architecture/03-tq-role-in-system.md](02-architecture/03-tq-role-in-system.md)
- [04-learning-tracks/03-tq-kotlin-coroutines.md](04-learning-tracks/03-tq-kotlin-coroutines.md)
- [04-learning-tracks/06-dag-algorithms-and-critical-path.md](04-learning-tracks/06-dag-algorithms-and-critical-path.md)
- [05-weekly-sprints/sprint-05-tq-coroutines.md](05-weekly-sprints/sprint-05-tq-coroutines.md)

Теория:

- Kotlin Coroutines: Deep Dive.
- Java Concurrency in Practice.
- CLRS graph chapters: topological sort, DFS/BFS, critical path.
- DDIA chapters on coordination/failures.

Код:

```text
TQ:
- TQ.kt
- TQTaskDsl.kt
- TQConfig.kt
- TQScheduler.kt
- TQHeartbeatService.kt
- TQJdbiTaskDao.kt
- tq_task
- tq_process
- tq_task_dependencies
- tq_node

Target:
- AuditoryLoadingTQTaskBuilder.kt
- process/task types
```

### Что сделать руками

1. Прочитать `TQScheduler.kt` построчно.

Аннотация должна отвечать:

- где loop;
- где suspend points;
- где task reservation;
- где dependency resolution;
- где heartbeat;
- где retry/failure;
- где cancellation risk;
- где transaction boundary.

2. Построить DAG model:

```text
nodes = tasks
edges = dependencies
available task = all predecessors completed
critical path = longest weighted path
```

3. Сделать recovery experiment:

- 3 процесса;
- один умирает во время `IN_PROGRESS`;
- замерить, когда задача вернулась;
- понять, какие настройки влияют.

4. Проверить `continuous=true`:

- DAG depth 5;
- compare `continuous=false` vs `true`;
- замерить latency difference.

### Pet-project

```text
mini-tq
```

Минимальная версия:

- process;
- task;
- dependency;
- in-memory scheduler;
- statuses;
- topological availability;
- worker pool;
- retry;
- timeout.

Расширенная версия:

- Postgres table;
- `FOR UPDATE SKIP LOCKED`;
- heartbeat;
- stale task release;
- coroutine-based scheduler.

Зачем:

```text
Чтобы понять TQ не как набор классов, а как модель исполнения.
```

### Real-code experiment

В реальном TQ/Target можно:

- поменять локально `interval`;
- поменять concurrency;
- включить/выключить continuous;
- добавить timings вокруг scheduler loop;
- снять DB queries;
- снять JFR/async-profiler.

Нельзя без отдельного решения:

- менять defaults для production;
- менять recovery semantics;
- менять retry/idempotency;
- менять schema.

### Research artifact

```text
practice/sprint-artifacts/04-tq-dag-coroutines.md
```

Содержание:

- annotated scheduler notes;
- diagram process/task/dependency;
- recovery experiment;
- continuous experiment;
- critical path explanation;
- list of tuning levers and risks.

### Gate

Модуль завершён, когда:

```text
[ ] Ты можешь объяснить TQScheduler main loop.
[ ] Понимаешь SupervisorJob / Dispatchers.IO / cancellation.
[ ] Можешь объяснить available task query.
[ ] Есть recovery time measurement.
[ ] Есть понимание critical path.
[ ] Есть mini-TQ или equivalent hands-on artifact.
```

---

## Module 5. Storage & Queues: Postgres internals

### Главный вопрос

```text
Как Postgres держит очередь TQ,
какие индексы и locks это обеспечивают,
и где потолок такого подхода?
```

### Что изучить

Из текущих материалов:

- [04-learning-tracks/04-postgresql-queues-listen-notify.md](04-learning-tracks/04-postgresql-queues-listen-notify.md)
- [05-weekly-sprints/sprint-06-postgres-queue.md](05-weekly-sprints/sprint-06-postgres-queue.md)
- [01-context/04-tq-domain-map.md](01-context/04-tq-domain-map.md)

Теория:

- Database Internals: B-tree, MVCC, storage.
- SQL Performance Explained.
- PostgreSQL docs: MVCC, locks, indexes, planner, VACUUM.
- Use The Index, Luke.

Код/SQL:

```text
- TQJdbiTaskDao.kt getAvailableTasks
- 002_create_tq_task_dependencies_table.xml
- 014_add_new_indices_on_database.xml
- tq_task
- tq_task_dependencies
- tq_node
```

### Что сделать руками

1. Synthetic table:

```text
1M tq_task rows
dependencies
statuses distribution
partial indexes
```

2. Query plan:

- run `EXPLAIN (ANALYZE, BUFFERS)`;
- drop partial indexes;
- compare;
- recreate one by one.

3. SKIP LOCKED experiment:

- 1 consumer;
- 4 consumers;
- 16 consumers;
- measure throughput and conflicts.

4. Bloat/autovacuum experiment:

- 100k updates;
- check dead tuples;
- VACUUM ANALYZE;
- compare.

5. LISTEN/NOTIFY experiment:

- producer inserts task and sends notify;
- consumer listens;
- fallback poll;
- compare latency vs poll.

### Pet-project

```text
pg-queue-lab
```

Функции:

- enqueue task;
- claim task with SKIP LOCKED;
- complete/fail;
- multiple workers;
- LISTEN/NOTIFY mode;
- benchmark script.

### Real-code experiment

Можно:

- снять query plans на реальной схеме;
- посмотреть index usage;
- проверить dead tuples;
- проверить `pg_stat_statements`;
- построить рекомендации по autovacuum.

### Research artifact

```text
practice/sprint-artifacts/05-postgres-queue.md
```

Содержание:

- query plans;
- throughput table;
- bloat observations;
- index explanation;
- LISTEN/NOTIFY comparison;
- recommendation: PG queue vs Kafka/NATS/other.

### Gate

Модуль завершён, когда:

```text
[ ] Ты читаешь EXPLAIN без угадывания.
[ ] Понимаешь MVCC и SKIP LOCKED.
[ ] Знаешь, зачем partial indexes.
[ ] Есть throughput numbers.
[ ] Есть bloat/autovacuum observation.
[ ] Можешь объяснить, когда PG queue достаточно, а когда нет.
```

---

## Module 6. Data Execution: ClickHouse, S3, predicate pushdown

### Главный вопрос

```text
Какие audience use-case-ы можно считать дешевле через ClickHouse/S3/pushdown,
и где текущий DAG делает лишнюю работу?
```

### Что изучить

Из текущих материалов:

- [04-learning-tracks/07-clickhouse-performance.md](04-learning-tracks/07-clickhouse-performance.md)
- [04-learning-tracks/08-data-locality-s3-pushdown.md](04-learning-tracks/08-data-locality-s3-pushdown.md)
- [05-weekly-sprints/sprint-07-clickhouse-baseline.md](05-weekly-sprints/sprint-07-clickhouse-baseline.md)
- [02-architecture/04-graph-vs-sql-vs-bitmap-vs-cache.md](02-architecture/04-graph-vs-sql-vs-bitmap-vs-cache.md)

Теория:

- ClickHouse docs: MergeTree, query execution, EXPLAIN, materialized views.
- Altinity materials.
- DDIA chapter 3: column-oriented storage.
- Parquet docs.
- S3 docs: CopyObject, multipart upload, ListObjectsV2.

### Что сделать руками

1. CH table map:

| Table | Engine | Rows | Size | Partition key | Sorting key | Used by | Query type |
|---|---|---:|---:|---|---|---|---|

2. One real query:

- `EXPLAIN ESTIMATE`;
- `EXPLAIN PIPELINE`;
- `EXPLAIN PLAN`;
- `system.query_log`;
- wall-clock.

3. Partition pruning:

- query without partition-friendly predicate;
- query with predicate;
- compare read rows/bytes/time.

4. Materialized view/projection:

- identify repeated query pattern;
- prototype MV on test data;
- compare raw vs MV.

5. S3 locality:

- find double packaging path;
- measure download/upload vs CopyObject if possible;
- compare CSV vs Parquet size/read time;
- test CH `s3()` if possible.

### Pet-project

```text
ch-audience-lab
```

Вариант:

- synthetic audience source table;
- dimensions: city, age, subscription, income, MCC;
- queries for simple audience predicates;
- benchmark raw query vs MV/projection;
- optional S3 Parquet input.

### Real-code experiment

Можно в реальном проекте:

- написать one-SQL equivalent for one simple audience;
- compare count/sample with DAG output;
- add local experimental path behind flag;
- measure wall-clock;
- inspect query_log.

Важно:

```text
One-SQL fast path считается кандидатом только если:
- определён класс аудиторий;
- correctness проверен;
- fallback на старый DAG понятен;
- performance выигрыш измерен;
- operational owner понятен.
```

### Research artifact

```text
practice/sprint-artifacts/06-clickhouse-s3-execution.md
```

Содержание:

- CH table map;
- annotated query plan;
- raw vs MV/projection numbers;
- partition pruning result;
- S3 locality note;
- список candidate use-cases for fast path.

### Gate

Модуль завершён, когда:

```text
[ ] Есть карта CH-таблиц.
[ ] Есть хотя бы один explained real query.
[ ] Есть numbers для raw vs optimized path.
[ ] Есть понимание partition pruning.
[ ] Есть вывод по S3 double packaging.
[ ] Есть 1-3 candidate use-cases для one-SQL fast path.
```

---

## Module 7. Alternatives: SQL, cache, incremental, bitmap

### Главный вопрос

```text
Когда нужен DAG, когда one-SQL, когда cache, когда incremental,
когда bitmap, и почему?
```

Это центральный архитектурный модуль. Из него обычно рождается RFC.

### Что изучить

Из текущих материалов:

- [02-architecture/04-graph-vs-sql-vs-bitmap-vs-cache.md](02-architecture/04-graph-vs-sql-vs-bitmap-vs-cache.md)
- [02-architecture/05-execution-router-planner.md](02-architecture/05-execution-router-planner.md)
- [02-architecture/06-result-registry-model.md](02-architecture/06-result-registry-model.md)
- [02-architecture/07-freshness-snapshot-model.md](02-architecture/07-freshness-snapshot-model.md)
- [04-learning-tracks/09-bitmap-inverted-index-serving.md](04-learning-tracks/09-bitmap-inverted-index-serving.md)
- [04-learning-tracks/10-cache-incremental-cdc.md](04-learning-tracks/10-cache-incremental-cdc.md)
- [05-weekly-sprints/sprint-04-graph-vs-alternatives.md](05-weekly-sprints/sprint-04-graph-vs-alternatives.md)
- [05-weekly-sprints/sprint-08-bitmap-experiment.md](05-weekly-sprints/sprint-08-bitmap-experiment.md)

Теория:

- DDIA: batch/stream, replication, materialized views.
- Streaming Systems: event time, watermarks, triggers.
- RoaringBitmap docs.
- Debezium/logical decoding docs.
- Software Architecture: The Hard Parts.

### Alternative paths

#### Current DAG

Плюсы:

- general-purpose;
- handles complex workflows;
- existing operational model;
- retries/recovery already there.

Минусы:

- orchestration overhead;
- depth/interval latency;
- export/repack cost;
- может быть избыточен для simple predicates.

#### One-SQL ClickHouse

Плюсы:

- simple path;
- less orchestration;
- good for predicates already in CH;
- easy to compare with count/sample.

Минусы:

- limited class of audiences;
- query complexity;
- CH load risk;
- correctness/freshness constraints.

#### Cache by definition hash

Плюсы:

- avoids repeated compute;
- great if many duplicate params;
- can be transparent.

Минусы:

- invalidation;
- freshness;
- stampede;
- negative cache semantics.

#### Incremental / CDC

Плюсы:

- avoids full rebuild;
- good for dynamic audiences;
- closer to streaming freshness.

Минусы:

- watermark complexity;
- drift;
- reconciliation;
- exactly-once illusions.

#### Bitmap / inverted index

Плюсы:

- very fast set operations;
- good for discrete predicates;
- count/intersect/union in milliseconds.

Минусы:

- build/update pipeline;
- range predicates;
- memory/warmup;
- operational owner;
- may duplicate CH functionality.

### Что сделать руками

1. Выбрать один use-case:

```text
simple predicate audience
known source tables
correctness check possible
not too large for experiments
```

2. Прогнать paths:

| Path | Result count | p50 | p95 | Bytes read | CPU/mem | Correctness | Notes |
|---|---:|---:|---:|---:|---:|---|---|
| Current DAG |  |  |  |  |  |  |  |
| One-SQL CH |  |  |  |  |  |  |  |
| Cache estimate |  |  |  |  |  |  |  |
| Bitmap standalone |  |  |  |  |  |  |  |
| CH bitmap |  |  |  |  |  |  |  |

3. Cache experiment:

- normalize AuditoryParams;
- hash;
- count duplicate params;
- estimate ceiling cache-hit-rate.

4. Bitmap experiment:

- 10M synthetic clients;
- 100 segments;
- RoaringBitmap;
- CH bitmap;
- hybrid with range predicates.

5. Decision table:

| Criterion | DAG | One-SQL | Cache | Incremental | Bitmap |
|---|---|---|---|---|---|
| Best for |  |  |  |  |  |
| Bad for |  |  |  |  |  |
| Latency |  |  |  |  |  |
| Freshness |  |  |  |  |  |
| Correctness risk |  |  |  |  |  |
| Operational complexity |  |  |  |  |  |
| Rollback |  |  |  |  |  |
| Owner |  |  |  |  |  |
| Required data |  |  |  |  |  |

### Pet-projects

Можно сделать несколько:

```text
audience-cache-hash-lab
bitmap-vs-sql-lab
incremental-refresh-lab
planner-decision-table-lab
```

Самый ценный:

```text
planner-lab
```

Идея:

- на вход audience params;
- classifier решает: DAG / SQL / cache / bitmap;
- decision объясняется reason codes;
- есть benchmark на synthetic cases.

### Real-code experiment

В реальном проекте можно попробовать:

- experimental fast path behind feature flag;
- cache hash calculation;
- result registry prototype;
- CH one-SQL path for one safe audience class;
- extra metrics around existing DAG path.

Не нужно сразу делать production implementation. Цель сначала — evidence.

### Research artifact

```text
practice/sprint-artifacts/07-execution-alternatives.md
```

Содержание:

- chosen use-case;
- all path measurements;
- correctness method;
- decision table;
- candidate RFC hypothesis;
- rejected alternatives with reasons.

### Gate

Модуль завершён, когда:

```text
[ ] Есть сравнение минимум 2 альтернатив с current path.
[ ] Есть correctness check.
[ ] Есть numbers.
[ ] Есть decision table.
[ ] Есть формулировка candidate RFC hypothesis.
```

---

## Module 8. Production Engineering: performance, reliability, observability, network

### Главный вопрос

```text
Если решение станет production path,
как оно будет вести себя под нагрузкой, сбоями, p99, retries, network issues и rollback?
```

### Что изучить

Из текущих материалов:

- [02-architecture/08-observability-model.md](02-architecture/08-observability-model.md)
- [04-learning-tracks/05-distributed-coordination.md](04-learning-tracks/05-distributed-coordination.md)
- [04-learning-tracks/11-jvm-go-performance.md](04-learning-tracks/11-jvm-go-performance.md)
- [04-learning-tracks/13-networks.md](04-learning-tracks/13-networks.md)

Теория:

- Systems Performance.
- Java Performance.
- Go GC guide.
- async-profiler docs.
- High Performance Browser Networking.
- Kafka protocol docs.
- gRPC docs.
- SRE book: SLI/SLO basics.

### Темы

Performance:

- latency vs throughput;
- p50/p95/p99;
- Little's Law;
- CPU/memory/allocation;
- GC pressure;
- connection pool saturation;
- query planner;
- coordinated omission.

Reliability:

- failure modes;
- retry storms;
- idempotency;
- stale tasks;
- partial failure;
- rollback;
- feature flags.

Observability:

- logs;
- metrics;
- traces/correlation id;
- RED/USE/golden signals;
- cardinality;
- SLI/SLO/SLA.

Network:

- TCP/TLS/HTTP/gRPC;
- Kafka acks and rebalance;
- DNS TTL;
- connection pools;
- timeouts/deadlines;
- S3 latency.

### Что сделать руками

1. Go profiling for A-P:

- pprof CPU;
- pprof heap;
- `go test -race`;
- escape analysis;
- one benchmark with `-benchmem`.

2. JVM profiling for Target/TQ:

- JFR or async-profiler;
- top CPU methods;
- top allocations;
- lock contention if any;
- GC observation.

3. Network experiments:

- `grpcurl` to endpoint;
- Kafka producer latency with different `acks`;
- WebClient connection pool exhaustion;
- DNS TTL check in JVM;
- `curl -vv` TLS handshake.

4. Observability design:

For the candidate RFC, define:

```text
SLI:
  p95 lead time for class X
  error rate
  fallback rate
  correctness mismatch rate
  CH query failure rate
  cache hit rate

SLO:
  target values

Dashboards:
  what graphs needed

Alerts:
  what pages someone
```

### Pet-projects

```text
go-pprof-lab
jvm-jfr-lab
kafka-latency-lab
webclient-pool-lab
grpc-deadline-lab
```

### Real-code experiment

Можно:

- enable existing pprof/JFR locally;
- add metrics around candidate path;
- run load test;
- simulate downstream timeout;
- test feature flag fallback.

### Research artifact

```text
practice/sprint-artifacts/08-production-readiness.md
```

Содержание:

- performance profile;
- reliability risks;
- observability proposal;
- rollback/fallback plan;
- SLO draft;
- production readiness checklist.

### Gate

Модуль завершён, когда:

```text
[ ] Есть profile хотя бы одного Go или JVM path.
[ ] Есть p95/p99 thinking, не только average.
[ ] Есть reliability failure-mode list.
[ ] Есть SLI/SLO draft.
[ ] Есть rollout/rollback idea.
```

---

## Module 9. RFC & Influence

### Главный вопрос

```text
Как превратить findings в инициативу, которую команда поймёт,
обсудит, примет и сможет безопасно внедрить?
```

### Что изучить

Из текущих материалов:

- [07-final-rfc/01-final-rfc-outline.md](07-final-rfc/01-final-rfc-outline.md)
- [07-final-rfc/02-senior-initiative-dod.md](07-final-rfc/02-senior-initiative-dod.md)
- [06-practice/03-rfc-template.md](06-practice/03-rfc-template.md)
- [04-learning-tracks/12-business-product-sales-leadership.md](04-learning-tracks/12-business-product-sales-leadership.md)
- [03-books/03-business-product-books.md](03-books/03-business-product-books.md)

Теория:

- The Mom Test.
- Continuous Discovery Habits.
- Good Strategy / Bad Strategy.
- Obviously Awesome.
- High Output Management.
- Team Topologies.
- The Staff Engineer's Path.
- Software Architecture: The Hard Parts.

### Что сделать руками

1. Stakeholder map:

| Stakeholder | Team | What they care about | Likely objection | Evidence they need |
|---|---|---|---|---|

2. Discovery conversations:

- PM;
- A-P engineer;
- Target engineer;
- downstream consumer;
- ops/SRE/DBA if relevant.

Вопросы должны быть фактологическими:

```text
Когда последний раз задержка аудитории повлияла на кампанию?
Как это заметили?
Какой workaround был?
Какая задержка уже считается проблемой?
Какие аудитории самые важные?
Какие изменения вы боитесь больше всего?
```

3. Strategy kernel:

```text
Diagnosis:
  Что не так и почему.

Guiding policy:
  Какой принцип решения.

Coherent actions:
  Какие конкретные шаги.
```

4. RFC draft:

```text
TL;DR
Problem
Current state
Baseline numbers
Diagnosis
Alternatives
Recommendation
Architecture
Rollout
Rollback
Metrics
Risks
Open questions
Effort
```

### Research artifact

```text
practice/rfc/final-rfc-draft.md
```

и отдельно:

```text
practice/rfc/stakeholder-map.md
practice/rfc/strategy-kernel.md
practice/rfc/review-objections.md
```

### Gate

RFC готов к review, когда:

```text
[ ] Problem подтверждена фактами.
[ ] Current state описан с code references.
[ ] Есть baseline numbers.
[ ] Есть alternatives considered.
[ ] Recommendation следует из decision table.
[ ] Есть rollout.
[ ] Есть rollback.
[ ] Есть success metrics.
[ ] Есть risks.
[ ] Есть open questions.
[ ] Ты знаешь, кто будет спорить и почему.
```

---

## 5. Интегрированный порядок прохождения

Ниже не дедлайны, а рекомендуемый порядок. Один пункт может занять 2 дня, неделю или месяц.

### Stage A. Foundation

Пройти:

- Module 0. Setup.
- Module 1. Domain.
- Module 2. Integration.

Книги параллельно:

- DDIA chapters 1, 3, 11.
- The Mom Test.

Pet-projects:

- audience-domain-playground;
- contract-roundtrip-playground.

Артефакты:

- audience domain map;
- end-to-end diagram;
- contract map;
- first open questions.

Результат stage:

```text
Ты понимаешь, что делает система и где проходят границы между сервисами.
```

### Stage B. Measurement

Пройти:

- Module 3. Baseline.

Книги параллельно:

- Systems Performance, selected chapters.
- SQL Performance Explained, если baseline упирается в DB/CH.

Pet-project:

- pipeline-timeline-analyzer.

Артефакты:

- compute/export/delivery table;
- benchmark note;
- bottleneck candidates.

Результат stage:

```text
Ты перестаёшь говорить "кажется медленно" и начинаешь говорить "вот где время".
```

### Stage C. Execution Engine

Пройти:

- Module 4. TQ/DAG/Kotlin/Coroutines.
- Module 5. Postgres queue.

Книги параллельно:

- Kotlin Coroutines: Deep Dive.
- Java Concurrency in Practice.
- Database Internals.
- DDIA chapters 7-9.

Pet-projects:

- mini-tq;
- pg-queue-lab.

Артефакты:

- annotated TQScheduler;
- recovery experiment;
- critical path note;
- SKIP LOCKED benchmark;
- LISTEN/NOTIFY demo.

Результат stage:

```text
Ты понимаешь orchestration cost и границы текущего DAG/TQ подхода.
```

### Stage D. Data Paths

Пройти:

- Module 6. ClickHouse/S3.
- Module 7. Alternatives.

Книги параллельно:

- ClickHouse docs/book.
- Streaming Systems selected chapters.
- CLRS graph chapters.
- Software Architecture: The Hard Parts.

Pet-projects:

- ch-audience-lab;
- planner-lab;
- bitmap-vs-sql-lab;
- audience-cache-hash-lab.

Артефакты:

- CH table map;
- query explain report;
- one-SQL vs DAG experiment;
- cache-hit estimate;
- bitmap report;
- decision table.

Результат stage:

```text
Ты можешь честно сказать, какой execution path подходит какому классу аудиторий.
```

### Stage E. Production Readiness

Пройти:

- Module 8. Performance/Reliability/Observability/Network.

Книги параллельно:

- Systems Performance.
- Java Performance.
- High Performance Browser Networking.
- SRE book selected chapters.

Pet-projects:

- go-pprof-lab;
- jvm-jfr-lab;
- kafka-latency-lab;
- webclient-pool-lab.

Артефакты:

- profiling report;
- SLI/SLO draft;
- reliability risk list;
- rollout/rollback sketch.

Результат stage:

```text
Ты можешь оценить не только "быстрее ли", но и "можно ли это безопасно эксплуатировать".
```

### Stage F. RFC

Пройти:

- Module 9. RFC & Influence.

Книги параллельно:

- Good Strategy / Bad Strategy.
- Obviously Awesome.
- Team Topologies.
- The Staff Engineer's Path.

Артефакты:

- stakeholder map;
- strategy kernel;
- RFC draft;
- review objections;
- final presentation outline.

Результат stage:

```text
У тебя есть инженерная инициатива, которую можно обсуждать с командой.
```

---

## 6. Книжная программа как часть курса

Книги не читаются отдельно от практики. Каждая книга должна дать артефакт.

### Engineering core

#### DDIA

Когда читать:

- Stage A для языка data systems;
- Stage C для transactions/consistency;
- Stage D для streams/materialized views.

Артефакты:

- glossary для A-P/Target/TQ;
- delivery semantics в contract map;
- freshness/event-time note.

#### SQL Performance Explained

Когда читать:

- Stage B/C/D.

Артефакты:

- EXPLAIN одного PG/CH запроса;
- index reasoning note.

#### Database Internals

Когда читать:

- Stage C.

Артефакты:

- MVCC + SKIP LOCKED note;
- bloat/autovacuum experiment.

#### Kotlin Coroutines: Deep Dive

Когда читать:

- Stage C.

Артефакты:

- annotated `TQScheduler.kt`;
- mini-TQ;
- coroutine cancellation note.

#### Java Concurrency in Practice

Когда читать:

- Stage C/E.

Артефакты:

- JVM threading note for Target/TQ;
- thread pool / lock contention analysis.

#### Streaming Systems

Когда читать:

- Stage D.

Артефакты:

- freshness/watermark model;
- incremental refresh design note.

#### CLRS Graph Chapters

Когда читать:

- Stage C/D.

Артефакты:

- critical path script;
- DAG depth/width report.

#### Systems Performance

Когда читать:

- Stage B/E.

Артефакты:

- profiling report;
- USE method dashboard proposal.

#### Software Architecture: The Hard Parts

Когда читать:

- Stage D/F.

Артефакты:

- trade-off table;
- architecture decision records.

#### Enterprise Integration Patterns

Когда читать:

- Stage F or late Stage A.

Артефакты:

- integration pattern map for RFC.

#### The Staff Engineer's Path

Когда читать:

- Stage F.

Артефакты:

- sphere of influence map;
- stakeholder strategy.

### Product / business / influence core

#### The Mom Test

Артефакт:

- переписать вопросы команде в fact-based style;
- провести 1 discovery conversation.

#### Continuous Discovery Habits

Артефакт:

- opportunity-solution tree для RFC.

#### Inspired

Артефакт:

- product brief: почему скорость/надёжность аудиторий важна.

#### Obviously Awesome

Артефакт:

- positioning canvas для RFC.

#### Good Strategy / Bad Strategy

Артефакт:

- diagnosis / guiding policy / coherent actions.

#### High Output Management

Артефакт:

- indicator pairs для pipeline health.

#### Team Topologies

Артефакт:

- team ownership map: кто владеет A-P/Target/TQ/Planner.

#### Empowered

Артефакт:

- self-assessment: где нужна autonomy/support.

---

## 7. Pet-project каталог

Pet-проекты нужны не для портфолио, а для понимания механизмов.

### 7.1 audience-domain-playground

Изучает:

- domain model;
- lifecycle;
- invalid transitions.

Минимум:

- entities;
- state machine;
- tests.

### 7.2 contract-roundtrip-playground

Изучает:

- REST/Kafka boundary;
- schema evolution;
- idempotency;
- duplicate message handling.

Минимум:

- producer;
- consumer;
- dedup;
- error case.

### 7.3 pipeline-timeline-analyzer

Изучает:

- baseline decomposition;
- timeline thinking;
- bottleneck analysis.

Минимум:

- parse CSV/log;
- calculate stage durations;
- output markdown table.

### 7.4 mini-tq

Изучает:

- process/task/dependency;
- scheduler;
- coroutines;
- critical path;
- recovery.

Минимум:

- in-memory DAG scheduler.

Сильная версия:

- Postgres;
- SKIP LOCKED;
- heartbeat.

### 7.5 pg-queue-lab

Изучает:

- MVCC;
- SKIP LOCKED;
- partial indexes;
- LISTEN/NOTIFY;
- bloat.

Минимум:

- queue table;
- multiple workers;
- benchmark.

### 7.6 ch-audience-lab

Изучает:

- CH query execution;
- MergeTree;
- partition pruning;
- MV/projections.

Минимум:

- synthetic table;
- simple audience queries;
- EXPLAIN reports.

### 7.7 bitmap-vs-sql-lab

Изучает:

- RoaringBitmap;
- CH bitmap functions;
- discrete predicate acceleration.

Минимум:

- 10M synthetic ids;
- 100 segments;
- intersect/union/count benchmark.

### 7.8 audience-cache-hash-lab

Изучает:

- params normalization;
- stable hashing;
- cache hit potential;
- single-flight.

Минимум:

- canonical JSON;
- sha256/blake3;
- duplicate detection.

### 7.9 planner-lab

Изучает:

- decision engine;
- execution path selection;
- reason codes.

Минимум:

```text
input: audience params + metadata
output: DAG / SQL / Cache / Bitmap + explanation
```

### 7.10 production-labs

Маленькие labs:

- go-pprof-lab;
- jvm-jfr-lab;
- kafka-latency-lab;
- webclient-pool-lab;
- grpc-deadline-lab.

---

## 8. Как превращать идею в эксперимент

Если появилась идея, не записывай её как "сделать". Записывай как hypothesis.

### Пример 1. One-SQL fast path

Плохая формулировка:

```text
Сделать быстрый путь через ClickHouse.
```

Хорошая:

```text
Observation:
  Для простых STATIC-аудиторий orchestration/export занимает большую часть total lead time.

Hypothesis:
  Для класса аудиторий с predicates полностью в CH one-SQL path даст p95 < X sec
  и результат совпадёт с текущим DAG по count/sample.

Experiment:
  Выбрать 3 аудитории, прогнать DAG и one-SQL по 5 раз,
  сравнить latency, rows read, result count, sample hash.
```

### Пример 2. Cache by definition hash

Плохая:

```text
Добавить кэш.
```

Хорошая:

```text
Observation:
  Пользователи часто создают похожие аудитории.

Hypothesis:
  Нормализованный params_hash даст cache-hit ceiling > 30% на последних 10k аудиторий.

Experiment:
  Выгрузить params, canonicalize, hash, посчитать duplicates by class.
```

### Пример 3. S3 CopyObject

Плохая:

```text
Убрать двойную упаковку S3.
```

Хорошая:

```text
Observation:
  A-P скачивает объект из target bucket и загружает заново.

Hypothesis:
  Server-side CopyObject уменьшит export time на X% для файлов > N MB.

Experiment:
  На одинаковых объектах сравнить download/upload vs CopyObject,
  учесть cross-account/region/security ограничения.
```

### Пример 4. TQ interval tuning

Плохая:

```text
Уменьшить interval TQ.
```

Хорошая:

```text
Observation:
  DAG depth добавляет idle time между уровнями.

Hypothesis:
  Для chain depth=5 уменьшение interval с 5s до 1s снижает total time,
  но увеличивает DB polling load не более чем на X.

Experiment:
  На mini-TQ и/или локальном TQ замерить latency и DB query rate.
```

---

## 9. Главные decision tables

### 9.1 Execution path table

Заполняется после Module 7.

| Class of audience | Current DAG | One-SQL CH | Cache | Incremental | Bitmap | Recommendation |
|---|---|---|---|---|---|---|
| Simple static predicates |  |  |  |  |  |  |
| Large dynamic audience |  |  |  |  |  |  |
| Repeated params |  |  |  |  |  |  |
| Range-heavy predicates |  |  |  |  |  |  |
| External S3 reference |  |  |  |  |  |  |
| Freshness-critical |  |  |  |  |  |  |

### 9.2 Risk table

| Idea | Correctness risk | Freshness risk | Operational risk | Contract risk | Rollback complexity | Owner risk |
|---|---|---|---|---|---|---|
| One-SQL fast path |  |  |  |  |  |  |
| Result registry |  |  |  |  |  |  |
| Cache by params hash |  |  |  |  |  |  |
| Bitmap serving |  |  |  |  |  |  |
| S3 CopyObject |  |  |  |  |  |  |
| LISTEN/NOTIFY |  |  |  |  |  |  |

### 9.3 Evidence table

| Claim | Evidence | Artifact | Confidence | Missing data |
|---|---|---|---|---|
|  |  |  | Low/Medium/High |  |

Это полезно перед RFC. Любое сильное утверждение должно иметь evidence.

---

## 10. Что должно быть в финальном пакете

К концу полного курса у тебя должен быть не один файл, а пакет:

```text
1. Domain map
2. Contract map
3. Baseline numbers
4. TQ/DAG analysis
5. PG queue benchmark
6. CH/S3 execution report
7. Alternatives decision table
8. Production readiness report
9. Stakeholder map
10. RFC draft
```

Минимальный senior-level DoD:

```text
[ ] Могу за 10 минут нарисовать end-to-end путь аудитории.
[ ] Могу назвать top bottleneck с числами.
[ ] Могу объяснить, почему DAG нужен не всегда.
[ ] Могу сравнить DAG / SQL / cache / bitmap / incremental.
[ ] Могу объяснить failure modes и rollback.
[ ] Могу показать хотя бы один pet-project/lab.
[ ] Могу показать хотя бы один real-code experiment.
[ ] Могу защитить RFC перед инженерами и продуктом.
```

---

## 11. Первый практический проход

Если хочется начать прямо сейчас и не думать:

```text
Step 1.
  Создай practice/reading-log.md и practice/experiments/.

Step 2.
  Открой Module 1.
  Сделай audience-domain artifact.

Step 3.
  Открой Module 2.
  Сделай contract map.

Step 4.
  Не переходи к оптимизациям, пока нет baseline из Module 3.

Step 5.
  После baseline выбери самую болезненную область:
    - если orchestration/TQ -> Module 4/5;
    - если CH/data path -> Module 6/7;
    - если production risk -> Module 8;
    - если уже есть идея -> Module 9 draft, но только с evidence.
```

---

## 12. Главное правило курса

Не пытайся "выучить всё" линейно.

Двигайся так:

```text
Система -> вопрос -> теория -> код -> pet-проект -> эксперимент -> числа -> вывод -> RFC.
```

Если после блока у тебя нет артефакта, значит блок ещё не закончен.

Если после идеи у тебя нет baseline, значит это ещё не инженерное предложение.

Если после эксперимента нет correctness check, значит numbers нельзя использовать для решения.

Если после RFC нет rollout/rollback, значит это не production proposal.

Финальная цель:

```text
Не "я изучил A-P / Target / TQ".

А:

"Я понимаю систему, вижу её ограничения, умею проверять гипотезы,
могу менять код как эксперимент, мерить эффект, доказывать correctness
и оформлять улучшения так, чтобы их можно было безопасно внедрять".
```
