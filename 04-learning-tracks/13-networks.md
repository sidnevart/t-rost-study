# Track 13 — Сети: TCP/IP, HTTP, gRPC, Kafka-protocol

## Зачем учить
A-P и Target — сетевые сервисы. A-P слушает gRPC от Target, пишет в Kafka, тянет/кладёт файлы в S3. Target гонит WebClient-запросы к A-P, потребляет Kafka. Если ты не понимаешь, что происходит на уровне сокетов — ты не можешь:
- диагностировать «почему p99 latency растёт»;
- рассуждать про таймауты и keepalive;
- понимать, где Kafka-запись занимает время;
- проектировать retry-логику без риска storm-а.

«Сеть — это быстро» — второе по популярности заблуждение после «network is reliable».

## Что учить — фундамент

### A. TCP/IP stack — практическая модель
```text
1. Физический уровень и Ethernet — MTU, fragmentation, jumbo frames.
2. IP: routing, TTL, ICMP (ping, traceroute — что измеряешь).
3. TCP:
   a. 3-way handshake: SYN / SYN-ACK / ACK — что занимает время.
   b. 4-way close: FIN / FIN-ACK — TIME_WAIT и почему это важно.
   c. Congestion control: slow start, congestion avoidance, CWND.
   d. Nagle algorithm: зачем, и почему TCP_NODELAY важно для latency-critical.
   e. TCP keepalive vs application keepalive — разница.
   f. Receive buffer / send buffer — влияние на throughput.
   g. TCP states: ESTABLISHED, CLOSE_WAIT, TIME_WAIT — диагностика через ss/netstat.
4. UDP: когда нужен (QUIC, DNS, некоторые метрики).
5. Socket options: SO_REUSEADDR, SO_LINGER, SO_RCVBUF, TCP_KEEPIDLE.
```

### B. TLS
```text
1. TLS handshake: ClientHello / ServerHello / Certificate / Finished.
2. Session resumption: TLS session tickets vs session ID.
3. ALPN (Application-Layer Protocol Negotiation) — как gRPC договаривается с сервером.
4. Certificate chain, OCSP, certificate pinning.
5. mTLS (mutual TLS): используется для internal gRPC.
6. Latency стоимость TLS: дополнительные round trips.
7. TLS 1.2 vs TLS 1.3: 1-RTT vs 0-RTT.
```

### C. HTTP/1.1, HTTP/2, HTTP/3
```text
HTTP/1.1:
1. Persistent connections (Connection: keep-alive).
2. Head-of-line blocking (HOL): почему parallel requests через один connection плохи.
3. Pipelining — почему не взлетело.

HTTP/2:
4. Multiplexing: streams поверх одного TCP.
5. HPACK: header compression, encoder state — stateful.
6. Server push — редко используется в API.
7. Stream priority.
8. Flow control per-stream и per-connection.
9. HOL blocking на уровне TCP остаётся.

HTTP/3 (QUIC):
10. UDP + криптография + flow control сами.
11. Устраняет TCP-level HOL blocking.
12. Когда реально имеет смысл.

Практически для нас:
- WebClient (Reactor Netty) использует HTTP/1.1 или HTTP/2?
- Что происходит при connection pool exhaustion в WebClient?
- Как idle connection timeout влияет на latency первого запроса?
```

### D. DNS
```text
1. Рекурсивный resolver vs authoritative.
2. TTL и DNS caching: почему service discovery через DNS — это не мгновенно.
3. Negative caching (NXDOMAIN TTL).
4. DNS в Kubernetes: ndots, search domains — почему первый запрос медленный.
5. DNS lookup в JVM: InetAddress caching (cacheTtl, negCacheTtl).
   Это критично: по умолчанию JVM кэширует DNS навечно.
6. DNS для service discovery vs dedicated registry (Consul, etcd).
```

