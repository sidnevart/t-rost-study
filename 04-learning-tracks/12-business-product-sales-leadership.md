# Track 12 — Business / Product / Sales / Leadership

## Зачем учить
Хорошее техническое предложение не побеждает плохую коммуникацию. Если RFC не пройдёт review — вся техническая работа уйдёт в стол. Чтобы быть staff-уровня, инжинер должен уметь продавать архитектуру.

## Где это есть в проектах
- Это не про код. Это про взаимодействие с людьми вокруг A-P/Target/TQ.
- В learning-os — `03-books/03-business-product-books.md` + ниже.

## Что изучить
```text
Communication & influence
  1. Mom Test (Fitzpatrick) — как задавать вопросы, чтобы получать факты.
  2. Empowered (Cagan) — как становиться empowered инженером.
  3. Staff Engineer's Path (Reilly) — sphere of influence, glue work, политика.

Strategy
  4. Good Strategy / Bad Strategy (Rumelt) — diagnosis + guiding policy + actions.
  5. Crossing the Chasm (Moore) — внутренний rollout = тоже early/mainstream/laggards.

Product
  6. Inspired (Cagan) — as a stakeholder для аудиторий.
  7. Continuous Discovery (Torres) — гипотезы валидируются непрерывно.
  8. Obviously Awesome (Dunford) — позиционирование RFC внутри компании.

Management & ops
  9. High Output Management (Grove) — leverage, indicator pairs, 1:1.
  10. Team Topologies — как Conway's law влияет на нашу архитектуру.
  11. Sales Acceleration Formula (Roberge) — funnel для RFC adoption.
  12. The Hard Thing About Hard Things (Horowitz) — неприятные решения.
```

## GPT-ускоритель

**Шаг 1 — Learning Curator** → вставь из `gpt-prompts.md` → Track 12 → Шаг 1.
**Шаг 3 — Deep Explanation** (если застрял на stakeholder management) → вставь из `gpt-prompts.md` → Track 12 → Шаг 3.
**Шаг 4 — Self-check** → вставь из `gpt-prompts.md` → Track 12 → Шаг 4.

---

## Практика на проекте

### Задача 1 — Stakeholder map (обязательно, до RFC)
Заполни таблицу для своего RFC (5–7 человек):

| Имя/Роль | Что им важно | Потенциальные возражения | Как вовлечь |
|---|---|---|---|
| Тимлид A-P | | | |
| PM продукта | | | |
| Архитектор | | | |
| Downstream команда | | | |
| ... | | | |

Правило: если не знаешь «что им важно» — это и есть тема следующего discovery-разговора.

### Задача 2 — Discovery conversation: шаблон + 3 разговора
Перед каждым разговором заполни:
```
Гипотеза, которую проверяю: [одно предложение]
Ожидаемый ответ: [да/нет/иначе]
Вопросы (по Mom Test — не leading):
  1. [вопрос о прошлом опыте, не о будущем]
  2. [вопрос о конкретном примере]
  3. [вопрос о приоритетах]
```
После разговора:
```
Что узнал: 
Что подтвердилось:
Что опровергнуто:
Следующий шаг:
```
Сохрани в `practice/track-12-discovery/conversation-NN.md`.

### Задача 3 — Strategy kernel для RFC (1 страница)
Структура по Rumelt:
```
Diagnosis: [что именно сломано / не оптимально — в одном абзаце]
Guiding policy: [принцип, которым руководствуемся при выборе решения]
Coherent actions:
  1. [первое конкретное действие]
  2. [второе конкретное действие]
  3. [третье конкретное действие]
```
Тест: если кто-то прочитает только Diagnosis — поймёт ли проблему без технических деталей?

### Задача 4 — Positioning canvas RFC
Заполни шаблон (по Obviously Awesome):
```
Для кого: [кто читает RFC]
Проблема: [что сейчас болит]
Наш подход: [что предлагаем]
Ценность: [что изменится — в числах]
Альтернатива: [что если ничего не делать]
Почему сейчас: [почему важно делать это сейчас, а не потом]
```

### Задача 5 — Elevator pitch (30 секунд)
Написать и 5 раз произнести вслух:
```
«[Контекст: кому болит]. Сейчас [проблема в числах]. 
Мы предлагаем [решение одним предложением]. 
Это даст [результат в числах]. 
Риск — [один главный риск].»
```
Тест: если после этого человек задаёт уточняющий вопрос — pitch работает.

## Артефакты
- `practice/track-12-stakeholder-map.md` — заполненная таблица.
- `practice/track-12-discovery/` — 3+ разговора по шаблону.
- `practice/track-12-strategy-kernel.md` — diagnosis + policy + actions.
- `practice/track-12-positioning-canvas.md` — 6 вопросов.
- `practice/track-12-elevator-pitch.md` — 30-секундный pitch.

## Self-check
1. Кто 5 ключевых stakeholder-ов твоего RFC? У каждого — что им важно?
2. Какое первое возражение прозвучит на review и как ты на него ответишь?
3. Какой самый дешёвый эксперимент, который убедит скептиков?
4. Если бы был 1 час с CTO — какой 1 слайд показал бы?
5. Что лично ты получишь, если RFC пройдёт? Что — если не пройдёт?
6. Прочитал ли ты страницу из Mom Test про «мамин тест»? Можешь ли применить к вопросам в discovery?
7. Каков твой Diagnosis в 1 предложении, без технических деталей?

## DoD
```text
[ ] Stakeholder map (5–7 человек, с возражениями и стратегией вовлечения).
[ ] Strategy kernel (1 страница: diagnosis + policy + actions).
[ ] 4+ discovery conversations по шаблону за квартал.
[ ] Positioning canvas RFC заполнен.
[ ] Elevator pitch — 30 секунд, произнесён вслух 5 раз.
```
