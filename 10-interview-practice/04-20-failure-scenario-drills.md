# 20 Incident Triage & Failure Scenario Drills

A comprehensive collection of **20 production incident response drills** designed for Senior & Staff System Design interviews. Each drill presents realistic symptoms, root cause diagnosis, immediate 5-minute mitigation, permanent architectural fixes, observability alerts, and key architectural takeaways.

---

### Drill 01: Ingress API Gateway 504 Gateway Timeout Spike
- **Symptoms & Blast Radius**: 99.8% of inbound public API requests fail with `HTTP 504 Gateway Timeout`. API Gateway container active thread count spikes from 40 to 500 (max capacity). Unrelated endpoints (login, billing, checkout) fail simultaneously.
- **Root Cause**: A slow downstream recommendation service experienced a 5-second database query delay. The API Gateway had a 30-second read timeout and lacked a Circuit Breaker. Incoming requests held gateway threads open waiting for the recommendation service, exhausting the global connection thread pool.
- **Immediate Triage (Stop the Bleeding)**:
  1. Trigger an emergency feature flag / kill-switch in the API Gateway to bypass recommendation widget calls.
  2. Fall back immediately to a static, in-memory list of trending items.
  3. Restart API Gateway pods in rolling batches to flush blocked thread queues.
- **Permanent Architectural Fix**:
  - Implement **Resilience4j Circuit Breaker**: Trip to `OPEN` state if $> 50\%$ of calls take $> 500\text{ms}$, returning cached fallbacks in $< 2\text{ms}$.
  - Apply the **Bulkhead Pattern**: Isolate thread pools per downstream service so slow non-critical widgets cannot starve core checkout threads.
  - Set strict network timeouts ($< 300\text{ms}$) on all inter-service HTTP/gRPC clients.
- **Alert Rule**: Trigger PagerDuty SEV-1 if API Gateway active worker thread pool utilization exceeds $80\%$ for $> 60\text{ seconds}$.

---

### Drill 02: Database Master CPU Hits 100% on Viral Post (Cache Stampede)
- **Symptoms & Blast Radius**: PostgreSQL Primary database CPU spikes from 15% to 100% in 3 seconds. All 1,000 PgBouncer connection slots become saturated. 85,000 checkout and read requests fail with `HTTP 500 Internal Server Error`.
- **Root Cause**: A viral celebrity post's Redis cache key expired with a uniform fixed TTL ($3600\text{s}$). 50,000 concurrent viewers experienced a simultaneous cache miss within 500ms and concurrently queried the primary database for the exact same record (Thundering Herd).
- **Immediate Triage (Stop the Bleeding)**:
  1. Manually warm the viral post key in Redis with a 24-hour TTL via Redis CLI.
  2. Enable aggressive query result caching in PgBouncer / ProxySQL for 60 seconds.
- **Permanent Architectural Fix**:
  - Deploy **Singleflight Mutex Locking**: Ensure only 1 worker thread executes the database query on a cache miss, while the remaining 49,999 requests wait for the mutex and read the newly populated cache.
  - Add **Jittered TTLs**: Randomize cache expiration ($3600\text{s} \pm \text{rand}(0, 300\text{s})$) to prevent synchronized mass expiration.
  - Implement **XFetch Probabilistic Early Expiration**: Background worker asynchronously refreshes the cache before TTL expires based on request frequency.
- **Alert Rule**: Alert if PostgreSQL active query count exceeds $80\%$ of pool size with average query latency $> 500\text{ms}$.

---

### Drill 03: Kafka Consumer Group Rebalance Storm & Processing Halt
- **Symptoms & Blast Radius**: Zero messages processed across an order fulfillment topic for 45 minutes. Consumer lag explodes from 100 to 2,000,000 messages. Kafka broker CPU spikes to 90% due to continuous metadata exchange.
- **Root Cause**: A batch of heavy PDF invoices took 6 minutes to process in a single worker batch. Because `max.poll.interval.ms` was set to the default 5 minutes (300,000ms), Kafka assumed the consumer was dead, triggered a group rebalance, revoked its partitions, and reassigned them to another worker—which also took 6 minutes, triggering an infinite rebalance storm.
- **Immediate Triage (Stop the Bleeding)**:
  1. Temporarily increase `max.poll.interval.ms` to 15 minutes (900,000ms) and decrease `max.poll.records` to 5 in consumer deployment config.
  2. Perform a rolling restart of the consumer worker fleet.
