# Track 11 — Go internals + JVM/Go performance profiling

## Зачем учить
A-P — Go. Target — JVM/Kotlin. Разные рантаймы, разные ловушки. Когда ты говоришь «давайте поднимем concurrency с 5 до 50» — ты должен понимать, что произойдёт в каждой из сред: heap pressure, scheduling, GC, lock contention. Без этого рекомендации — гадание.

Кроме того, «оно медленнее в проде, но я не знаю почему» — это не состояние senior-инженера. Профилирование — стандартный рабочий инструмент.

## Что учить — фундамент

### A. Go internals
```text
1. Goroutines — не OS threads.
   - G (goroutine), M (OS thread), P (processor): G-M-P scheduler.
   - GOMAXPROCS: сколько P одновременно, связь с числом ядер.
   - Stack goroutine: начинается с 2-8 KB, растёт (split stack → contiguous stack).
   - Goroutine lifecycle: runnable → running → waiting.
   - Starvation: что происходит, если горутин больше, чем P, и они CPU-bound.

2. Go memory model.
   - Happens-before: sync.Mutex, channel operations, sync.Once.
   - sync/atomic: что гарантирует, что нет (нет total order без вмешательства).
   - Race detector (-race): как работает, почему медленнее в 2-10x.
   - Общая ошибка: closure захватывает loop variable — что происходит.

3. Go GC.
   - Tri-color mark-and-sweep: concurrent, STW только в короткие паузы.
   - GOGC: trade-off heap size vs GC frequency.
   - GOMEMLIMIT (Go 1.19+): soft memory limit.
   - Write barrier: что это, почему нужен при concurrent mark.
   - Allocation rate vs heap size: основные two levers.
   - Large allocations: объекты > 32 KB прямо в heap (не mcache).
   - sync.Pool: уменьшить alloc rate для temporary objects.
   - Finalizers: почему их почти никогда не нужно использовать.

4. Escape analysis.
   - Когда значение уходит на heap (escapes): pointer return, interface boxing, closures.
   - go build -gcflags="-m" — как читать вывод.
   - Allocations в hot path: почему interface{} / any boxing — скрытый alloc.
   - Value types vs pointer types: что выбрать для hot path.

5. Concurrency primitives Go.
   - sync.Mutex vs sync.RWMutex: когда RW выгоден (много readers, редкие writers).
   - sync.WaitGroup, sync.Once, sync.Cond.
   - Channels: buffered vs unbuffered; select; default.
   - context.Context: дерево контекстов, cancellation propagation, deadline.
   - Типичные goroutine leaks: goroutine без exit condition.

6. Go profiling.
   - pprof: CPU профиль (sampling), heap профиль (allocs + inuse), goroutine dump,
     mutex профиль, block профиль.
   - net/http/pprof: включить endpoint, снять с curl.
   - go tool pprof: top, list, web (flame graph через -http).
   - benchmark с -cpuprofile, -memprofile.
   - trace: go tool trace — goroutine scheduling, GC pauses, network waits.
   - Differential profile: сравнить два снимка, найти regression.
```

### B. JVM internals (для Kotlin/Target)
Этот раздел дополняет Track 03 — здесь акцент на performance, а не на Coroutines.

```text
1. JVM memory model (JMM).
   - happens-before: volatile, synchronized, final, java.util.concurrent.
   - Visibility проблема: без синхронизации compiler/CPU перекладывает операции.
   - Double-checked locking: почему небезопасен без volatile.

2. JIT компилятор.
   - Interpreted → C1 (client) → C2 (server) compilation.
   - Deoptimization: когда JIT отменяет компиляцию (class loading, branch misprediction).
   - Inlining: метод должен быть <= bytecodeLimit для инлайна.
   - Loop unrolling, vectorization (SIMD через JIT).
   - JVM flags: -XX:+PrintCompilation, -XX:+UnlockDiagnosticVMOptions.
   - OSR (On-Stack Replacement): горячий loop компилируется прямо в процессе.

3. Garbage collectors.
   - GC basics: allocation, minor GC (young gen), major GC (old gen), full GC.
   - G1: concurrent mark, mixed GC, remembered sets.
   - ZGC: почти полностью concurrent, sub-millisecond pauses — как это работает
     (load barriers, colored pointers).
   - Shenandoah: похож на ZGC, но другая реализация barriers.
   - GC tuning: -Xmx / -Xms, -XX:MaxGCPauseMillis, -XX:G1HeapRegionSize.
   - Allocation pressure: чем быстрее создаём → тем чаще minor GC.
   - Object lifecycle: short-lived (stay young gen) vs long-lived (promoted to old).
   - GC logs: -Xlog:gc* — как читать.

4. JVM profiling tools.
   - JFR (Java Flight Recorder): низкий overhead, production-safe.
     Методы: CPU sampling, allocation profiling, lock contention, thread states, GC events.
   - async-profiler: стэки без safepoint bias (проблема JFR-based sampling).
     Режимы: -e cpu (CPU), -e alloc (аллокации), -e lock (lock contention).
   - JDK Mission Control: GUI для JFR recordings.
   - jstack: thread dump — найти deadlock, long waits.
   - jmap: heap histogram, dump — для анализа утечек памяти.
   - jcmd: швейцарский нож JVM diagnostics.
   - Safepoint bias: почему JFR CPU sampling иногда искажает картину.
   
5. Lock contention и thread pool.
   - Thread pool sizing: CPU-bound → N+1; IO-bound → экспериментально.
   - ThreadPoolExecutor: corePoolSize, maximumPoolSize, workQueue — как они взаимодействуют.
   - Virtual threads (Project Loom, JDK 21+): что это, когда замена Coroutines.
   - Lock contention в profiler: как выглядит, как устранить.
   - CAS (Compare-And-Swap) и false sharing в cache lines.
```

