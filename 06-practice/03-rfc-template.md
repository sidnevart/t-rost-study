# 03. RFC Template

> Шаблон финального RFC. Цель — собрать всё, что наработано в learning-os, в один документ, который пройдёт review.

---

```md
# RFC: <Название инициативы>

**Author:** <name>
**Date:** YYYY-MM-DD
**Status:** Draft | Review | Approved | Rejected | Implemented
**Reviewers (required):** <names>
**Reviewers (optional):** <names>
**Related:** <links to docs / Jira / previous RFCs>

---

## TL;DR (3-4 строки)
<одна формулировка проблемы + одно решение + один ожидаемый эффект>

---

## 1. Problem statement

### 1.1 Что мы наблюдаем
<факты, не интерпретации. Numbers from sprint 03 / experiment 8>

### 1.2 Кому это болит
<кто страдает: какая команда, какой канал, какой бизнес-метрик>

### 1.3 Почему сейчас
<что изменилось / какая стратегическая цель / какой trigger>

---

## 2. Current architecture

### 2.1 End-to-end flow
<reference to 01-context/05-end-to-end-audience-flow.md или картинка>

### 2.2 Где это в коде
<file:line ссылки>

### 2.3 Какие гарантии текущая система держит
<delivery, ordering, idempotency, ...>

---

## 3. Baseline numbers

### 3.1 Compute / Export / Delivery breakdown
<таблица из experiment 8 / sprint 3>

### 3.2 Размеры аудиторий и распределение
<p50/p95 size, daily counts, ...>

### 3.3 Error / retry / timeout rates
<метрики>

---

## 4. Bottlenecks (diagnosis)

### 4.1 Что доминирует и почему
<ссылки на эксперименты>

### 4.2 Узкие точки
1. ...
2. ...
3. ...

---

## 5. Alternatives considered

### 5.1 Decision table
<твоя таблица сравнений из экспериментов — с числами по 7 критериям>

### 5.2 Вариант A — <название>
- Когда применим
- Ожидаемый выигрыш (число)
- Риски
- Почему отклонён или выбран

### 5.3 Вариант B — <название>
- ...

### 5.N Вариант Z — оставить как есть
- Почему не делать тоже требует обоснования

---

## 6. Recommended path

### 6.1 Что предлагаем
<один или комбинация путей>

### 6.2 Почему именно так
<с обоснованием trade-off-ов>

### 6.3 Что НЕ делаем (явно)
<какие вещи отвергнуты и почему>

---

## 7. Architecture proposal

### 7.1 Components
<что новое, что изменяется, что остаётся>
- Компонент A (новый / изменённый): назначение
- Компонент B: ...
- Что явно не трогаем и почему

### 7.2 Sequence diagrams
<mermaid / plantuml>

### 7.3 Контракты
- Avro changes
- REST changes
- Kafka topics — новые / изменённые

### 7.4 Что меняется в каждом сервисе
- A-P: ...
- Target: ...
- TQ: ...

---

## 8. Rollout plan

### 8.1 Phases

```text
Phase 0 — <shadow / audit only>
  - Что делаем: ...
  - Метрики наблюдения: ...
  - Критерии перехода к Phase 1: ...

Phase 1 — <первый production rollout>
  - Cohort: ... % трафика / какой критерий отбора.
  - Метрики успеха: ...
  - Критерии stop: ...

Phase N — <дальнейшее расширение, если Phase 1 прошёл>
  - ...
```

> Нет фиксированного числа фаз. Есть критерии перехода и stop.

### 8.2 Feature flags
- Где (config keys).
- Кто может включать / выключать.
- Default state.

### 8.3 Cohort selection
- На основе чего разбиваем (audience_id mod, channel, ...).

---

## 9. Rollback plan

### 9.1 Per phase rollback
- Phase 0: <как откатить. Нет production change? Ничего откатывать.>
- Phase 1: <feature flag / dark launch → как быстро вернуть всё назад. Time to recover: X min.>
- Phase N: <особые случаи: что нельзя автоматически откатить и почему>

### 9.2 Auto-rollback triggers
- error_rate > X% за Y minutes → автоматически выключаем.
- p95 lead time > 2x baseline → alert + auto-rollback.

---

## 10. Metrics / Success criteria

### 10.1 SLO targets
| Metric | Baseline | Target | After Phase |
|---|---|---|---|
| <метрика 1> | XX | YY | N |
| <метрика 2> | XX | YY | N |

> Baseline — из твоих измерений, не из предположений.

### 10.2 Anti-goals
- Не ухудшать error rate.
- Не повышать complexity для debugging.
- Не ломать downstream-консьюмеров.

---

## 11. Risks

### 11.1 Technical
- ...

### 11.2 Operational
- ...

### 11.3 Organizational
- ...

### 11.4 Mitigations
- ...

---

## 12. Open questions

1. <ссылка на 01-context/07-open-questions-for-team.md>
2. ...

---

## 13. Implementation effort

### 13.1 Estimated FTE-weeks
- A-P backend: ...
- Target backend: ...
- TQ contributions: ...
- Infra / SRE: ...
- Product / docs: ...

### 13.2 Dependencies
- Какие команды должны участвовать.
- Какие downstream-консьюмеры — обязательно.

### 13.3 Critical path
- Что делается первым, что блокирует что.

---

## 14. Appendix

### 14.1 References
- 01-context/05-end-to-end-audience-flow.md
- 02-architecture/04-graph-vs-sql-vs-bitmap-vs-cache.md
- 02-architecture/05-execution-router-planner.md
- 06-practice/01-experiment-backlog.md
- ...

### 14.2 Glossary
<термины, которые могут быть незнакомы читателю>
- <термин> — определение

### 14.3 Decision log
<решения, принятые в процессе написания RFC, и почему>
- 2026-MM-DD: ...

---

## 15. Approvals

| Reviewer | Status | Date | Comment |
|----------|--------|------|---------|
| ...      | ✓/✗   | ...  | ...     |
```

---

## Чек-лист перед отправкой RFC

```text
[ ] TL;DR умещается в 3-4 строки.
[ ] Problem statement содержит numbers, не «нам кажется».
[ ] Все альтернативы рассмотрены, отклонённые — с обоснованием.
[ ] Rollout plan с фазами и временными рамками.
[ ] Rollback plan с конкретными триггерами.
[ ] Success metrics с baseline + target.
[ ] Open questions явно обозначены.
[ ] Implementation effort оценён в FTE-weeks.
[ ] Reviewers list согласован.
```