- **Permanent Architectural Fix**:
  - Decouple the **Kafka polling thread** from the **worker compute thread**: The poll thread immediately dispatches records to an internal bounded `ThreadPoolExecutor` and calls `poll()` continuously.
  - Switch partition assignment strategy to `CooperativeStickyAssignor` to avoid stop-the-world full group rebalances.
- **Alert Rule**: Alert if consumer group rebalance frequency exceeds 3 rebalances in a 10-minute window.

---

### Drill 04: Two-Node Cluster Split-Brain & Divergent Ledgers
- **Symptoms & Blast Radius**: Customer complaints regarding balance discrepancies and duplicate order shipments. Audit reveals Node A and Node B both acted as Master for 45 minutes, accepting 12,400 conflicting transactions on Node A and 9,800 on Node B.
- **Root Cause**: A 2-node database cluster experienced a transient inter-datacenter network partition. Lacking a third tie-breaker witness node, a 1-1 split occurred where neither node could determine majority quorum, causing both nodes to self-promote to Master.
- **Immediate Triage (Stop the Bleeding)**:
  1. Immediately revoke write access from Node B (set `read_only = true`).
  2. Lock impacted user accounts to stop ongoing conflicting mutations.
  3. Execute offline WAL log reconciliation scripts to merge split transactions.
- **Permanent Architectural Fix**:
  - Always deploy an **odd number of consensus nodes** ($N = 3, 5, 7$) across independent Availability Zones.
  - Enforce **Strict Majority Quorum ($N/2 + 1$)**: Any isolated partition with $\le N/2$ nodes must immediately demote itself to Read-Only.
  - Implement **STONITH** (Shoot The Other Node In The Head) fencing hardware mechanisms.
- **Alert Rule**: Trigger critical page if active cluster node count drops below majority quorum ($< (N/2 + 1)$).

---

### Drill 05: Client Retry Storm & Self-Inflicted DDoS
- **Symptoms & Blast Radius**: Following a brief 2-second network hiccup, inbound API traffic spikes from 2,000 QPS to 65,000 QPS (3,000% increase), overwhelming API Gateways and keeping backend databases offline for 40 minutes.
- **Root Cause**: Mobile client apps and upstream microservices implemented aggressive retry loops with fixed 100ms intervals and zero jitter. When the backend returned 503 errors, all clients retried simultaneously in lockstep, creating an avalanche of traffic that prevented the database from recovering.
- **Immediate Triage (Stop the Bleeding)**:
  1. Enable aggressive Edge WAF / API Gateway rate limiting to drop 85% of retry traffic.
  2. Return `HTTP 429 Too Many Requests` with `Retry-After: 30` headers.
- **Permanent Architectural Fix**:
  - Enforce **Exponential Backoff with Full Jitter**: $\text{Delay} = \text{rand}(0, \min(\text{MaxBackoff}, \text{BaseBackoff} \times 2^{\text{attempt}}))$.
  - Implement **Global Retry Budgets**: Upstream clients track success/error ratio and reject retries if $> 10\%$ of requests are failing.
  - Require client `Idempotency-Key` headers on all write operations to safely handle retries without duplicate side effects.
- **Alert Rule**: Alert if total request rate exceeds $200\%$ of the 7-day rolling baseline with an error rate $> 10\%$.

---

### Drill 06: Database Hotspotting / Shard Meltdown on Low-Cardinality Keys
- **Symptoms & Blast Radius**: During Black Friday, 75% of checkout requests time out. Monitoring shows Shard Node 1 CPU is at 100% and disk IOPS saturated at 50,000 IOPS, while Shard Nodes 2, 3, and 4 sit completely idle at 1% CPU.
- **Root Cause**: The database was sharded by `order_status` (`'PENDING'`, `'PROCESSING'`, `'SHIPPED'`). Because 98% of Black Friday traffic created new orders in `'PENDING'` state, all write traffic hit Shard Node 1 exclusively.
- **Immediate Triage (Stop the Bleeding)**:
  1. Temporarily buffer new pending order writes into a distributed Redis queue.
  2. Vertically scale Shard Node 1 instance size (scale CPU and IOPS).