### C. Общие принципы производительности
```text
1. Измерять прежде чем оптимизировать.
   - «Я думаю, что медленно» ≠ «я знаю, что медленно».
   - Baseline: задокументировать текущие числа до любых изменений.
   - Принцип «изменяй одно за раз».

2. USE method (Brendan Gregg).
   - Для каждого ресурса: Utilization + Saturation + Errors.
   - Ресурсы: CPU, memory, disk I/O, network I/O, connection pool, thread pool.

3. Латентность vs Throughput.
   - Optimize for latency: уменьшай критический путь.
   - Optimize for throughput: увеличивай параллелизм, batch-и.
   - Little's Law: L = λ × W (число в системе = arrival rate × mean time in system).

4. Microbenchmarks — частые ошибки.
   - JVM warmup: без прогрева JIT не выйдет на пик.
   - JMH (Java Microbenchmark Harness): правильный способ бенчмаркинга JVM.
   - testing.B в Go: правильный способ для Go.
   - Dead code elimination: JIT / Go compiler может выкинуть benchmark-тело.
   - Coordinated omission: measure only latency of what you sent, miss queued time.
   - Что memoize: p50, p95, p99, stddev — не только среднее.

5. Allocation и GC pressure.
   - Hot path allocation: что создаётся в каждом request/task.
   - Object pool для дорогих объектов (osync.Pool в Go, Pool в Java).
   - String interning / byte slices vs strings в Go.
   - Escape analysis как инструмент Zero-Allocation критического пути.
```

## Где это есть в проектах

- A-P, `internal/lib/`: kafka-клиент — посмотреть goroutine модель, буферы.
- A-P, `internal/service/s3/`: chunk upload — allocation в hot path?
- Target, TQConfig `Dispatchers.IO + SupervisorJob` — threading model (Track 03).
- TQ, `TQJdbiTaskDao.kt` — JDBI запросы, connection pool contention?
- A-P, Go binary: включить pprof endpoint при локальном запуске.
- Target: JFR в Spring Boot (2.x, через actuator или -XX:StartFlightRecording).

## Что почитать

| Источник | Что брать |
|---|---|
| **The Go Programming Language** (Donovan & Kernighan) | гл. 8-9: goroutines, concurrency |
| **Concurrency in Go** (Cox-Buday) | практика: pipelines, fan-in/out, patterns |
| **100 Go Mistakes** (Harsanyi) | ловушки: goroutine leaks, slice aliasing, interface overhead |
| Aleksey Shipilëv blog (shipilev.net) | JIT, JMM, allocation — самый глубокий материал по JVM |
| **Java Performance** (Scott Oaks) | GC, JIT tuning, microbenchmarks |
| **Systems Performance** (Brendan Gregg) | USE method, perf tools, flame graphs |
| async-profiler docs (github.com/async-profiler) | как пользоваться без safepoint bias |
| Dave Cheney blog (dave.cheney.net) | Go internals — escape analysis, performance |
| **Go GC guide** (go.dev/doc/gc-guide) | официальный туториал по GC tuning |

## Что сделать руками

```text
1. Включить pprof в A-P (net/http/pprof) при локальном запуске.
   - Прогнать синтетическую нагрузку (100 задач).
   - Снять cpu.pprof, heap.pprof.
   - Открыть flame graph: go tool pprof -http=:9090 cpu.pprof.
   - Найти top-3 функции по CPU и по аллокациям.

2. Запустить A-P с -race детектором.
   - Прогнать тесты.
   - Зафиксировать: есть ли race conditions?

3. go build -gcflags="-m" на 1-2 hot-path функциях в A-P.
   - Найти: что уходит на heap (escapes), что остаётся на стэке.
   - Вопрос: можно ли убрать хотя бы 1 escape allocation?

4. Написать benchmark (testing.B) для одной функции в A-P.
   - Запустить с -benchmem.
   - Записать: ns/op, B/op, allocs/op.
   - Изменить одну вещь, замерить снова — видишь разницу?

5. Target: запустить с JFR или async-profiler при исполнении TQ-задачи.
   - Найти top-3 методов по CPU, top-3 по аллокациям.
   - Найти: есть ли lock contention?
   - Если есть — что конкурирует за один lock?

6. Go trace: go tool trace.
   - Прогнать маленькую синтетическую нагрузку с trace.
   - Посмотреть: goroutine scheduling events, GC pauses, network wait.
```

## Self-check

```text
1. Сколько OS threads создаёт Go для 10000 goroutine? Почему.
2. Что произойдёт с GC в Go, если allocation rate вдруг вырос в 10x?
3. Escape analysis: почему `return &Foo{...}` заставляет Foo уйти на heap?
4. В JVM: почему JFR CPU sampling может давать неточный профиль (safepoint bias)?
5. Чем async-profiler устраняет safepoint bias?
6. Что такое coordinated omission и почему ваш benchmark latency заниженный?
7. Little's Law: throughput = 100 rps, mean latency = 50ms — сколько запросов в flight?
8. Когда sync.Pool в Go НЕ помогает?
9. Что произойдёт при connection pool exhaustion в JDBI под нагрузкой?
10. Почему измерять только p50 — ошибка в latency-critical системе?
```

## DoD

```text
[ ] Профиль A-P (cpu flame graph + heap) снят и интерпретирован.
[ ] Профиль Target (JFR или async-profiler) снят и интерпретирован.
[ ] -race детектор на A-P-тестах запущен.
[ ] Escape analysis на 1-2 Go-функциях прочитан.
[ ] Benchmark написан, -benchmem замерен.
[ ] Я могу объяснить G-M-P scheduler и когда goroutine не запускается.
[ ] Я понимаю разницу между allocation rate и heap size как lever для GC.
```
