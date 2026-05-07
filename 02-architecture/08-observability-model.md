# 08. Observability — рамка, не дашборд

> **Цель файла:** научиться **проектировать** observability в любой системе. Не «вот мой список метрик» — а **критерии**, по которым ты сам решишь, что измерять.

---

## 8.1 Зачем observability существует

```text
1. Знать, что система работает (health).
2. Знать, БЫСТРО, что система НЕ работает (alerting).
3. Понимать ПОЧЕМУ не работает (diagnosis).
4. Видеть тенденции (capacity planning, drift).
5. Отвечать на новые вопросы post-hoc (high-cardinality exploration).
6. Audit и compliance.
```

Если ты строишь observability только под (1) и (2) — этого мало для senior-уровня. Цель (3) и (5) — главные.

---

## 8.2 Три кита

```text
Metrics    — числа во времени, агрегаты (counter, gauge, histogram)
Logs       — события с контекстом, structured
Traces     — связанная цепочка spans через сервисы
```

Каждый отдельно полезен, но **по-настоящему** наблюдаемая система = все три **связаны**:
- trace_id в логе = тот же trace в trace storage.
- метрика exemplar = ссылка на trace.
- лог event = метрика count + dimensions.

OpenTelemetry — стандарт, который пытается это унифицировать.

---

## 8.3 Cardinality — главный инвариант

```text
Cardinality = число уникальных комбинаций label values.

Низкая cardinality (status, region) → metrics OK.
Средняя (user_id, audience_id) → traces OK, metrics дорого.
Высокая (request_id, x-trace-id) → logs/traces только.
```

Ошибки cardinality убивают мониторинг:
- Метрика с user_id label → Prometheus умирает.
- Метрика без partition label → можно только видеть «всё в куче».

Каждое поле, которое ты добавляешь в metric, спроси: какова его cardinality? Будет ли расти?

---

## 8.4 SLI / SLO / SLA — иерархия

```text
SLI (Service Level Indicator)   — то, что меряешь (request_success_rate)
SLO (Service Level Objective)   — целевое значение SLI (99.5%)
SLA (Service Level Agreement)   — контракт с потребителем (с штрафами)
```

В большой компании SLA публичны, SLO — внутренние, SLI — технические. Они **не** должны совпадать. SLO обычно строже SLA (запас на ошибки).

Senior-engineer должен уметь:
- предложить правильный SLI (что меряем, чтобы это означало здоровье),
- обосновать SLO (почему именно 99.5%, не 99.9% и не 99%),
- объяснить error budget (сколько нарушений допустимо за период).

---

## 8.5 RED / USE / Golden Signals

Три популярные «рамки» — что меряем для разных типов компонентов:

```text
RED (request-driven services):
  Rate     — RPS
  Errors   — fail rate
  Duration — latency p95/p99

USE (resources):
  Utilization — % busy
  Saturation  — queue depth
  Errors      — error rate

Golden Signals (Google SRE):
  Latency
  Traffic
  Errors
  Saturation
```

Эти рамки пересекаются. Подбирай по природе компонента:
- HTTP-сервис → RED
- Worker pool / queue / диск → USE
- Combined system → Golden Signals (best of both)

---

## 8.6 Correlation — без неё всё разваливается

Когда у тебя 5 сервисов и проблема — в одном из них:

```text
- Без correlation: каждый сервис — отдельный лог. Ты сидишь и угадываешь.
- С correlation: один trace_id сквозит через все 5. ОДИН query = вся цепочка.
```

Реализация в нашем стеке:
- A-P (Go): `context.Context` + middleware.
- Target (Kotlin): `MDC` + Spring filter.
- Через Kafka: header `traceId` в record.
- Через TQ: дополнительное поле в `tq_task.params` (JSONB).

Если correlation **отсутствует** — ты не сможешь debug-ить системную проблему в разумное время.

---

## 8.7 Observability vs monitoring

Это разные понятия:

```text
Monitoring:  заранее знаешь, что измерять. Дашборды, алерты на known-unknowns.
Observability: можешь ответить на новый вопрос, который не предвидел.
```

Observability требует:
- high-cardinality data (не просто метрики),
- query language над raw events,
- ability to slice by ANY dimension post-hoc.

Charity Majors писала об этом много — её посты обязательны.

---

## 8.8 Что изучить

| Тема | Что даёт | Источник |
|---|---|---|
| **Google SRE book** | SLI/SLO/SLA, error budget, alerting philosophy | sre.google/books (бесплатно) |
| **Charity Majors blog** | observability vs monitoring, cardinality | honeycomb.io blog |
| Brendan Gregg «Systems Performance» | USE method | book |
| OpenTelemetry spec | стандартный язык metrics/logs/traces | opentelemetry.io |
| Prometheus / Grafana | metrics в нашем стеке | их docs |
| Distributed tracing | Jaeger/Tempo/Honeycomb внутри | docs |
| Structured logging | формирование логов так, чтобы их можно было query | блоги Honeycomb / Datadog |

---

## 8.9 Упражнения

```text
УПРАЖНЕНИЕ A. Найди, как сейчас сделана correlation между A-P и Target.
              Если нет — что нужно, чтобы сделать?

УПРАЖНЕНИЕ B. Для 1 ключевого SLI компании по аудиториям — предложи SLI/SLO/SLA.
              Согласуй с PM-ом, что это правильное «зеркало здоровья».

УПРАЖНЕНИЕ C. Возьми 1 «неизвестный» вопрос про систему ("как часто аудитория
              переходит из GATHERING в ERROR?") — попробуй ответить ТОЛЬКО через
              текущие метрики/логи. Если не получается — что не хватает?

УПРАЖНЕНИЕ D. Прочти 1 incident post-mortem (свой или из inboxes).
              Что **должно** было быть видно заранее, чтобы предотвратить?
```

---

## DoD

```text
[ ] Я понимаю различие RED/USE/Golden Signals и могу применить.
[ ] У меня есть СВОЙ список 5 SLI для пайплайна аудиторий.
[ ] Я знаю, что такое cardinality и не превышу её случайно.
[ ] Я могу предложить план correlation_id, если её сейчас нет.
[ ] Я понимаю разницу monitoring vs observability и могу обосновать выбор инструмента.
```