- **Permanent Architectural Fix**:
  - Re-shard table using a **High-Cardinality Partition Key** (e.g. `order_id` or `user_id`) using uniform MurmurHash3 distribution.
  - If querying by low-cardinality status is required, use **Key Salting** (append random integer `status_0` to `status_9`) across 10 virtual buckets.
- **Alert Rule**: Alert if CPU or IOPS skew between database shard nodes exceeds $300\%$.

---

### Drill 07: Poison Pill Message Crashing Consumer Pods in CrashLoopBackOff
- **Symptoms & Blast Radius**: All 50 order processing worker pods in Kubernetes crash continuously within 10 seconds of startup (`CrashLoopBackOff`). Order queue backlog accumulates 2M unprocessed orders.
- **Root Cause**: A malformed JSON message with an unexpected null value in a required field was published to the Kafka topic. Every worker that polled the message crashed with a `NullPointerException`, failed to commit its offset, restarted, re-polled the exact same message, and crashed again in an infinite loop.
- **Immediate Triage (Stop the Bleeding)**:
  1. Identify the partition and offset of the poisoned message from worker logs.
  2. Manually advance the consumer group commit offset by 1 using `kafka-consumer-groups.sh --reset-offsets --to-offset <offset+1>`.
- **Permanent Architectural Fix**:
  - Wrap all message deserialization and processing in **Top-Level `try-catch` blocks**.
  - Configure a **Dead-Letter Queue (DLQ)**: If a message fails after 3 retries, route it to a `.DLQ` topic, log the error, and commit the offset to continue processing subsequent messages.
  - Enforce binary schema validation (Avro/Protobuf) at the producer via Confluent Schema Registry.
- **Alert Rule**: Alert if more than 5 messages are routed to the Dead-Letter Queue in a 5-minute window.

---

### Drill 08: Java Stop-The-World GC Pause & Zombie Master Node
- **Symptoms & Blast Radius**: Double-allocation of resources and corrupted distributed state machine records. Audit reveals two nodes executed master coordination commands simultaneously for 15 seconds.
- **Root Cause**: A Java-based coordinator node experienced a 20-second Stop-The-World (STW) Garbage Collection pause due to memory fragmentation. Zookeeper declared the node dead after missing its 10-second heartbeat lease and elected a new leader. When the old coordinator resumed from GC, it executed stale buffered writes unaware that it had been deposed.
- **Immediate Triage (Stop the Bleeding)**:
  1. Revoke the old coordinator's database credentials.
  2. Restart the storage tier and rollback conflicting state updates.
- **Permanent Architectural Fix**:
  - Enforce **Monotonically Increasing Epoch / Fencing Tokens**: The storage tier records the highest seen epoch number and strictly rejects any command with an epoch lower than current.
  - Tune JVM Garbage Collection: Switch from legacy G1GC to **ZGC (Z Garbage Collector)** to guarantee sub-millisecond maximum STW pause times.
- **Alert Rule**: Alert if JVM garbage collection pause time exceeds $500\text{ms}$.

---

### Drill 09: NTP Clock Backward Step & Snowflake ID Collisions
- **Symptoms & Blast Radius**: 15,000 user signup requests fail with database `DuplicateKeyException` on the primary key `id`.
- **Root Cause**: An NTP daemon synchronized physical system time with an upstream time server and executed a hard backward clock step (-500ms). The Snowflake ID generator on that node generated IDs using the backward timestamp, colliding with IDs generated half a second earlier.
- **Immediate Triage (Stop the Bleeding)**:
  1. Divert ID generation traffic away from the offending node.
  2. Reconfigure NTP daemon to use clock slewing (`adjtime`) rather than step jumps.
- **Permanent Architectural Fix**:
  - Implement **Clock Drift Detection in Snowflake**: Maintain `lastTimestamp`. If `currentTimestamp < lastTimestamp`, either sleep until clock catches up (for small drift $\le 5\text{ms}$) or throw an explicit `ClockDriftException` refusing to generate IDs.
- **Alert Rule**: Alert if server physical clock offset from NTP stratum exceeds $\pm 10\text{ms}$.

---

### Drill 10: Read-Your-Own-Writes Inconsistency & User Confusion
- **Symptoms & Blast Radius**: 5,000 customer support tickets submitted within 1 hour reporting that password and profile changes "did not save". Users repeatedly submitted duplicate updates, compounding system load.
- **Root Cause**: Following a profile update (written to PostgreSQL Master), the client web app immediately redirected to the profile page. The subsequent read query was routed to an asynchronous read replica that had a 600ms replication lag, displaying stale profile data.
- **Immediate Triage (Stop the Bleeding)**:
  1. Update API Gateway routing rules to direct all profile page reads to the Primary database for 5 seconds following any write operation.
