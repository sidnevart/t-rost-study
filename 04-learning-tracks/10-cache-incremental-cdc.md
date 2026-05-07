# Track 10 — Cache / Incremental / CDC

## Зачем учить
Самые лёгкие выигрыши лежат в «не считать заново то, что уже считали». Cache-by-definition-hash и incremental refresh — два инструмента, каждый со своей областью применимости.

## Где это есть в проектах
- В A-P **нет** cache-by-definition-hash (поиск подтвердит).
- В Target есть `MATCHING_DELTA_EXCLUSION` — это incremental на уровне исключений.
- DLH-процессы (`a-p/docs/processes/dlh/`) — потенциальные источники CDC.

## Что изучить
```text
1. Cache invariants: TTL, LRU, write-through, write-back, stale-while-revalidate.
2. Stable hashing: нормализация params + sha256 / blake3.
3. Negative caching: как кэшировать "результата ещё нет".
4. Cache stampede protection (single-flight, request coalescing).
5. CDC: Debezium, wal2json, logical decoding.
6. Incremental compute: low-water-mark, checkpoints.
7. Idempotent merge: применил delta дважды = один раз.
8. Reconciliation: периодический full-rebuild для catch drift.
9. Saga / Outbox для consistency между «compute» и «cache».
10. Time-travel: Iceberg / Delta Lake (для общего бэкграунда).
```

## Практика на проекте
- Прототип hash-функции для AuditoryParams (Go): нормализация + sha256.
- На staging БД посчитать: сколько уникальных params_hash на 10000 audience-id-ов.
  - Это — потолок cache-hit rate.
- Прочитать `MATCHING_DELTA_EXCLUSION` логику в Target: где watermark, как идемпотентность.
- Прототип маленький: in-memory cache + single-flight + TTL 1 час.

## Артефакты
- Скрипт hash + результаты выборки «уникальных params_hash на 10k».
- 1-страничная заметка «когда cache, когда incremental, когда оба».
- Раздел в `02-architecture/06-result-registry-model.md` пополнить.
- Эксперимент №6 и №8 в `06-practice/01-experiment-backlog.md` заполнены.

## Self-check
1. Почему важна нормализация params перед хэшированием?
2. Что такое cache stampede и как single-flight его решает?
3. Когда incremental предпочтительнее cache?
4. Что делать с drift в incremental — реконсиляция через сколько?
5. Чем outbox pattern помогает в consistency cache + DB?

## DoD
```text
[ ] Скрипт hash + цифра «уникальных params_hash на N audience».
[ ] Эксперимент №6 и №8 заполнены.
[ ] Решение: какой path подходит для каких типов аудиторий.
```
