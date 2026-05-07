# Track 09 — Bitmap / Inverted Index / Audience Serving

## Зачем учить
Если ввести serving-сервис с bitmap-индексами, набор операций «intersect / union / negate / count» становится миллисекундным. Это потенциально радикально ускоряет popular-аудитории и аналитику. Но bitmap — не серебряная пуля, нужно понимать, где он не помогает.

## Где это есть в проектах
- Сейчас bitmap не используется (поиск по коду подтвердит).
- Косвенно: `IncludeExclude` в `AuditoryParams` — концептуально это set-операции, идеальные для bitmap.

## Что изучить
```text
1. Roaring bitmaps: container types (array, bitmap, run), сжатие, intersect cost.
2. RoaringBitmap Java и Go библиотеки.
3. Inverted index: term → posting list. Применимо к "city → [client_ids]".
4. Когда bitmap проигрывает: range predicates, geo, hash из S3.
5. Build pipeline: «когда обновлять index», append-only vs rebuild.
6. Persistent bitmaps: serialization, хранение на S3 / EBS.
7. Serving-сервис паттерн: in-memory + warmup, gRPC API.
8. Comparison: ClickHouse `bitmap` функции (groupBitmapAnd, bitmapAnd) — нативный CH-путь.
```

## Практика на проекте
- Маленький Kotlin-проект: 10M client_id-ов, разделённых на 100 «сегментов» (city+age_bracket+subscription).
- Сериализовать как RoaringBitmap, замерить:
  - размер на диске (vs CSV);
  - время `bitmapAnd(A, B)` — intersect;
  - время `bitmapCount` — оценка размера аудитории.
- Сравнить с `groupBitmapAnd` в CH на тех же данных.
- Понять, можно ли поверх bitmap построить «эстимейт rows» для Planner.

## Артефакты
- Бенчмарк-репозиторий «bitmap-vs-sql».
- 1-страничный отчёт «когда bitmap, когда CH-bitmap, когда обычный CH-SELECT».
- Эксперимент №2 в `06-practice/01-experiment-backlog.md` заполнен числами.

## Self-check
1. В чём разница между bitmap и inverted index — это одно и то же?
2. Когда RoaringBitmap проигрывает простому HashSet?
3. Кто строит и обновляет bitmap — пайплайн от source-источника или отдельный сервис?
4. Что делать с range-predicates (income from-to) — bitmap-friendly они или нет?
5. Почему CH `groupBitmapAnd` может быть лучше отдельного serving-сервиса?

## DoD
```text
[ ] Бенчмарк-репозиторий.
[ ] Эксперимент №2 заполнен.
[ ] Понимание границ применимости.
```
