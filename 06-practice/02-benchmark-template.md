# 02. Benchmark Template

> Каждый замер должен быть воспроизводим, иметь fix-fixed конфиг и явную дисперсию. Без этого числа в RFC не выдержат разбора.

---

## Шапка бенчмарка

```md
# Benchmark: <название>

## Дата
2026-MM-DD

## Кто прогонял
@aisidnev

## Окружение
- Где: local / staging / prod-shadow
- Версия A-P: <git sha>
- Версия Target: <git sha>
- Версия TQ: <maven version>
- PG: 15.x
- ClickHouse: 23.x (если используется)
- Kafka: 3.x

## Ресурсы
- CPU: 4 cores, 2.5GHz
- RAM: 16GB
- Disk: SSD NVMe

## Конфиги
- TQConfig.interval: 5s
- TQConfig.maxConcurrentTasks: 5
- ClickHouse settings: max_threads=8, ...

## Цель
- что замеряем (compute time / total lead / throughput / ...)
```

---

## Параметры эксперимента

```md
## Inputs
- Audience type: STATIC
- Predicates: clientParams.cardType=DEBIT, regions=[Москва], subscriptions={include:[PRO]}
- Estimated rows: 350k

## Прогоны
- N runs: 5
- Cache state: cold (DROP MARK CACHE перед каждым)
- Order: randomized
```

---

## Результаты

```md
## Raw numbers (per run)

| run | compute_ms | export_ms | delivery_ms | total_ms |
|-----|-----------|-----------|-------------|----------|
| 1   | ...       | ...       | ...         | ...      |
| 2   | ...       | ...       | ...         | ...      |
| 3   | ...       | ...       | ...         | ...      |
| 4   | ...       | ...       | ...         | ...      |
| 5   | ...       | ...       | ...         | ...      |

## Aggregated

| metric | p50 | p95 | min | max | mean | stddev |
|--------|-----|-----|-----|-----|------|--------|
| compute_ms  |  |  |  |  |  |  |
| export_ms   |  |  |  |  |  |  |
| delivery_ms |  |  |  |  |  |  |
| total_ms    |  |  |  |  |  |  |
```

---

## Correctness

```md
## Verification
- [ ] count() совпадает с reference
- [ ] sample(100) — все client_id присутствуют в reference
- [ ] checksum партиций совпадает
```

---

## Анализ

```md
## Where time goes
- compute: ... (ClickHouse SELECT, distribution по этапам если можно)
- export: ... (S3 multipart upload)
- delivery: ... (Kafka publish + ack downstream)

## Bottleneck candidate
<какой слой / какая операция доминирует>

## Что бы ускорило
1. ...
2. ...
3. ...
```

---

## Воспроизводимость

```md
## How to repro

```bash
# Команды/SQL
SET MAX_THREADS=8;
SELECT ...
```

```sql
EXPLAIN PIPELINE
SELECT ...
```

## Где сами raw числа
- логи: /var/log/...
- Grafana panel: <link>
- Kafka consumer-lag screenshot: <path>
```

---

## Известные искажения

```md
## Caveats
- Прогон на staging — данные в 5x меньше прода.
- В кластере были fluctuations (другие нагрузки).
- Test может быть biased на cache от предыдущего прогона.
```

---

## Ссылки

```md
## See also
- experiment id: 06-practice/01-experiment-backlog.md#experiment-N
- related sprint: 05-weekly-sprints/sprint-NN-...
- relevant track: 04-learning-tracks/...
```

---

## Чек-лист перед commit бенчмарка

```text
[ ] N >= 3 прогонов (лучше 5).
[ ] p50 + p95 + stddev.
[ ] Конфиги фиксированы.
[ ] Correctness check.
[ ] Воспроизводимый рецепт (команда / SQL).
[ ] Известные искажения вынесены отдельно.
[ ] Ссылка на эксперимент / спринт / трек.
```

---

## Антипаттерны

```text
- "взял один прогон, считаю baseline" — нет дисперсии.
- "сравниваю числа без correctness check" — могу мерять не то.
- "не зафиксированы конфиги" — нельзя повторить.
- "прогнал в час пик и в час null" — биас по нагрузке.
- "не задокументировал, как ставил окружение" — забуду через 2 недели.
```
