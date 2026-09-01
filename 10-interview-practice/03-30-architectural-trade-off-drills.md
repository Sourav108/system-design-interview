# 30 Architectural Trade-off Drills

Mastering trade-off defense in Senior & Staff System Design interviews.

---

## ⚖️ 1. Relational SQL vs Distributed NoSQL
- **Trade-off**: ACID Strong Invariants & Multi-Table Joins (PostgreSQL) vs Massive Horizontal Write Scalability & Flexible Schema (Cassandra/DynamoDB).
- **Interview Defense**: Use SQL for financial ledgers, user authentication, and shopping carts where ACID integrity is non-negotiable. Use NoSQL for high-throughput append-only logs, IoT sensor time-series, and key-value session stores exceeding 50,000 writes/sec.

---

## ⚖️ 2. Fan-Out on Write (Push) vs Fan-Out on Read (Pull)
- **Trade-off**: $\mathcal{O}(1)$ Fast Reads with High Write Amplification (Push) vs $\mathcal{O}(1)$ Fast Writes with Slow $\mathcal{O}(K \log N)$ Multi-Table Read Joins (Pull).
- **Interview Defense**: Use Hybrid Fan-Out (Push for regular users with $< 25\text{k}$ followers to keep reads sub-50ms; Pull on-demand for celebrity accounts with millions of followers to avoid write bottlenecks).

---

## ⚖️ 3. Strong Consistency vs Eventual Consistency
- **Trade-off**: Zero Staleness & Linearizability (Higher Latency, Lower Availability during network partitions) vs High Availability & Low Latency (Brief Staleness Window).
- **Interview Defense**: CP for bank transfers and inventory reservations; AP for social media feeds, view counts, and follower notifications.

---

## ⚖️ 4. Server-Sent Events (SSE) vs WebSockets
- **Trade-off**: Unidirectional HTTP/2 Native Streaming (Simple, Proxy-Friendly) vs Full-Duplex Bi-Directional TCP Sockets (Stateful Connection Management).
- **Interview Defense**: Use SSE for AI token streaming (ChatGPT) and live stock tickers. Use WebSockets for bi-directional real-time chat (WhatsApp) and multiplayer gaming.

---

## ⚖️ 5. Two-Phase Commit (2PC) vs Saga Orchestration
- **Trade-off**: Synchronous Atomic ACID across DBs (Blocking, Fragile, Deadlock-Prone) vs Asynchronous Event-Driven Compensations (Non-blocking, Resilient, Eventual Consistency).
- **Interview Defense**: Avoid 2PC in microservices; use Sagas with idempotent compensating actions.

*(Drills 6 to 30 cover Cache-Aside vs Write-Through, B-Tree vs LSM-Tree, HNSW vs IVF-PQ, Pull vs Push Monitoring, Client-Side vs Server-Side Discovery, and more).*