### E. gRPC protocol internals
```text
1. gRPC поверх HTTP/2: 1 TCP, много streams.
2. Framing: DATA frame (payload) + HEADERS frame (metadata).
3. Length-prefixed message encoding: [compression flag 1 byte] + [length 4 bytes] + [payload].
4. Protobuf wire format: field tags, varint, length-delimited.
5. Типы streaming: unary / server-streaming / client-streaming / bidirectional.
6. Deadlines vs timeouts: как deadline propagates через цепочку вызовов.
7. Cancellation: RST_STREAM по HTTP/2 — что происходит на сервере.
8. Error model: status codes (0-16), gRPC status vs HTTP status.
9. Retry в gRPC: built-in retry policy (service config), hedging.
10. Connection management: go-away frame, channel-level reconnect.
11. Load balancing в gRPC: client-side vs proxy-side; pick_first vs round_robin vs grpclb.
12. Interceptors/middleware: унарные и streaming.
13. Reflection: grpc_cli, grpcurl — как дебажить.
```

### F. Kafka protocol internals
```text
1. Kafka wire protocol: request/response, API keys и versions.
2. Produce request: acks=0/1/-1 (all), что именно подтверждает брокер.
3. Batch (RecordBatch): magic byte, компрессия, CRC, timestamp.
4. Consumer fetch request: min.bytes, max.wait.ms — как broker-side batching.
5. Offset commit: committed vs latest — при чём crashloop.
6. ISR (In-Sync Replicas): что значит acks=all при ISR из 1 реплики.
7. Leader election на уровне partition: KRaft vs ZooKeeper режим.
8. Consumer group protocol: JoinGroup / SyncGroup / Heartbeat — rebalance trigger.
9. Partition assignment: Range, RoundRobin, StickyAssignor.
10. Backpressure в consumer: pause() / resume() (Kafka Streams / Reactor Kafka).
11. Socket connection per broker: почему много consumer-инстансов = много connections.
12. Compression: lz4, zstd, snappy — когда применять.
```

### G. Latency anatomy
```text
1. Speed of light: 1 мс ≈ 300 км. Москва–Амстердам ≈ 40-50 мс RTT минимум.
2. Propagation delay vs processing delay vs queuing delay vs transmission delay.
3. Network round trips в сложных операциях:
   - Новый TCP + TLS 1.2: 3-4 RTT прежде чем первый байт данных.
   - Новый TCP + TLS 1.3: 1-2 RTT.
   - HTTP/2 поверх готового соединения: 0 RTT на handshake.
4. Connection pool importance: почему warm pool = меньше latency.
5. Bandwidth vs latency: «широкий канал» не помогает маленьким RPCам.
6. Bandwidth-delay product (BDP): TCP buffer должен быть >= BDP.
7. Latency amplification при fan-out: если 1 из 10 запросов медленный.
```

### H. Observability и диагностика сети
```text
1. curl -v / --http2 — смотреть заголовки, TLS negotiation.
2. ss -s / netstat -an — состояние сокетов, TIME_WAIT, CLOSE_WAIT.
3. tcpdump + Wireshark — когда нужно поймать пакеты.
4. grpcurl / grpc_cli — тестировать gRPC endpoint без кода.
5. nc / ncat — проверить connectivity без HTTP.
6. traceroute / mtr — где задержка в сети.
7. eBPF инструменты (bpftrace) — latency trace на уровне syscall.
8. Metrics: connection pool size, connection errors, DNS lookup time, TCP retransmits.
```

## Где это есть в проектах

- A-P, `internal/lib/`: kafka-клиент, s3-клиент, grpc-сервер — entry points для изучения конфигурации.
- Target, `application/target-auditory/..../AuditoryPlatformApi.kt` — WebClient к A-P: таймауты, retry.
- Target, Spring Boot application.yaml — Kafka consumer config (max.poll.records, session.timeout.ms).
- TQ, `TQConfig`: `interval`, `heartbeatInterval` — application-level keep-alive поверх сети.
- A-P, `docs/components/kafka/` — Kafka topic contracts.

## Что почитать

