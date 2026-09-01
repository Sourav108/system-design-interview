# Module 09: Runnable Reference Implementations

Ten production-grade, runnable reference implementations in **Java 21 / Spring Boot 3 / Docker / Redis / Kafka / PostgreSQL** demonstrating fundamental distributed system building blocks and concurrency algorithms.

---

## 💻 Catalog of 10 Reference Implementations

| ID | Implementation | Primary Technology | Key Architectural Features |
|---|---|---|---|
| [**01**](./01-distributed-rate-limiter/) | **Distributed Rate Limiter** | Redis + Lua | Token Bucket, Sliding Window Counter, 429 Headers |
| [**02**](./02-distributed-lock-redis/) | **Distributed Lock Manager** | Redis + Jedis | Redlock Algorithm, Background Watchdog, Fencing Tokens |
| [**03**](./03-distributed-id-snowflake/) | **Distributed ID Sequencer** | Java 21 | Twitter Snowflake, 64-bit Bitwise Ops, Clock Drift Protection |
| [**04**](./04-consistent-hashing-ring/) | **Consistent Hash Ring** | Java + Guava | Virtual Nodes (200/node), MurmurHash3, Minimal Key Rehash |
| [**05**](./05-lru-cache-multithreaded/) | **Concurrent LRU Cache** | Java 21 Concurrent | Lock-Striped Doubly Linked List, $\mathcal{O}(1)$ Concurrent Ops |
| [**06**](./06-url-shortener-base62/) | **TinyURL Microservice** | Spring Boot + Postgres | Base62 Bijective Mapping, Redis Cache-Aside, HTTP 302 |
| [**07**](./07-bloom-filter-concurrent/) | **Counting Bloom Filter** | Atomic Bit Arrays | Optimal Hash Functions, Dynamic Deletions, 1% False Positive |
| [**08**](./08-kafka-pub-sub-consumer/) | **Kafka Consumer with DLQ** | Spring Kafka | Idempotent Producer, Manual ACKs, Dead-Letter Queues |
| [**09**](./09-circuit-breaker-resilience4j/) | **Resilience4j Circuit Breaker** | Resilience4j | Circuit Breaker, Threadpool Bulkhead, TimeLimiter Fallback |
| [**10**](./10-payment-idempotency-engine/) | **Payment Idempotency & Ledger** | Spring + Postgres | Idempotency Keys, Redis Mutex, Double-Entry Balanced Ledger |
