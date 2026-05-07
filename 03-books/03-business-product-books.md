# 03. Business / Product / Management Books

> Engineer ≠ только код. Чтобы RFC прошёл — нужно понимать продукт, бизнес, людей. Ниже — минимальный набор, без которого staff-уровень не зайдёт.

---

## *The Mom Test*, Rob Fitzpatrick

### Почему мне
Я часто задаю команде/PM вопросы вроде «удобно ли так?», «нравится ли подход?». Это плохие вопросы — они ведут к лжи. Книга учит спрашивать про факты.

### Связь с проектами
- Когда выясняю SLA с продакта — вместо «нужно ли быстрее?» спрашивать «когда последний раз кампания запустилась с задержкой и что произошло?».
- Когда выясняю с бэкендером — вместо «как лучше сделать?» спрашивать «что больше всего болело за последний квартал?».

### Артефакт
- Переписать `01-context/07-open-questions-for-team.md` под Mom Test-стиль.
- 30-минутное discovery-интервью с PM, conspectus в `memory_bank` или в открытых вопросах.

---

## *Continuous Discovery Habits*, Teresa Torres

### Почему мне
Архитектурное предложение — это тоже продукт. Без discovery — это «начальник захотел». Хочу научиться валидировать гипотезы регулярно.

### Связь
- Гипотезы из `02-architecture/04-graph-vs-sql-vs-bitmap-vs-cache.md` нужно валидировать с командой в течение 4-8 недель, не «один раз решил и поехал».
- Opportunity-Solution Tree даст структуру: пройти от бизнес-выгоды до технического решения.

### Артефакт
- Opportunity-Solution Tree для RFC.
- 3 discovery-вопроса в неделю, ответы фиксировать.

---

## *Inspired*, Marty Cagan

### Почему мне
Чтобы говорить с продуктом на одном языке. Особенно: discovery vs delivery, риски (value/usability/feasibility/viability).

### Связь
- Аудитории — это feature для других продуктов (push, ads, cb-platform). Понимать их product-context.
- Выделять «product opportunities»: где аудитории недостаточно эффективны = где нужен RFC.

### Артефакт
- Product brief на 1 страницу: «Audience Platform 2026 — opportunity, value, risks, milestones».

---

## *Obviously Awesome*, April Dunford

### Почему мне
Если RFC будет внутренний — его нужно «продать». Позиционирование решает.

### Связь
- Чем RFC отличается от «оптимизировать TQ»? — это разные обещания.
- Кто целевая аудитория RFC: A-P команда, Target команда, продакт, инфраструктура?

### Артефакт
- Positioning canvas: alternatives → unique attributes → value → who-cares.

---

## *Good Strategy / Bad Strategy*, Richard Rumelt

### Почему мне
Hard rule: strategy = diagnosis + guiding policy + coherent actions. Без diagnosis — «список улучшений», не стратегия.

### Связь
- Diagnosis из `01-context/06-compute-export-delivery-split.md`: где именно слабое место.
- Guiding policy: «default cheap, opt-in expensive», «layer-by-layer SLA».
- Coherent actions: phase-rollout из `02-architecture/05-execution-router-planner.md`.

### Артефакт
- Strategy kernel на 1 страницу. Положить в `07-final-rfc/01-final-rfc-outline.md`.

---

## *High Output Management*, Andrew Grove

### Почему мне
Книга про управленческий рычаг. Я не менеджер сейчас, но проектная инициатива требует тех же навыков (1:1, ясные цели, индикаторы, делегирование).

### Связь
- Indicator pairs: «скорость + качество». Для пайплайна аудиторий тоже работает.
- Task-relevant maturity: разные люди в команде — разной готовности.

### Артефакт
- Шаблон 1:1 с тимлидом A-P / Target.
- 2-3 indicator pairs для пайплайна.

---

## *Crossing the Chasm*, Geoffrey Moore

### Почему мне
Внутри компании rollout фичи проходит ту же кривую: early adopters → mainstream → laggards.

### Связь
- Phase 0/1 RFC — early adopters (1-2 команды, готовы тестировать новые пути).
- Phase 3+ — mainstream (требует стабильности, документации, training).

### Артефакт
- Список «pragmatist» и «innovator» команд в компании, которые могут стать early-adopters Planner-а.

---

## *Sales Acceleration Formula*, Mark Roberge

### Почему мне
Хочу научиться превращать «идею» в «принятое решение» в большой компании. RFC — это продажа.

### Связь
- Process > heroics: чем формализованнее путь от идеи до approval, тем выше шанс пройти.
- Metrics-driven: какие индикаторы покажут, что RFC «работает» после внедрения.

### Артефакт
- «Sales funnel» для RFC: gathered feedback → reviewed → approved → adopted.

---

## *The Startup Owner's Manual*, Steve Blank, Bob Dorf

### Почему мне
Customer development model. Применимо даже к внутренним «клиентам» (downstream-команды).

### Связь
- «Get out of the building» — не сидеть в коде, идти к downstream-командам и спрашивать про их боль.

### Артефакт
- 3 интервью с downstream-командами (cb-platform / extra-cb / push). Конспекты.

---

## *Team Topologies*, Skelton & Pais

### Почему мне
Архитектура и топология команд связаны (Conway's law). Понимать, какие границы команд / систем — реальные и какие — артефакт.

### Связь
- A-P ↔ Target — две команды или одна? Реальный API между ними как границы.
- TQ — library, у неё есть owner-команда (ninja-team по README A-P).
- Если предложу новый сервис (Planner) — кто его владелец?

### Артефакт
- Team-topology diagram: stream-aligned / platform / enabling / complicated subsystem.

---

## *Empowered*, Marty Cagan

### Почему мне
Empowered teams принимают решения. Вопрос: насколько empowered наша команда? Что нужно сделать, чтобы быть empowered для архитектурных предложений?

### Связь
- Trust battery с менеджментом / архитектурой.
- Outcomes vs outputs — RFC должен быть про outcome, не «давайте поменяем технологию».

### Артефакт
- Самооценка empowerment: где требуется усилие.

---

## *The Hard Thing About Hard Things*, Ben Horowitz

### Почему мне
Большие архитектурные изменения создают политические проблемы. Нужны ментальные модели для «как принимать неудобные решения».

### Связь
- RFC может потребовать остановки чьей-то текущей работы. Как это сообщить?
- Trade-off между «быстро» и «правильно».

### Артефакт
- 3 «hard things» в RFC: что мы признаем неудобным.

---

## Принципы чтения этих книг

```text
1. Читать "под себя", не подряд.
   - Если завтра — сложный разговор с PM → срочно главу из Mom Test.
   - Если предстоит rollout → срочно Crossing the Chasm.

2. Конспект — не полный, а только actionable.
   "В разговоре с X буду пробовать пункт Y".

3. Через 1 неделю после книги — рефлексия:
   "Какие 2 разговора стали лучше?"

4. Для управленческих книг (Grove, Cagan, Reilly) — 1 раз попробовать на практике каждую идею.
   Лучше уж имитировать, но всё-таки делать.
```
