# 100 System Design Interview Questions & Categorized Bank

A comprehensive collection of 100 System Design interview questions categorized by core infrastructure domain.

---

## 🌐 1. Ingress & Traffic Management (Q1 - Q15)
1. How does Anycast BGP routing direct global users to the nearest datacenter?
2. Explain the difference between Layer 4 (Transport) and Layer 7 (Application) load balancing.
3. How does Maglev consistent hashing eliminate load balancer connection disruption during scale-out?
4. What is the Power of Two Random Choices (P2C) algorithm and why is it superior to Round Robin?
5. How does TLS 1.3 0-RTT session resumption work, and what is its security trade-off (Replay attacks)?
6. When should you use an API Gateway versus a decentralized Service Mesh (Sidecar)?
7. How does a Reverse Proxy prevent Slowloris DDoS attacks?
8. Explain the mechanics of HTTP/2 Multiplexing vs HTTP/3 QUIC over UDP.
9. How does GeoDNS resolve client IP subnets using EDNS Client Subnet (ECS)?
10. How do you implement zero-downtime blue/green traffic shifting at the load balancer tier?
11. What is the Backend for Frontend (BFF) architectural pattern?
12. How does an API Gateway handle JWT validation with zero database queries?
13. How do you mitigate SYN flood attacks using SYN Cookies?
14. Explain connection draining during container pod termination.
15. What are the trade-offs of terminating SSL at the Load Balancer vs end-to-end mTLS?

---

## 💾 2. Storage & Databases (Q16 - Q35)
16. How does Write-Ahead Logging (WAL) guarantee durability before committing to B-Tree pages?
17. Explain how Multi-Version Concurrency Control (MVCC) eliminates read-write lock contention.
18. What is the difference between B+ Trees and Log-Structured Merge-trees (LSM-Trees)?
19. How does Leveled Compaction in RocksDB reduce read amplification?
20. What is a Bloom Filter and how does it prevent unnecessary disk seeks in SSTables?
21. Explain the 4 ANSI SQL isolation levels and the anomalies they prevent.
22. How does Google Spanner achieve External Consistency (Linearizability) without 2PC deadlocks using TrueTime?
23. What is the difference between Row-Oriented (PostgreSQL) and Column-Oriented (ClickHouse/Parquet) storage?
24. How does Write Amplification differ between B-Trees and LSM-Trees?
25. How do you handle database table schema migrations on a 100-million row table with zero downtime?
26. What is the difference between Synchronous and Asynchronous database replication?
27. How does Change Data Capture (CDC via Debezium) stream database mutations to Kafka?
28. Explain Range-based Sharding vs Hash-based Consistent Sharding.
29. How do you solve the 'Hot Partition' problem in Cassandra or DynamoDB?
30. What is Phantom Read anomaly and which isolation level prevents it?
31. How does Read Repair work in Cassandra leaderless replication?
32. What is Hinted Handoff and how does it maintain write availability during node downtime?
33. How does Postgres MVCC vacuuming reclaim dead tuple disk space?
34. Explain Database Connection Pooling mechanics (HikariCP / PgBouncer).
35. When should you choose Amazon S3 Object Storage over a Distributed SQL database?

---

## ⚡ 3. Caching & Memory (Q36 - Q50)
36. Explain Cache-Aside vs Write-Through vs Write-Behind (Write-Back) caching patterns.
37. How do you prevent a Cache Stampede (Thundering Herd) using Singleflight mutex locking?
38. What is Probabilistic Early Expiration (XFetch) and how does it prevent cache misses?
39. How does Redis Cluster partition keys across 16,384 hash slots using `CRC16(key) % 16384`?
40. Explain the LRU (Least Recently Used) eviction algorithm implementation using a Doubly Linked List and Hash Map.
41. What is the difference between LRU, LFU, and 2Q / ARC eviction policies?
42. How do you prevent Cache Penetration (queries for non-existent IDs)?
43. How does Redis Sentinel achieve automated master failover and split-brain defense?
44. Why is Redis single-threaded for command execution, and how does it handle 100k QPS?
45. What is the difference between Redis RDB snapshots and AOF (Append-Only File) persistence?
46. How do you invalidate distributed caches across multiple regions in real time?
47. What is the 80-20 Pareto rule in memory sizing?
48. How does Client-Side Local Caching (L1 Caffeine + L2 Redis) maintain consistency?
49. What are Redis Sorted Sets (`ZSET`) and how do they achieve $\mathcal{O}(\log N)$ leaderboard operations?
50. How do you handle Redis memory fragmentation?

