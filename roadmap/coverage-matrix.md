# System Design Comprehensive Coverage Matrix

This matrix maps every foundational theoretical concept to its concrete building block, runnable Java implementation, end-to-end case study, interview question bank, and quick-revision cheat sheet.

---

| Domain / Concept | Fundamental Lesson | Building Block | Runnable Implementation | End-to-End Case Study | Interview Questions | Cheat Sheet |
|---|---|---|---|---|---|---|
| **Networking & Ingress** | `DST-01`, `DST-02` | `BB-01` (DNS), `BB-02` (LB), `BB-19` (Gateway) | `IMP-09` (API Gateway) | All Case Studies | 10 Ingress Questions | `load-balancing.md` |
| **Distributed Caching** | `DST-05`, `NFR-04` | `BB-10` (Distributed Cache) | `IMP-03` (L1/L2 Cache) | `CS-06`, `CS-07`, `CS-09` | 10 Caching Questions | `caching.md` |
| **Relational Databases** | `DST-04`, `NFR-06` | `BB-03` (Distributed RDBMS) | `IMP-08` (Idempotent Ledger) | `CS-15` (Payment Gateway) | 10 Database Questions | `database-selection.md` |
| **NoSQL Key-Value / LSM** | `DST-05`, `DST-06` | `BB-04` (Distributed KV Store) | `IMP-02` (Snowflake URL Shortener)| `CS-09`, `CS-11` | 10 NoSQL Questions | `database-selection.md` |
| **Sharding & Consistent Hashing**| `DST-05` | `BB-04`, `BB-10`, `BB-18` | `IMP-02`, `IMP-03` | `CS-06`, `CS-09`, `CS-12` | 10 Sharding Questions | `partitioning.md` |
| **Replication & Quorum** | `DST-04`, `DST-06` | `BB-03`, `BB-04` | `IMP-03`, `IMP-05` | `CS-01`, `CS-06`, `CS-15` | 10 Replication Questions| `replication.md` |
| **Consistency Models & CAP** | `DST-06`, `DST-07` | `BB-03`, `BB-21` | `IMP-05`, `IMP-08` | `CS-11`, `CS-13`, `CS-15` | 10 Consistency Questions| `consistency.md` |
| **Distributed Sequencing & IDs**| `DST-08` | `BB-06`, `BB-24` | `IMP-02` (Snowflake ID) | `CS-06`, `CS-09` | 5 Sequencer Questions | `system-design-interview.md` |
| **Event Streaming & Pub/Sub** | `DST-04`, `DST-10` | `BB-11` (Queue), `BB-12` (Kafka) | `IMP-04` (Kafka Streaming) | `CS-05`, `CS-06`, `CS-14` | 10 Kafka Questions | `kafka.md`, `messaging.md` |
| **Distributed Rate Limiting** | `NFR-02`, `NFR-03` | `BB-13` (Rate Limiter) | `IMP-01` (Redis Token Bucket)| `CS-09`, `CS-15`, `CS-17` | 5 Rate Limiting Questions| `reliability.md` |
| **Distributed Locks & Consensus**| `DST-09` | `BB-21` (Lock), `BB-31` (Leader) | `IMP-05` (Redlock / Leader) | `CS-14`, `CS-15`, `CS-16` | 10 Consensus Questions | `reliability.md` |
| **Distributed Task Schedulers** | `DST-03` | `BB-17` (Scheduler), `BB-27` (Job) | `IMP-06` (Task Scheduler) | `CS-10`, `CS-14`, `CS-16` | 5 Scheduling Questions | `reliability.md` |
| **Notification Fan-out** | `DST-01`, `NFR-05` | `BB-23` (Notification Engine) | `IMP-07` (Fan-out Engine) | `CS-06`, `CS-07`, `CS-11` | 5 Notification Questions| `messaging.md` |
| **Financial Idempotency & Saga** | `DST-10` | `BB-36` (Payment Idempotency) | `IMP-08` (Idempotent Ledger) | `CS-15` (Payment Gateway) | 10 Payment Questions | `consistency.md` |
| **Geospatial Indexing** | `DST-05` | `BB-32` (Geohash / Quadtree / S2) | None | `CS-03`, `CS-04`, `CS-05` | 5 Geospatial Questions | `database-selection.md` |
| **Search & Inverted Index** | `DST-05` | `BB-15` (Distributed Search) | None | `CS-02`, `CS-04`, `CS-12` | 5 Search Questions | `database-selection.md` |
| **Real-Time Collaboration** | `DST-06`, `DST-08` | `BB-35` (Sync), `BB-37` (WebSocket)| None | `CS-13` (Google Docs) | 5 Sync Questions | `consistency.md` |
| **Video Streaming & CDNs** | `NFR-04`, `NFR-05` | `BB-05` (CDN), `BB-14` (Blob Store)| None | `CS-01` (YouTube / Netflix) | 5 Media Questions | `load-balancing.md` |
| **AI Inference & LLM Gateways** | `AI-01`, `AI-02`, `AI-04`| `BB-40` (AI Gateway) | None | `CS-17`, `CS-19`, `CS-20` | 10 AI Questions | `ai-system-design.md` |
| **Vector Search & RAG Pipelines**| `AI-06`, `AI-07` | `BB-39` (Vector Search HNSW) | None | `CS-17`, `CS-19`, `CS-20` | 10 Vector Questions | `ai-system-design.md` |
| **Failure Analysis & Post-Mortems**| `08-system-failures` | All Building Blocks | `IMP-10` (Circuit Breaker) | All Case Studies | 20 Failure Scenarios | `reliability.md` |
| **Capacity Estimation Math** | `05-capacity-estimation`| All Building Blocks | All Implementations | All Case Studies | 30 Estimation Questions| `capacity-estimation.md` |
