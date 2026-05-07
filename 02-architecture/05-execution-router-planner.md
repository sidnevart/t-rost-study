# 05. Capacity & Performance — как реально оценивать

> **Цель файла:** научить тебя **системно** оценивать performance любой системы. Это критичнее, чем уметь выбирать путь — без этого выбор делается интуицией. (Файл переименован концептуально: вместо «вот мой Planner-предложение» здесь то, чему ты должен научиться, чтобы СВОЙ Planner спроектировать.)

---

## 5.1 Что значит «performance»

Это не одно число. Это связанная пятёрка:

```text
Latency       — сколько ждёт один запрос
Throughput    — сколько запросов в единицу времени
Utilization   — насколько занят ресурс (CPU, IO, queue)
Saturation    — есть ли очередь (тот же ресурс, но взгляд от потребителя)
Errors        — что произошло при недоступности / отказе
```

Brendan Gregg назвал это USE method. Помни: оптимизация одной из пяти **обычно меняет другие**, иногда — в худшую сторону. Например, повышение throughput за счёт батчинга увеличивает latency.

---

## 5.2 Закон Little

```text
L = λ × W
  где L — среднее число запросов в системе,
      λ — rate входящих,
      W — среднее время в системе.
```

Это не теория — это **проверка sanity**. Если кто-то говорит «увеличим throughput в 2 раза, latency не поменяется» — Little сразу подсказывает, что без увеличения параллельности это невозможно.

Применяй Little везде. Например: в TQ, при interval=5s и maxConcurrentTasks=5, какой максимальный throughput? Что станет с очередью, если λ превысит этот предел?

---

## 5.3 Где время уходит — иерархия задержек

Когда ты говоришь «X секунд», это сумма:

```text
1. Network round-trip            (LAN ~0.5ms, region ~5ms, cross-region ~80ms)
2. Disk IO                       (SSD random 100us, HDD random 10ms, S3 ~50ms)
3. CPU compute                   (зависит от данных)
4. Lock / serialization waits    (часто скрыто)
5. Queue wait time               (зависит от λ и capacity)
6. GC / runtime pauses           (JVM: ms-seconds, зависит от collector)
7. Coordination overhead         (consensus, broadcast, multi-phase commit)
```

**Если ты не знаешь, в какой из 7 категорий тратятся секунды — у тебя нет права говорить «оптимизируем».**

---

## 5.4 Tail latency — почему p99 важнее, чем кажется

```text
p50 = 50ms       — большинству нормально
p99 = 500ms      — но 1% запросов в 10x хуже
p99.9 = 5s       — 0.1% — катастрофа

В пайплайне из 5 шагов, где каждый p99=500ms,
вероятность того, что запрос «попадёт» хотя бы в один tail =
1 - 0.99^5 ≈ 5%.
То есть p95 всего пайплайна ≥ 500ms.

Tail amplifies through pipelines.
```

Что отсюда:
- При длинном пайплайне tail latency одного звена **доминирует** над average всех.
- Сокращение p50 на 50% может ничего не дать; сокращение p99 на 30% — много.

---

## 5.5 Bottleneck analysis — как находить узкое место

```text
Шаг 1. Замерь end-to-end. Если оно «нормальное» — нет работы.

Шаг 2. Декомпозируй на компоненты с timestamps:
       t_start_A → t_end_A → t_start_B → t_end_B → ...
       Где разрыв между t_end_X и t_start_X+1 — это IO, queue, или scheduling.

Шаг 3. Для каждого компонента — USE:
       - Utilization (CPU%, IO%, ratio занятых coroutines)
       - Saturation (queue depth)
       - Errors (rate)

Шаг 4. Самый высокий Saturation = bottleneck. Часто это НЕ самый медленный
       компонент в isolation; bottleneck — это компонент с самой длинной
       очередью при текущей нагрузке.

Шаг 5. Освободишь bottleneck — bottleneck сместится в другое место.
       Это нормально. Performance — итеративная задача.
```