---

## 📬 4. Messaging & Stream Processing (Q51 - Q65)
51. What is the difference between a Message Queue (RabbitMQ) and an Event Log (Kafka)?
52. How does Apache Kafka achieve zero-copy I/O using the Linux `sendfile()` system call?
53. What is an In-Sync Replica (ISR) set in Kafka, and what does `acks=all` guarantee?
54. How does a Kafka Consumer Group rebalance work, and what is the Cooperative Sticky Assignor?
55. Explain Log Compaction in Kafka and its use case for changelogs.
56. What is the difference between At-Least-Once, At-Most-Once, and Exactly-Once (EOS) processing?
57. How do Watermarks handle late, out-of-order data in Apache Flink stream processing?
58. Explain the Chandy-Lamport distributed snapshot algorithm used in Flink checkpointing.
59. What is a Dead-Letter Queue (DLQ) and when should messages be routed to it?
60. How does RabbitMQ handle consumer backpressure using prefetch limits (`basic.qos`)?
61. What is Schema Registry and how does it enforce Backward and Forward schema compatibility?
62. How does Tumbling Window differ from Sliding Window and Session Window?
63. What is Message Deduplication / Idempotent Consumer pattern?
64. How do you preserve strict message ordering across multiple Kafka partitions?
65. Explain the Transactional Outbox Pattern for atomic database writes and message publishing.

---

## ⏱️ 5. Distributed Coordination & Consensus (Q66 - Q80)
66. How does Raft consensus elect a leader using randomized election timers?
67. What is Split-Brain and how does majority quorum ($N/2 + 1$) prevent it?
68. Explain why Fencing Tokens are mandatory when using distributed locks (Redlock / Zookeeper).
69. How does Lamport Logical Clocks establish a total causal ordering of events?
70. What is Vector Clocks and how do they detect concurrent conflicting writes?
71. How does the Redlock algorithm acquire locks across 5 independent Redis masters?
72. What is the Two-Phase Commit (2PC) protocol and why is it considered blocking?
73. How does the Saga Pattern with compensating transactions replace 2PC?
74. How does the SWIM Gossip protocol detect node failures in HashiCorp Consul?
75. What is the Bully Leader Election algorithm?
76. How does etcd use Raft to provide linearizable key-value reads and watches?
77. What is an Epoch / Term generation counter in distributed state machines?
78. How does distributed rate limiting work across multiple regions without lock contention?
79. What is the difference between Pessimistic Locking and Optimistic Concurrency Control (OCC)?
80. How do you implement a distributed delay scheduler using Redis Sorted Sets?

---

## 🛡️ 6. Reliability, Observability & AI Systems (Q81 - Q100)
81. Explain the 3 states of a Circuit Breaker (CLOSED, OPEN, HALF-OPEN).
82. What is the difference between Exponential Backoff and Full Jitter?
83. How does Gorilla TSDB compress floating-point time-series metrics using XOR delta compression?
84. What is High-Cardinality label explosion in Prometheus and how do you prevent it?
85. How does Distributed Tracing propagate `trace_id` and `span_id` across microservices (W3C Trace Context)?
86. What is the difference between SLA, SLO, and SLI?
87. How does HNSW graph search achieve $\mathcal{O}(\log N)$ Approximate Nearest Neighbor vector search?
88. What is the difference between the Prefill (compute-bound) and Decode (memory-bound) phases of LLM inference?
89. How does PagedAttention eliminate GPU VRAM fragmentation in KV Caching?
90. Explain Continuous Batching (Iteration-level scheduling) in vLLM.
91. What is Semantic Prompt Caching and what cosine similarity threshold is standard?
92. How does Hybrid RAG combine Dense Vector Search with Sparse BM25 Keyword Search?
93. What is Speculative Decoding and how does it speed up LLM token generation by $2\times$?
94. How does Content-Defined Chunking (FastCDC) solve the boundary shift problem in file synchronization?
95. How does Double-Entry Bookkeeping guarantee financial ledger integrity in Stripe?
96. How do you implement Idempotency Keys using Redis atomic operations?
97. How does Google S2 Geometry project latitude/longitude coordinates onto a 1D Hilbert space-filling curve?
98. What is the difference between Fan-Out on Write (Push) and Fan-Out on Read (Pull) in newsfeeds?
99. How does Operational Transformation (OT) resolve concurrent text edits in Google Docs?
100. How do Linux cgroups v2 and seccomp-bpf isolate untrusted user code in online judges?
