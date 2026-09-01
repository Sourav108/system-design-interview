# Module 08: Production Outage Post-Mortems & Failure Modes

A curated collection of **15 production-grade distributed system failure post-mortems**, analyzing real-world architectural failure modes, root causes, detection metrics, immediate mitigations, and permanent code/infrastructure fixes.

---

## 💥 Catalog of 15 Production Outage Post-Mortems

| ID | Incident Title | Failure Category | Root Cause & Core Defense |
|---|---|---|---|
| [**01**](./01-cascading-failure-circuit-breaker.md) | **Cascading Failure & Thread Pool Saturation** | Resilience | Slow downstream service exhausted gateway threads $\to$ Resilience4j Circuit Breakers |
| [**02**](./02-thundering-herd-cache-stampede.md) | **Thundering Herd & Cache Stampede** | Caching | Synchronized key expiration melted database $\to$ Singleflight Mutex & Jittered TTLs |
| [**03**](./03-split-brain-network-partition.md) | **Split-Brain in Two-Node Cluster** | Consensus | 2-node cluster split caused dual masters $\to$ Odd-node Quorum ($N/2 + 1$) & STONITH |
| [**04**](./04-retry-storm-exponential-backoff.md) | **Retry Storm & Self-Inflicted DDoS** | Networking | Aggressive zero-jitter retries $\to$ Full Jitter Exponential Backoff & Retry Budgets |
| [**05**](./05-database-hotspot-hash-partitioning.md) | **Database Hotspot from Low-Cardinality Keys** | Partitioning | Sharding by `status` melted pending node $\to$ MurmurHash3 & Key Salting |
| [**06**](./06-message-queue-poison-pill-dlq.md) | **Poison Pill Message & Infinite Crash Loop** | Messaging | Unparseable JSON crashed consumer fleet $\to$ Dead-Letter Queues (DLQ) |
| [**07**](./07-memory-leak-gc-pause-zombie-nodes.md) | **GC Pause & Zombie Master Node** | Coordination | 20s GC pause caused stale writes $\to$ Monotonic Epoch Fencing Tokens & ZGC |
| [**08**](./08-clock-skew-distributed-systems.md) | **NTP Clock Skew & Snowflake Collisions** | Ordering | NTP backward step caused duplicate IDs $\to$ Drift Detection & Clock Slewing |
| [**09**](./09-eventual-consistency-race-condition.md) | **Read-Your-Own-Writes Violation** | Consistency | Read replica lag showed stale profile $\to$ Primary Routing & Causal Cookies |
| [**10**](./10-distributed-deadlock-two-phase-commit.md) | **Distributed Deadlock in 2PC** | Transactions | Inconsistent lock ordering froze database $\to$ Event-Driven Saga Pattern |
| [**11**](./11-high-cardinality-metrics-oom.md) | **High-Cardinality Metric Explosion** | Observability | UUID in Prometheus labels crashed TSDB $\to$ Label Cardinality Limits |
| [**12**](./12-cdn-origin-saturation-stampede.md) | **Global CDN Purge & Origin Collapse** | Ingress | Wildcard purge sent 500k QPS to origin $\to$ Origin Shielding & Versioned URLs |
| [**13**](./13-connection-pool-exhaustion-deadlock.md) | **Nested DB Transactions Pool Deadlock** | Database | Slow HTTP in `@Transactional` starved pool $\to$ Narrow Transaction Boundaries |
| [**14**](./14-kafka-consumer-group-rebalance-storm.md) | **Kafka Consumer Rebalance Storm** | Messaging | 6-min processing exceeded poll timeout $\to$ Thread Pools & Cooperative Sticky Assignor |
| [**15**](./15-dns-ttl-misconfiguration-outage.md) | **DNS TTL Misconfiguration Outage** | Networking | 24-hour TTL sent traffic to dead IP $\to$ 60-second TTL & Reverse Proxies |
