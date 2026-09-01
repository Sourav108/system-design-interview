# Module 03: Distributed Systems Fundamentals

Deep dive into the physical, mathematical, and algorithmic foundations of distributed systems engineering.

---

## 📚 Lessons in this Module

1. [**01: Network Abstractions**](./01-network-abstraction.md) — Transport protocols, HTTP/1 vs HTTP/2 vs HTTP/3, gRPC binary framing, connection pooling.
2. [**02: Partial Failure & Failure Models**](./02-partial-failure-and-failure-models.md) — Crash-Stop, Crash-Recovery, Byzantine faults, tri-state timeouts, heartbeats, leases.
3. [**03: Distributed State Management**](./03-distributed-state-management.md) — Stateless compute tiers, distributed session stores, Write-Ahead Logs (WAL).
4. [**04: Replication Strategies**](./04-replication-strategies.md) — Single-Leader, Multi-Leader, Dynamo-style Quorum math ($R + W > N$).
5. [**05: Partitioning & Sharding**](./05-partitioning-and-sharding.md) — Range sharding, hash sharding, consistent hashing with virtual nodes.
6. [**06: Consistency Models**](./06-consistency-models.md) — Linearizability, Sequential, Causal, Read-Your-Writes, Eventual consistency.
7. [**07: CAP & PACELC Theorems**](./07-cap-and-pacelc-theorems.md) — Formal definitions, network partition realities, and PACELC taxonomy.
8. [**08: Time, Clocks & Ordering**](./08-time-clocks-and-ordering.md) — NTP clock drift, Lamport Logical Clocks, Vector Clocks, Google TrueTime ($\epsilon$).
9. [**09: Distributed Consensus**](./09-distributed-consensus.md) — Paxos, Raft leader election, quorum log replication, epoch fencing.
10. [**10: Idempotency & Exactly-Once**](./10-idempotency-and-exactly-once.md) — At-least-once delivery, idempotency key architectures, 2PC vs Saga patterns.