- **Permanent Architectural Fix**:
  - Implement **Read-Your-Own-Writes Consistency**: Set a session cookie containing `last_write_timestamp`. If `System.currentTimeMillis() - last_write_timestamp < 5000ms`, route reads directly to Master.
  - Implement **Causal Consistency / Replication LSN Tracking**: Client passes the write Log Sequence Number (LSN) in the request header; read replica only serves query once its local replication LSN catches up.
- **Alert Rule**: Alert if database read replica replication lag exceeds $1,000\text{ms}$ ($1\text{s}$).

---

### Drill 11: Distributed Deadlock in Synchronous Two-Phase Commit (2PC)
- **Symptoms & Blast Radius**: 100% of e-commerce checkout transactions freeze. All 500 PostgreSQL database connections are locked in `Waiting for Lock` state, causing total platform collapse.
- **Root Cause**: Two microservices executed distributed 2PC transactions locking resources across Order and Inventory databases in opposite order: Tx1 locked Order $A$ then Inventory $B$; Tx2 locked Inventory $B$ then Order $A$. Both transactions held exclusive locks and waited for each other indefinitely.
- **Immediate Triage (Stop the Bleeding)**:
  1. Run `SELECT pg_terminate_backend(pid)` to kill blocked database transactions.
  2. Disable the synchronous 2PC cross-database coordinator.
- **Permanent Architectural Fix**:
  - Eliminate distributed 2PC; replace with **Event-Driven Saga Pattern** orchestrated by Temporal or Kafka events with compensating transactions.
  - If distributed locking is strictly unavoidable, enforce **Global Strict Resource Ordering** (always lock resource IDs in alphabetical/numerical order).
- **Alert Rule**: Alert if database transactions waiting for lock exceeds 10 concurrent sessions for $> 30\text{ seconds}$.

---

### Drill 12: High-Cardinality Metric Label Explosion Causing Prometheus OOM
- **Symptoms & Blast Radius**: Prometheus TSDB memory usage surges from 16GB to 128GB (100% capacity), triggering Linux kernel OOM killer. Prometheus enters an infinite restart crash loop. All Grafana dashboards and production alert rules go dark.
- **Root Cause**: A developer added `user_id` (UUID) as a Prometheus metric label (`http_requests_total{user_id="<uuid>"}`). With 50M active users, the time-series cardinality exploded from 1,000 to 50,000,000 series, exhausting all TSDB RAM.
- **Immediate Triage (Stop the Bleeding)**:
  1. Roll back the application deployment that introduced the `user_id` label.
  2. Add `metric_relabel_configs` to Prometheus scrape config to drop the `user_id` label immediately.
  3. Wipe Prometheus WAL cache and restart Prometheus.
- **Permanent Architectural Fix**:
  - Strictly enforce **Low-Cardinality Metric Labels** (HTTP method, status code, bounded service name).
  - CI/CD linting rule blocking any metric label matching UUID or email regex.
  - Use Distributed Tracing (Jaeger/Zipkin) or structured logs (ClickHouse) for user-level forensic debugging.
- **Alert Rule**: Alert if Prometheus total active time-series count increases by $> 50\%$ in a 1-hour window.

---

### Drill 13: Global CDN Wildcard Cache Purge (`/*`) & Origin Storage Collapse
- **Symptoms & Blast Radius**: Origin media storage servers experience a 500,000 QPS traffic spike. Origin network egress saturates at 10 Gbps (100% capacity). Video and photo streaming fails for 100% of global users.
- **Root Cause**: An engineer issued an emergency wildcard cache purge (`/*`) on Cloudflare during peak traffic to deploy a CSS fix. 500,000 concurrent global users bypassed edge caches and hit the origin media servers simultaneously.
- **Immediate Triage (Stop the Bleeding)**:
  1. Enable Cloudflare emergency "Under Attack Mode" / Origin rate limiting to drop 90% of requests.
  2. Pre-warm top 1,000 hot media objects via CDN Push API.