| Источник | Что брать |
|---|---|
| **Computer Networking: A Top-Down Approach** (Kurose & Ross) | гл. 2-4: application layer, transport layer, network layer |
| **TCP/IP Illustrated, Vol. 1** (Stevens) | TCP state machine, congestion control — классика |
| **High Performance Browser Networking** (Ilya Grigorik, hpbn.co) | TCP, TLS, HTTP/1-2-3, WebSocket — бесплатно онлайн |
| **gRPC Up and Running** (Indrasiri & Kuruppu) | protocol, patterns, практика |
| Kafka source / КafKa protocol guide (kafka.apache.org) | wire protocol, fetch internals |
| Brendan Gregg «Network Tools» (brendangregg.com) | диагностика eBPF / perf / tcpdump |
| **QUIC RFC 9000** | если хочешь понять HTTP/3 на уровне протокола |
| TLS 1.3 RFC 8446 | разбор handshake |

## Что сделать руками

```text
1. Запустить A-P локально, сделать gRPC-запрос через grpcurl:
   - Посмотреть через Wireshark/tcpdump, что идёт по сети.
   - Найти HTTP/2 HEADERS и DATA frames.
   - Убедиться, что видишь Protobuf payload.

2. Локально: 2 producer + 1 consumer в Kafka.
   - Изменить acks: 0 → 1 → all, замерить latency produce.
   - Изменить max.poll.records: 1 → 500 → 5000, замерить throughput.
   - Симулировать rebalance: убить одного consumer в группе.
   - Наблюдать в метриках: consumer lag.

3. WebClient (Target → A-P):
   - Найти конфигурацию connection pool (maxConnections, pendingAcquireTimeout).
   - Написать нагрузочный тест: 100 concurrent запросов при pool size = 10.
   - Наблюдать, что происходит: latency, error rate.
   - Вопрос: что вернёт WebClient при pool exhaustion?

4. DNS в JVM:
   - Запустить Java-процесс, сделать DNS lookup, добавить запись, сделать снова.
   - Убедиться в кешировании. Проверить -Dnetworkaddress.cache.ttl.
   - Поставь вопрос: как наши сервисы переживают Rolling restart?

5. TCP keepalive:
   - В A-P или Target найти настройки keepalive на Kafka и gRPC клиентах.
   - Узнать: какой у нас idle timeout?
   - Что происходит, если клиент удерживает соединение 5 минут без трафика?

6. TLS handshake:
   - curl -vv https://... (любой внутренний endpoint) — разобрать вывод.
   - Найти: какая версия TLS, какой cipher suite, сколько round trips.
```

## Self-check

```text
1. Почему 3-way handshake + TLS 1.2 = 3 RTT до первого байта? Нарисуй sequence.
2. Что такое TIME_WAIT и почему при большом числе коннектов он накапливается?
3. Зачем TCP_NODELAY в gRPC/Redis клиентах?
4. Что означает acks=-1 (all) в Kafka и когда оно не даёт гарантию?
5. Как rebalance Kafka consumer group влияет на сервис? Каков latency-spike?
6. Почему в JVM DNS resolver нужно явно настраивать TTL?
7. Что происходит при pool exhaustion в WebClient — ошибка сразу или блокировка?
8. Зачем у gRPC deadlines — чем они лучше connect timeout + read timeout?
9. Чем HTTP/2 лучше HTTP/1.1 для A-P→Target fanout запросов?
10. Что такое ISR=1 и почему это нарушает смысл acks=all?
```

## DoD

```text
[ ] Я видел gRPC в Wireshark — знаю, что внутри фрейма.
[ ] Я замерял latency Kafka produce при разных acks.
[ ] Я понимаю, почему DNS TTL важен и что с ним делать в JVM.
[ ] Я могу по ss / netstat прочитать состояние сокетов.
[ ] Я могу объяснить, что происходит при connection pool exhaustion.
[ ] Я понимаю, почему gRPC deadline != timeout.
```
