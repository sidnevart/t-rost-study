# 05. Artifact Checklist

> Главный анти-burnout инструмент: каждое усилие → артефакт. Если артефакта нет — не было «работы».

---

## После каждого спринта (8 спринтов × N артефактов)

```text
Sprint 1:
  [ ] practice/sprint-01-ap-end-to-end-map.md (sequence diagram + observations)
  [ ] обновлён 01-context/02-ap-domain-map.md «личные заметки»
  [ ] список 5 неясных мест в 01-context/07-open-questions-for-team.md

Sprint 2:
  [ ] practice/sprint-02-target-contract.md (full contract)
  [ ] таблица topic | producer | consumer | format | semantics
  [ ] заметка о JWT contract

Sprint 3:
  [ ] practice/sprint-03-numbers.md (compute/export/delivery таблица)
  [ ] обновлён 01-context/06-compute-export-delivery-split.md (раздел 6.7)
  [ ] 3 гипотезы для оптимизации

Sprint 4:
  [ ] practice/sprint-04-decision-table.md
  [ ] обновлён 02-architecture/04-graph-vs-sql-vs-bitmap-vs-cache.md (раздел 4.2)
  [ ] числа one-SQL vs DAG для одного use-case

Sprint 5:
  [ ] practice/sprint-05/ (pet-project на TQ)
  [ ] аннотированный TQScheduler.kt
  [ ] числа continuous=true

Sprint 6:
  [ ] practice/sprint-06-pg-queue/
  [ ] EXPLAIN до/после партиальных индексов
  [ ] LISTEN/NOTIFY demo

Sprint 7:
  [ ] practice/sprint-07-clickhouse-baseline.md
  [ ] карта CH-таблиц
  [ ] аннотированный план + MV experiment

Sprint 8:
  [ ] practice/sprint-08-bitmap-report.md
  [ ] числа bitmap vs CH
  [ ] вывод о границах применимости bitmap в твоём use-case
```

---

## После каждой книги

```text
[ ] Конспект 1-2 страницы.
[ ] 1 артефакт, привязанный к A-P/Target/TQ.
[ ] Запись в reading-log.md.
```

---

## После каждого learning track (12 треков)

```text
[ ] Самопроверка по 5 вопросам.
[ ] Артефакт (заметка / код / эксперимент).
[ ] Обновлён соответствующий файл в 01-context/ или 02-architecture/.
```

---

## После каждого эксперимента (10 в backlog)

```text
[ ] Numbers собраны.
[ ] Correctness проверен.
[ ] Записаны в шаблон benchmark (06-practice/02-benchmark-template.md).
[ ] Связан с конкретным use-case в RFC.
```

---

## Перед финальным RFC

```text
[ ] Пройден self-check (06-practice/04-self-check-questions.md) >= 70%.
[ ] Все 10 экспериментов либо выполнены, либо явно отложены с причиной.
[ ] Decision table заполнен числами.
[ ] Stakeholder map готов.
[ ] Strategy kernel сформулирован.
[ ] Rollout / rollback продуманы.
[ ] Open questions ≥ 50% закрыты.
[ ] Шаблон RFC из 06-practice/03-rfc-template.md полностью заполнен.
```

---

## Артефакт-минимум Senior DoD (см. 07-final-rfc/02-senior-initiative-dod.md)

```text
8 ключевых артефактов:
1. ap-end-to-end-trace.md
2. target-contract.md
3. compute-export-delivery-times.md
4. graph-vs-alternatives.md
5. tq-delivery-guarantees.md
6. clickhouse-baseline.md
7. bitmap-experiment-report.md
8. final-rfc.md
```

---

## Анти-паттерны (артефактного дисциплины)

```text
1. «Я понял эту тему» — не артефакт.
   → артефакт = что-то, что можно показать другому.

2. «У меня заметки в голове / черновиках» — не артефакт.
   → артефакт = в нужном md-файле learning-os.

3. «Я прочитал главу» — не артефакт.
   → артефакт = что я применил в проекте.

4. «Я прогнал бенчмарк, не зафиксировал» — не артефакт.
   → артефакт = в шаблоне 02-benchmark-template.md.

5. «Я обсудил с тимлидом» — не артефакт.
   → артефакт = заметка с outcome (что узнал, что подтвердилось).
```

---

## Регулярный ревью

```text
Раз в 2 недели:
  - Открыть этот файл.
  - Пройти по чек-листу: что сделано, что нет.
  - Если pending > 1 спринта → переоценить план.
  - Если burnout signs → отдохнуть, замедлиться.

Раз в месяц:
  - Открыть 03-books/04-how-to-read-books-with-project-practice.md → reading log.
  - Что из чтения реально применил?
  - Корректировать book plan, если нужно.

Раз в квартал:
  - Полный обзор learning-os.
  - Sphere of influence map (track 12).
  - Что РЕАЛЬНО изменилось в проектах из-за моих усилий?
```