- **Permanent Architectural Fix**:
  - Deploy **CDN Origin Shielding**: Origin Shield POPs collapse simultaneous regional cache misses into a single request to origin.
  - Use **Versioned Asset URLs** (`bundle.v2.js`, `image_hash.webp`) rather than global cache purges.
  - Configure `Cache-Control: stale-while-revalidate=86400` headers so edges serve stale content while refreshing.
- **Alert Rule**: Alert if CDN Origin request volume exceeds $300\%$ of normal baseline.

---

### Drill 14: Nested DB Transactions & HikariCP Connection Pool Deadlock
- **Symptoms & Blast Radius**: Application servers throw `ConnectionTimeoutException: Connection is not available, request timed out after 30000ms`. 100% of API requests return HTTP 500.
- **Root Cause**: A Spring Boot service opened an outer `@Transactional` database connection, executed a slow external HTTP API call (5s latency), and then requested a second nested database connection from the same HikariCP pool. Under 50 concurrent requests, all 50 pool connections were held by outer transactions waiting for inner connections, causing total thread deadlock.
- **Immediate Triage (Stop the Bleeding)**:
  1. Restart application containers to clear the deadlocked thread pools.
  2. Temporarily double the HikariCP pool size from 50 to 100 connections.
- **Permanent Architectural Fix**:
  - **Never perform network I/O inside `@Transactional` blocks**: Fetch all external HTTP data first, then execute a narrow, fast ($< 5\text{ms}$) database transaction.
  - Enforce HikariCP pool sizing rule: $\text{Pool Size} = (\text{CPU Cores} \times 2) + \text{Disk Spindles}$.
- **Alert Rule**: Alert if HikariCP active connection pool utilization exceeds $90\%$ for $> 15\text{ seconds}$.

---

### Drill 15: DNS 24-Hour TTL Misconfiguration & Traffic Black Hole
- **Symptoms & Blast Radius**: During an emergency datacenter migration, engineers updated the DNS A-record to point to a new datacenter IP. However, 80% of global users continued hitting the old dead IP address for 24 hours, causing millions in lost revenue.
- **Root Cause**: The DNS A-record had a 24-hour TTL (`86400s`) configured. Global ISP recursive resolvers cached the old dead IP address for 24 hours, ignoring the new IP address until the TTL expired.
- **Immediate Triage (Stop the Bleeding)**:
  1. Stand up a temporary TCP reverse proxy (HAProxy / NGINX) at the old IP address to forward all inbound traffic to the new datacenter IP.
- **Permanent Architectural Fix**:
  - Reduce DNS TTL to **60 seconds** at least 48 hours prior to any scheduled IP migration.
  - Use **Anycast BGP Routing** or Cloud Load Balancers (Route 53 / Cloudflare) where the public IP remains static and traffic is steered internally.
- **Alert Rule**: CI/CD check warning if production DNS A/AAAA records have a TTL $> 300\text{ seconds}$.

---

### Drill 16: Unbounded In-Memory Request Queue & JVM Heap Collapse (OOM)
- **Symptoms & Blast Radius**: Application servers crash with `java.lang.OutOfMemoryError: Java heap space`. Pods restart continuously and fail readiness health probes.
- **Root Cause**: An asynchronous task processor used an unbounded `LinkedBlockingQueue` (`new LinkedBlockingQueue<>()`). A downstream database slowdown caused task generation to outpace processing ($1,000\text{ tasks/s in}$ vs $100\text{ tasks/s out}$), accumulating 5,000,000 task objects in RAM and exhausting the 8GB JVM heap.
- **Immediate Triage (Stop the Bleeding)**:
  1. Restart application containers with rate-limiting filters enabled at the API Gateway.
- **Permanent Architectural Fix**:
  - Always use **Bounded Queues** (`new ArrayBlockingQueue<>(1000)`).
  - Implement a **Rejection Policy** (`ThreadPoolExecutor.CallerRunsPolicy` or `AbortPolicy` with HTTP 429) to apply upstream backpressure when the queue is full.
- **Alert Rule**: Alert if internal thread pool queue size exceeds $80\%$ capacity for $> 30\text{ seconds}$.

---

