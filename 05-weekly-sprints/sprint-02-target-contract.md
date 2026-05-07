# Sprint 02 — Target Contract

## Цель недели
Точно знать, что A-P отдаёт Target и что Target отдаёт A-P. На уровне Avro-полей, REST-эндпоинтов и инвариантов. Без угадываний.

## Почему это важно
Контракт — место, где ломаются все архитектурные изменения. Без понимания, что можно расширять, а что трогать нельзя — RFC обречён.

---

## GPT-ускоритель

**Перед началом спринта** → вставь из `gpt-prompts.md` → Track 02 → Шаг 1 (Learning Curator).
**Если застрял на Avro** → вставь из `gpt-prompts.md` → Track 02 → Шаг 3 (Deep Explanation).
**В конце спринта** → вставь из `gpt-prompts.md` → Track 02 → Шаг 4 (Self-check).

---

## День 1 — REST direction A-P → Target
**Цель:** разобрать вызовы A-P → Target.

- `internal/client/target/apiclient.gen.go` — сгенерированный клиент.
- `internal/client/target_http_client.go` — обёртка.
- В Target: `target-auditory-core/.../entrypoint/rest/TargetAuditoryControllerV2.kt`.

**Практика:** проследить один вызов от A-P до Target endpoint в логах.

**Артефакт:** список всех endpoint-ов Target, которые дёргает A-P, с краткой ролью.

**DoD:** знаю все REST-направления A-P → Target.

---

## День 2 — REST direction Target → A-P (JWT)
**Цель:** разобрать вызовы Target → A-P.

- Target: `target-auditory-core/.../service/api/AuditoryPlatformApi.kt` (WebClient + JWT HS512).
- A-P: middleware валидации JWT (поиск по коду).

**Практика:** на тесте сгенерировать невалидный JWT и убедиться, что A-P отказывает с правильным статусом.

**Артефакт:** заметка «JWT contract: claims, exp, secret distribution».

**DoD:** понимаю security-границу между сервисами.

---

## День 3 — Avro `auditory-export.avsc`
**Цель:** разобрать формат main-сообщения возврата.

- A-P: `contract/avro/auditory-export.avsc`.
- Target: `contract/avro/auditory-export.avsc` (должен совпадать).
- A-P consumer: `internal/delivery/kafka/target/auditory/consumer.go`.

**Практика:** проверить совместимость BACKWARD (avro tools / schema registry CLI).

**Артефакт:** схема + список «можно расширять / нельзя ломать».

**DoD:** понимаю, как добавить новое поле без breaking change.

---

## День 4 — Дополнительные topic-и
**Цель:** разобрать остальные kafka-топики между A-P и Target.

- `nfs-partner-platforms.auditory` — что внутри?
- `nfs-partner-platforms.auditory.event` — events stream.
- `nfs-partner-platforms.auditory.unknown-client-availability`.

**Практика:** найти producer и consumer каждого. Понять, кто что пишет.

**Артефакт:** табличка `topic | producer | consumer | format | semantics`.

**DoD:** для каждого топика знаю «кто и зачем».

---

## День 5 — Inv ariants и edge-cases
**Цель:** проверить инварианты контракта.

- Idempotency: где dedup, по какому ключу.
- Ordering: гарантирован ли порядок сообщений в одной аудитории.
- Error semantics: error != null → FAILED, есть ли retry?
- Timeout: как долго A-P ждёт Target.

**Практика:** написать 5 минимальных «провокационных» сценариев (повтор сообщения, out-of-order, error, timeout) и трассировать поведение по коду.

**Артефакт:** заполнить `02-architecture/02-target-contract.md` секциями «можно расширять» / «нельзя ломать» / «открытые вопросы».

**DoD:** список 3-5 вопросов, на которые код ответ не даёт — фиксируем как Open.

---

## Итог недели

```text
- Полная карта контракта (REST + Kafka + Avro + JWT).
- Список инвариантов.
- Список 3-5 нерешённых вопросов в 01-context/07-open-questions-for-team.md.
```

## Финальный артефакт
`practice/sprint-02-target-contract.md` — оверкилл-полная карта контракта + 1 страница «что мы знаем точно, что — гипотеза».

## Self-check
1. Что произойдёт, если Target пошлёт в Kafka одно и то же `gathering_id` дважды?
2. JWT exp у Target → A-P = 1ч. Что если запрос дольше 1ч?
3. Что в Avro `auditory-export.avsc` — обязательно, что — опционально?
4. Как добавить новое поле без breaking change?
5. Кто гарантирует exactly-once в `nfs-partner-platforms.internal.auditory`?
