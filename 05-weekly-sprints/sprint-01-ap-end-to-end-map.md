# Sprint 01 — A-P End-to-End Map

## Цель недели
Иметь рабочую end-to-end карту: как запрос на создание аудитории идёт от HTTP до доставки в downstream. Без TQ-внутренностей, на верхнем уровне.

## Почему это важно
Все остальные спринты опираются на эту карту. Без неё «оптимизации» прицеливаются вслепую.

---

## GPT-ускоритель

**Перед началом спринта** → вставь из `gpt-prompts.md` → Sprint 01 → Шаг 2 (Weekly Sprint).
**Если застрял в теме** → вставь из `gpt-prompts.md` → Track 01 → Шаг 3 (Deep Explanation).
**В конце спринта** → вставь из `gpt-prompts.md` → Track 01 → Шаг 4 (Self-check).

---

## День 1 — Repo orientation
**Цель:** разобраться в структуре A-P.

- Прочитать `work_projects/a-p/README.md`, `CLAUDE.md`, `REPO_MAP.md`.
- Просмотреть `internal/` верхнеуровневые модули.
- Открыть `docs/` индекс.

**Практика:** запустить локально `init_dev_environment.sh`. Поднять зависимости (PG, Kafka).

**Артефакт:** заметка «8 модулей A-P, 1 строкой каждый».

**DoD:** локально стартует A-P, hitting `/health` отвечает 200.

---

## День 2 — Domain model
**Цель:** разобрать модель Auditory.

- `internal/domain/model/auditory.go:16-209` — пройти полностью.
- Сопоставить с публичным API: `internal/api/api.go` (типы AuditoryParams, CreateAuditoryRequest).
- Прочитать миграции `migrations/changelog/2025-08-18-TGT-55889-create-auditory.xml` и т.д.

**Практика:** руками заполнить структуру Auditory с примером (фейковая аудитория «москва, PRO, MCC рестораны»).

**Артефакт:** дополнить `01-context/02-ap-domain-map.md` секцией «личные заметки».

**DoD:** могу за 3 минуты пересказать lifecycle Auditory.

---

## День 3 — HTTP path
**Цель:** проследить путь запроса от HTTP до БД.

- `internal/api/auditory/controller.go` → service → repository → PG.
- Прочитать `internal/delivery/http/validator/auditory_validator.go`.

**Практика:** через curl создать аудиторию (или через Insomnia/Postman). Поймать запись в `auditory` таблице.

**Артефакт:** sequence-diagram (mermaid) одного create-запроса.

**DoD:** запись создалась в БД, lifecycle = DRAFT.

---

## День 4 — Loading request + cron
**Цель:** разобрать «start gathering» и crontab.

- `internal/repository/load_auditory_request_repository.go`.
- `internal/application/cron.go` — все cron-джобы.
- `docs/components/job/LoadAuditoryRequestProcessingJob.mdx`.

**Практика:** сделать POST `/auditory/{id}/loading`. Поймать запись в `load_auditory_request`. Проследить, как cron-джоба её подхватит.

**Артефакт:** добавить в sequence-diagram последовательность от loading_request до старта Target.

**DoD:** видел в логах вызов Target REST.

---

## День 5 — Kafka consumer (return path)
**Цель:** разобрать `nfs-partner-platforms.internal.auditory` consumer.

- `internal/delivery/kafka/target/auditory/consumer.go`.
- Avro-схема `contract/avro/auditory-export.avsc`.
- `docs/components/kafka/nfs-partner-platforms.internal.auditory.mdx` — диаграмма.

**Практика:** написать вручную тестовое Avro-сообщение, отправить в Kafka, проверить, что A-P его обработает (возможно через test-container).

**Артефакт:** finalize sequence-diagram (от HTTP create до записи в `t-segment.offline-upload.upload`).

**DoD:** на бумаге могу нарисовать всю цепочку без подсказки.

---

## Итог недели

```text
- Карта A-P end-to-end (1 страница mermaid + 1 страница пояснений).
- Раздел "Личные наблюдения" в 01-context/02-ap-domain-map.md.
- Список 5 неясных мест → пополнить 01-context/07-open-questions-for-team.md.
```

## Финальный артефакт
`practice/sprint-01-ap-end-to-end-map.md` — твоя версия карты с реальными ссылками на код (file:line), которую можешь показать тимлиду.

## Self-check
1. Где именно меняется status DRAFT → GATHERING?
2. Кто и когда проставляет gathering_id?
3. Как A-P узнаёт, что Target закончил?
4. Где dedup-логика kafka-consumer-а?
5. Что A-P отправляет в `t-segment.offline-upload.upload` — full payload или reference?
