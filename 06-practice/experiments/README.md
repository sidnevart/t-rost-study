# Experiments

Папка для конкретных эксперимент-репортов. Шаблон — `06-practice/templates/experiment-template.md`.

## Naming

```
EXP-YYYYMMDD-NN-<short-slug>.md
```

Примеры:
- `EXP-20260512-01-bitmap-vs-postgres-join.md`
- `EXP-20260518-01-coroutines-throughput.md`

Если эксперимент включает raw бенчмарк — кладём в `06-practice/benchmarks/` по `02-benchmark-template.md` и ссылаемся отсюда.

## Workflow

1. Скопировать `../templates/experiment-template.md` → новый файл с правильным ID.
2. Сначала заполнить **Контекст / Гипотеза / Метод / Setup** — до того, как начать что-то мерить.
3. Прогнать → заполнить **Результаты / Анализ / Decision**.
4. **Next steps** превратить в Jira-задачи через скилл `jira-task-from-research`.
5. Если эксперимент породил архитектурное изменение — обновить LikeC4 модель через `likec4-diagrams` и встроить SVG.

## Index

| ID | Дата | Slug | Статус | Decision |
|----|------|------|--------|----------|
| _add rows here_ | | | | |
