# 06. Compute / Export / Delivery split

> Когда заказчик слышит «аудитория за 2 секунды» — он подразумевает «через 2с её можно использовать в кампании». Когда инженер слышит то же самое — это часто значит «SQL вернул результат за 2с». Это разные числа.

---

## 6.1 Три разных времени

```text
Compute time     — посчитать множество client_id
Export time      — подготовить артефакт (CSV в S3, нарезка партиций, checksum, kafka-сообщение)
Delivery time    — доставить downstream-у (T-Segment / channel) и подтверждение
Total lead time  — от запроса заказчика до того момента, когда аудитория «доступна»
```

### Что можно посчитать дёшево, а что — дорого
| Время | Дешёвая часть | Дорогая часть |
|---|---|---|
| Compute | агрегаты, COUNT, есть индекс | join с big table, filter без index, sliding date window |
| Export | размер ≤ 100k, S3 PutObject | размер 10M+, нарезка партиций, multipart upload |
| Delivery | Kafka-publish | downstream-обработка (partition rebuild, t-segment ingest), confirmation roundtrip |
| Total | если кэш hit или precomputed | каждый раз с нуля + downstream → всё последовательно |

---

## 6.2 Где это видно в текущей системе

### Compute (Target + источники данных)
```text
Where: Target / TQ-задачи (auditory-loading-core .../gathering/job/...)
What:  обращается к ClickHouse / Postgres / S3-files / внешним сервисам.
Time:  доминирует размер аудитории + сложность predicate-set.
```

### Export (Target → S3 → Kafka → A-P → S3)
```text
Where: Target пишет CSV в S3 (target-send-auditory).
       A-P переупаковывает в свой S3 (`internal/service/s3/chunk/chunk_upload_service.go`).
What:  CSV нарезается на партиции по 500k записей (docs/components/kafka/...auditory.mdx).
Time:  size / S3 throughput. Двойная упаковка → ~2x IO.
```

### Delivery (A-P → T-Segment → каналы)
```text
Where: A-P пишет Kafka `t-segment.offline-upload.upload`.
       T-Segment читает, импортирует, отвечает в `t-segment.offline-upload.upload-info`.
What:  размер партиции + ack downstream + при необходимости batch updates каналам.
Time:  обычно вне SLA A-P, но влияет на total lead time.
```

---

## 6.3 Вопросы, которые стоит задать

```text
1. Compute-оптимизации ускорят total lead time — при каком условии?
2. Если export занимает больше compute — где bottleneck реально?
3. Delivery происходит после или параллельно с export?
4. Что бизнес реально ждёт: момент compute или момент downstream-готовности?
5. Если ускорить один слой в 10x — насколько уменьшится total lead time?
```

> Замерь сначала — потом отвечай. Не наоборот.

---

## 6.4 Что замерить (baseline)

| Метрика | Как считать |
|---|---|
| t_compute_start | первая `tq_task.date_started` в процессе |
| t_compute_end | последняя `tq_task.date_finished` (без EXPORT/SEND) |
| t_export_end | время записи последней партиции CSV в S3 |
| t_delivery_publish | время kafka-publish в `t-segment.offline-upload.upload` |
| t_delivery_ack | время ответа в `t-segment.offline-upload.upload-info` |
| size_clientids | `auditory.size` |
| partitions | количество CSV-файлов в S3 (по сообщению Kafka) |
| latency_compute | t_compute_end - t_compute_start |
| latency_export | t_export_end - t_compute_end |
| latency_delivery | t_delivery_ack - t_delivery_publish |
| total_lead | t_delivery_ack - HTTP_create_time |

> Шаблон для записи: `06-practice/02-benchmark-template.md`.

---

## 6.5 Почему «аудитория за 2-3 секунды» опасное обещание

```text
- 2-3с легко достижимы для compute мелкой аудитории на индексе.
- Но даже для 10k записей export+delivery + ack T-Segment могут быть 5-30с.
- Если перестроить план на reference (downstream сам читает по result_id) —
  можно вернуть результат за 2-3с, но downstream-обработка станет lazy.
- Бизнесу часто это устраивает (если он в ответ ждёт «готово, забирайте»),
  но иногда — нет (push-кампания должна реально уже отправиться).
- Поэтому числа в SLA нужно атрибутировать к слою.
```

### Ловушки, которые повторяются
- «Compute = N секунд, значит ускорим compute и всё будет N» — неверно, остальное тоже N не станет.
- «Bitmap быстрее SQL» — да, для serving. Но не помогает с export.
- «Поднимем maxConcurrentTasks в TQ» — ускорит compute, но не поможет, если bottleneck — IO к S3.

---

## 6.6 Доска для быстрого ориентира

```text
          time-budget(заказчика)
           |
           v
          [Total lead time]
           = compute + export + delivery (+ overhead)

  | compute | export | delivery |
  |---------+--------+----------|
  |  X      |   Y    |   Z      |  ← измеряем отдельно

  Узкое место одного типа НЕ оптимизируется
  способом из другого типа.
```

---

## 6.7 DoD на этот файл

```text
[ ] Снять реальные числа на 3 разных размерах аудитории (10k / 1M / 10M).
[ ] Записать сюда таблицу X/Y/Z + total.
[ ] Указать дисперсию (p50, p95) — сколько прогонов.
[ ] Сделать вывод: какой слой доминирует в твоих данных.
```

### Личные числа (заполнить самому)
| Размер | compute | export | delivery | total |
|---|---|---|---|---|
| 10k | | | | |
| 1M | | | | |
| 10M | | | | |
