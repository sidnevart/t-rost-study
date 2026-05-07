# 02. Senior Initiative — Definition of Done

> Что именно мне нужно достичь, чтобы считать инициативу выполненной на staff/senior уровне. Не «прочитал и понял», а «продвинул систему вперёд».

---

## A. Knowledge DoD

```text
[ ] Могу за 10 минут на доске нарисовать end-to-end путь любой аудитории.
[ ] Знаю file:line-ссылки на 10 ключевых мест в каждом из трёх проектов.
[ ] Понимаю гарантии TQ (delivery, ordering, recovery, retry).
[ ] Понимаю BACKWARD-семантику Avro контракта.
[ ] Знаю, какой layer (compute/export/delivery) доминирует для каждого размера.
[ ] Прошёл self-check questions ≥ 70%.
```

---

## B. Artifact DoD (8 ключевых)

```text
[ ] ap-end-to-end-trace.md             — sequence diagram + numbers
[ ] target-contract.md                  — REST + Kafka + Avro + JWT
[ ] compute-export-delivery-times.md    — таблица для 3 размеров
[ ] graph-vs-alternatives.md            — decision table с числами
[ ] tq-delivery-guarantees.md           — guarantees + failure modes
[ ] clickhouse-baseline.md              — карта таблиц + аннотированный план
[ ] bitmap-experiment-report.md         — числа RoaringBitmap vs CH-bitmap vs CH
[ ] final-rfc.md                        — заполненный шаблон 03-rfc-template.md
```

Все артефакты:
- лежат в `practice/` или соответствующих файлах learning-os,
- ссылаются на конкретный код (file:line),
- имеют дату и автора,
- проверены на correctness (где применимо).

---

## C. Experiment DoD (10 экспериментов)

```text
[ ] 10 экспериментов из 06-practice/01-experiment-backlog.md либо выполнены, либо явно отложены с обоснованием.
[ ] Каждый эксперимент имеет numbers с дисперсией (≥ 3 прогона).
[ ] Каждый эксперимент имеет correctness check.
[ ] Каждый эксперимент имеет связь с конкретной частью RFC.
```

---

## D. RFC DoD

```text
[ ] RFC написан по шаблону 06-practice/03-rfc-template.md.
[ ] TL;DR ≤ 4 строк.
[ ] Все альтернативы рассмотрены, отклонённые — обоснованы.
[ ] Decision table с реальными числами.
[ ] Phase rollout с временами и feature-flag-ами.
[ ] Rollback plan с автоматическими триггерами.
[ ] Success metrics с baseline и target.
[ ] Open questions явно перечислены.
[ ] Implementation effort оценён в FTE-weeks.
[ ] Stakeholder map составлен (5-7 человек).
```

---

## E. Review DoD

```text
[ ] Pre-review с тимлидом A-P (1:1).
[ ] Pre-review с тимлидом Target (1:1).
[ ] Pre-review с product manager.
[ ] Pre-review с SRE / on-call.
[ ] RFC отправлен в формальную review.
[ ] Получены feedback от ≥ 5 людей.
[ ] Все feedback обработаны (либо учтены, либо обоснованно отклонены).
```

---

## F. Books DoD

```text
[ ] Прочитано ≥ 8 engineering книг из списка (по 1 в месяц в среднем).
[ ] Прочитано ≥ 8 business книг из списка.
[ ] Каждая книга → артефакт в проекте.
[ ] Reading log актуален.
```

---

## G. Tracks DoD

```text
[ ] Все 12 learning tracks пройдены.
[ ] По каждому — артефакт + self-check ответы.
[ ] Все 12 файлов в 04-learning-tracks/ имеют пометку «✓ done» (рукой).
```

---

## H. Sprints DoD

```text
[ ] Все 8 спринтов прошли в течение 2-6 месяцев.
[ ] По каждому — финальный артефакт в practice/.
[ ] DoD каждого спринта закрыт.
```

---

## I. Rollout DoD (если RFC принят)

```text
[ ] Phase 0 (shadow mode) реализован согласно rollout plan.
[ ] Метрики из RFC реально мониторятся.
[ ] Auto-rollback триггеры конфигурированы.
[ ] Я могу показать «было / стало» tabular на реальных данных.
[ ] Phase 1 production cohort — данные собираются.
```

---

## J. Career DoD (для себя)

```text
[ ] У меня есть Sphere of Influence map (по track 12).
[ ] У меня есть как минимум 3 stakeholder-а, которые публично подтверждают вклад.
[ ] У меня готовый 1-слайд elevator pitch инициативы.
[ ] Я провёл ≥ 1 публичной презентации внутри компании на эту тему.
[ ] Я знаю свой следующий шаг карьеры (тимлид / staff / разные команды).
```

---

## Анти-цели (что НЕ считается DoD)

```text
- «Прочитал DDIA» — без артефакта это не done.
- «Написал документ» без чисел — это не done.
- «Сделал прототип» без тестов correctness — это не done.
- «Согласовал устно» без письменного approve — это не done.
- «Сделал слайд» без metrics — это не done.
- «Поговорил с тимлидом» без записи outcome — это не done.
```

---

## Метрики прогресса

| Метрика | Цель | Текущая |
|---|---|---|
| Sprints completed | 8 | 0 |
| Tracks completed | 12 | 0 |
| Engineering books finished | 8 | 0 |
| Business books finished | 8 | 0 |
| Experiments executed | 10 | 0 |
| Artifacts produced | 8 | 0 |
| Stakeholder conversations | 12 | 0 |
| RFC drafts | 1 | 0 |
| Production rollout phases | ≥ 1 | 0 |

(обновлять раз в месяц)

---

## Когда считать инициативу провалившейся

```text
- Через 3 месяца < 2 спринтов завершены.
- Через 6 месяцев < 5 артефактов.
- Через 9 месяцев нет даже черновика RFC.
- Цель потеряла актуальность (бизнес-перестроились).
- Я не вижу outcome для себя профессионально.

В таком случае — open & honest pivot. Записать причину провала в отдельный файл.
Из этого тоже можно учиться.
```
