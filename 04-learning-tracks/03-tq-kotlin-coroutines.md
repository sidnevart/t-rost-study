# Track 03 — Kotlin / JVM internals + Coroutines

## Зачем учить
Target написан на Kotlin/Spring, TQ — Kotlin-библиотека на Coroutines. Чтобы СВОИМИ глазами видеть, что происходит в коде, нужно понимать JVM (memory model, GC, JIT), Kotlin (как компилируется в bytecode), и Coroutines (structured concurrency).

Без этого ты будешь подсматривать у других. С этим — сможешь сам читать чужой Kotlin-код, профилировать его и оценивать предложения по threading.

## Что учить — фундамент

### A. JVM
```text
1. Bytecode и JIT (C1, C2, Graal)
   - что компилируется, когда, как deopt-ится
2. Memory model (JMM)
   - happens-before, volatile, final
   - synchronized, java.util.concurrent atomics
3. Garbage collectors
   - G1, ZGC, Shenandoah — какой у нас, что они оптимизируют
   - что такое pause-time goal, allocation rate, fragmentation
4. Memory layout
   - heap, metaspace, native memory
   - off-heap (ByteBuffer.allocateDirect)
5. Threads и thread-pool
   - native thread vs virtual thread (Project Loom)
   - ForkJoinPool, Executors.*
6. Profiling tools
   - JFR (Flight Recorder) + JDK Mission Control
   - async-profiler (CPU, alloc, lock)
   - jstack, jmap, jcmd
   - GC-логи (анализ через GCViewer / JITWatch)
```

### B. Kotlin
```text
1. Как Kotlin компилируется в JVM bytecode
   - data classes, sealed classes, inline functions, reified types
   - companion objects vs static
2. Null safety и cost
3. Coroutines — теория
   - structured concurrency
   - CoroutineScope, Job, SupervisorJob
   - Dispatchers (IO, Default, Main, Unconfined, custom)
   - suspend functions: как работает state machine под капотом
   - CancellationException — почему special
   - Flow / SharedFlow / StateFlow
   - Channel
4. Coroutines — практика
   - testing с runTest и virtualTime
   - structured concurrency rules (parent-child cancellation)
   - exception handling: CoroutineExceptionHandler
   - withContext vs async vs launch
5. Spring + Coroutines
   - WebFlux Reactor vs co-routines
   - spring-coroutine integration
```

### C. Конкретно для нас
- `TQConfig.schedulingScope = CoroutineScope(Dispatchers.IO + SupervisorJob())`
  - Почему IO, не Default?
  - Почему SupervisorJob?
  - Что произойдёт при exception в одной из coroutine?
- `TQScheduler.kt` — пройди построчно. Какая часть — pure logic, какая — Coroutines glue?
- `TQApiService.kt`, `TQPersister.kt` — где транзакции, где background work.

## Что почитать

| Источник | Что брать |
|---|---|
| **Kotlin Coroutines: Deep Dive** (Moskała) | вся книга, главы 1-15 |
| **Java Concurrency in Practice** (Goetz) | главы 5, 6, 10, 11 (memory model, executors, lock) |
| Aleksey Shipilëv blog (shipilev.net) | JIT, JMM, allocation, microbench |
| Brian Goetz lectures на YouTube | JVM design, Loom |
| **Project Loom JEP** | virtual threads — что это, когда применять |
| JetBrains Kotlin docs / kotlinlang.org | bytecode generation, advanced types |
| Async profiler docs | как пользоваться |
| Honest profiler (или async-profiler сравнение) | избегаем "safepoint bias" |

## Что сделать руками

```text
1. Mini-TQ-engine на 200-300 строк Kotlin: process + task + dependency, in-memory
   (без Postgres). Свои Coroutines. Не подсматривать в TQ.

2. Прочитать TQScheduler.kt и аннотировать построчно: «здесь suspend point»,
   «здесь cancellation», «здесь race condition risk», «здесь backpressure».

3. Профилировать одну реальную TQ-задачу:
   - снять JFR / async-profiler.
   - найти top 3 hot methods, top 3 allocations.

4. Изучить bytecode своей suspend функции: kotlinc -include-runtime + javap.
   Увидеть, как state machine.

5. Тест с runTest и virtualTime — для асинхронной логики.
```

## Self-check

```text
1. Что произойдёт, если CancellationException пойман и не пропущен дальше?
2. Чем отличаются Dispatchers.IO от Dispatchers.Default? Для какого workload какой?
3. Почему SupervisorJob, а не обычный Job, для root scope шедулера?
4. Что такое safepoint в JVM, и как safepoint bias искажает профили?
5. Как Kotlin реализует suspend под капотом (state machine, Continuation)?
6. Какие GC-настройки сейчас у нас, и почему?
7. Где в TQ есть потенциальный race condition? Как его проверить?
8. Чем virtual thread (Loom) отличается от Coroutine в Kotlin?
9. Что делает `withContext(IO)` — суть, не «переключает на IO-pool»?
10. Какие allocation hot spots в одной TQ-задаче и можно ли их убрать?
```

## DoD

```text
[ ] Mini-TQ-engine работает.
[ ] Я могу читать любой Kotlin/Coroutines-код без шпаргалки.
[ ] У меня есть JFR-профиль одной TQ-задачи + анализ.
[ ] Я могу объяснить SupervisorJob через 3 примера.
[ ] Я понимаю, как Coroutines делают то, что они делают.
```
