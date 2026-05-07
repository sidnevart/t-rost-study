# Diagrams (LikeC4)

Architecture diagrams as code. Source — `.c4` файлы в этой папке. Скилл — `.claude/skills/likec4-diagrams`.

## Структура

```
diagrams/
├── model.c4         # элементы: actors / systems / containers / components
├── views.c4         # views: какие элементы куда идут
├── styles.c4        # (опционально) общие стили
└── generated/       # SVG/PNG, регенерируется CLI, в .gitignore
```

## Команды

```bash
# Live preview
npx likec4@latest start

# Экспорт всех views в SVG
npx likec4@latest export svg -o diagrams/generated

# Один view как PNG
npx likec4@latest export png --view containers -o diagrams/generated
```

## Embed в документ

```markdown
![Audience Platform — Containers](../diagrams/generated/containers.svg)
```

## Когда LikeC4, а когда Mermaid

- **LikeC4** — для **живой модели**: одно описание элементов, много views, много документов ссылаются.
- **Mermaid inline в md** — для **одноразового** наброска внутри одного документа.

Подробнее — `.claude/skills/likec4-diagrams/SKILL.md` и https://likec4.dev/dsl/.