---

## 5.6 Микробенчмарки и их враги

Микробенчмарк — это попытка измерить кусок системы изолированно. Это полезно и **очень опасно**:

```text
Враги микробенчмарка:
1. JIT warmup       — JVM первые тысячи итераций медленные
2. Cache state      — CH MARK CACHE, OS page cache, JIT inline cache
3. GC timing        — большой GC попал в один из прогонов и исказил
4. Thermal scaling  — CPU замедлился из-за температуры
5. Power state      — laptop CPU throttle
6. Noisy neighbours — другие процессы / контейнеры
7. Optimization     — компилятор/JIT убрал твой code как dead
8. Realistic data   — синтетический dataset не отражает реальность
```

Правила:
- Минимум 3 прогона, лучше 10. Считай p50/p95 + stddev.
- Drop первых N итераций (warmup).
- Контроль cache state (что прогрел, что нет).
- Сравнивай в одинаковых условиях (день недели, час, нагрузка кластера).

---

## 5.7 Capacity planning vs performance tuning

Это разные задачи:

```text
Performance tuning:
  "У меня запрос медленный, как ускорить?"
  Метод: профилирование, оптимизация alg, индексы, переписать query.

Capacity planning:
  "У меня нагрузка вырастет в 10x, что сломается первым?"
  Метод: модель ресурсов, USE method, headroom, growth projection.
```

Senior-engineer должен уметь оба. Они часто противоречат друг другу: aggressive tuning под текущую нагрузку увеличивает чувствительность к её изменению.

---

## 5.8 Что изучить

| Тема | Что даёт | Источник |
|---|---|---|
| **USE method** | системный подход к диагнозу | brendangregg.com |
| Queueing theory | M/M/c, Little, утилизация vs latency | Harchol-Balter book / её слайды Carnegie Mellon |
| Profiling tools | perf, async-profiler, pprof, JFR | Brendan Gregg's «Systems Performance» |
| Latency numbers (Norvig) | порядки задержек CPU/RAM/disk/network | famous «Latency Numbers Every Programmer Should Know» |
| Microbenchmark traps | JMH (Java), testing.B (Go), Criterion (Rust) | их доки + Aleksey Shipilëv blog (для JVM) |
| Tail latency | google «The Tail at Scale» Dean & Barroso | их paper, ~10 страниц |
| Capacity planning | Production-Ready Microservices + SRE book | books |
| Storage engine perf | Database Internals, Petrov | book |

---

## 5.9 Упражнения

```text
УПРАЖНЕНИЕ A. Замерь latency 1 операции в трёх режимах: cold cache, warm cache, peak load.
              Запиши p50/p95/stddev. Выведи 1 наблюдение.

УПРАЖНЕНИЕ B. Применить USE method к 1 узлу системы (например, JVM-нода Target).
              Найди, какой из 3 (U/S/E) ближе всего к limit-у.

УПРАЖНЕНИЕ C. Модель Little для TQ: при текущих TQConfig defaults — какой максимум throughput?
              Какова latency, если λ = 80% capacity?

УПРАЖНЕНИЕ D. На EXPLAIN ANALYZE одного реального запроса — найди самый дорогой узел плана.
              Опиши 3 разных способа его удешевить.

УПРАЖНЕНИЕ E. Прочти «Tail at Scale» (Dean, Barroso). Найди в нашей системе 1 компонент,
              который страдает от tail amplification.
```

---

## DoD

```text
[ ] Я понимаю Little's law и могу применить к нашему пайплайну.
[ ] У меня есть USE-таблица для 1 узла системы.
[ ] Я провёл 1 профилировку с pprof или async-profiler.
[ ] У меня есть числа latency p50/p95 для 1 реальной операции.
[ ] Я понимаю, в чём разница между capacity planning и performance tuning, и применил оба.
```