### Drill 17: Redis Cluster Hot Key Network Bandwidth Saturation
- **Symptoms & Blast Radius**: Redis command latency spikes from $0.5\text{ms}$ to $500\text{ms}$. Single Redis shard network bandwidth hits 10 Gbps (100% NIC saturation). Read requests for other keys on the same shard time out.
- **Root Cause**: A breaking news article was stored under a single Redis key (`news:1001`). 200,000 concurrent users queried that exact key, concentrating all network egress traffic on the single Redis shard hosting that key's hash slot.
- **Immediate Triage (Stop the Bleeding)**:
  1. Deploy an in-memory L1 Caffeine cache inside application instances to cache the hot key for 10 seconds.
- **Permanent Architectural Fix**:
  - Implement **Hot Key Read Replicas & Key Salting**: Replicate hot keys across $N$ shards by appending random suffixes (`news:1001#1`, `news:1001#2`, ... `news:1001#10`) and reading from a randomized shard.
  - Deploy a **Multi-Tier Cache**: L1 In-Process Cache (Caffeine, TTL 5s) + L2 Redis Cluster.
- **Alert Rule**: Alert if any single Redis node network egress exceeds $80\%$ of NIC bandwidth ($8\text{ Gbps}$ on 10 Gbps NIC).

---

### Drill 18: Distributed Rate Limiter Redis Downtime (Fail-Open vs Fail-Close)
- **Symptoms & Blast Radius**: A network partition isolates the Redis rate limiter cluster. API Gateways configured with Fail-Close reject 100% of legitimate user traffic with HTTP 429, turning a cache outage into a total company outage.
- **Root Cause**: The API Gateway rate-limiting filter was configured with a strict "Fail-Close" policy, rejecting all requests whenever Redis could not be reached within 50ms.
- **Immediate Triage (Stop the Bleeding)**:
  1. Toggle rate-limiter fallback configuration from Fail-Close to Fail-Open via dynamic configuration service (Consul/etcd).
- **Permanent Architectural Fix**:
  - Implement a **Fail-Open with Local In-Memory Fallback Policy**: If Redis is unreachable, the gateway logs a warning, falls back to a local in-memory token bucket, and allows traffic to pass to prevent catastrophic outages.
  - Deploy Multi-AZ Redis replication with automated Sentinel failover.
- **Alert Rule**: Alert if Redis rate limiter health check fails for $> 3$ consecutive probes.

---

### Drill 19: SSL/TLS Certificate Expiration on Internal Service Mesh
- **Symptoms & Blast Radius**: 100% of inter-service gRPC calls fail with `SSLHandshakeException: Certificate expired`. All business workflows across 50 microservices fail instantly.
- **Root Cause**: The internal mTLS root CA certificate expired at midnight. Automated cert-manager renewal had failed silently 30 days prior due to an invalid Kubernetes RBAC permission.
- **Immediate Triage (Stop the Bleeding)**:
  1. Manually issue a renewed root certificate using HashiCorp Vault / cert-manager and force-restart Envoy sidecar proxies.
- **Permanent Architectural Fix**:
  - Deploy **Automated Certificate Rotation** (cert-manager / Vault) with automated renewal at 60% of certificate lifespan.
  - Implement **Synthetic Certificate Expiration Monitoring**: Prometheus alerts when any internal or external TLS certificate has $< 30\text{ days}$ remaining.
- **Alert Rule**: Trigger PagerDuty SEV-2 alert if any TLS certificate expires in $< 30\text{ days}$; SEV-1 if $< 7\text{ days}$.

---

### Drill 20: Asymmetric Network Partition (Unidirectional Packet Drop)
- **Symptoms & Blast Radius**: Node A can send heartbeat packets to Node B, but Node B's acknowledgments are dropped by a malfunctioning network switch. Node A assumes Node B is dead and attempts failover, while Node B assumes Node A is healthy, causing continuous cluster oscillation.
- **Root Cause**: Asymmetric network switch failure where ingress packets were dropped while egress packets functioned normally.
- **Immediate Triage (Stop the Bleeding)**:
  1. Manually isolate the malfunctioning node by shutting down its switch port.
- **Permanent Architectural Fix**:
  - Implement **Bidirectional Heartbeat Quorums (SWIM Gossip Protocol)**: Failure detection requires indirect pinging through secondary witness nodes ($C$ checks if $B$ can talk to $A$) before declaring a node dead.
  - Use TCP Keepalive with bounded socket timeouts and bidirectional handshake verification.
- **Alert Rule**: Alert if packet drop asymmetry between cluster nodes exceeds $5\%$ over a 1-minute window.
