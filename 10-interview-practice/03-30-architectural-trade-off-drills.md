# 30 Architectural Trade-off Drills

A comprehensive collection of **30 complete architectural trade-off drills** structured with core options, detailed technical trade-offs, multi-dimensional decision matrices, and interview defense playbooks for Senior & Staff System Design interviews.

---

## ⚖️ 1. Relational SQL vs Distributed NoSQL
- **Options**: ACID Relational RDBMS (PostgreSQL, MySQL) vs Horizontally Partitioned NoSQL (Cassandra, DynamoDB).
- **Trade-off**: ACID Strong Invariants & Multi-Table Joins (SQL) vs Massive Horizontal Write Scalability & Tunable Consistency (NoSQL).
- **Decision Matrix**:
  - Use **SQL** when entity relationships are dense, multi-table transactions are required, and write volume is $< 20,000\text{ TPS}$ (Financial ledgers, user auth, shopping carts).
  - Use **NoSQL** when data schema is dynamic, access is strictly by single primary key, and write volume exceeds $100,000\text{ writes/sec}$ (IoT telemetry, chat messages, clickstreams).
- **Interview Defense**: *"I choose PostgreSQL because financial transaction ledgers require strict ACID serializability and multi-row debit/credit balance integrity. NoSQL's eventual consistency risks silent ledger drift."*

---

## ⚖️ 2. Fan-Out on Write (Push) vs Fan-Out on Read (Pull)
- **Options**: Push to follower feeds during post creation vs Query all followed accounts on-demand during feed view.
- **Trade-off**: $\mathcal{O}(1)$ Fast Reads with High Write Amplification (Push) vs $\mathcal{O}(1)$ Fast Writes with Slow $\mathcal{O}(K \log N)$ Multi-Table Read Joins (Pull).
- **Decision Matrix**:
  - Use **Push (Fan-Out on Write)** for regular users with $< 25\text{k}$ followers to achieve sub-50ms timeline loads.
  - Use **Pull (Fan-Out on Read)** for celebrity accounts with millions of followers to prevent write saturation.
- **Interview Defense**: *"I implement a Hybrid Fan-Out architecture: standard users push post IDs to follower Redis timelines asynchronously via Kafka; celebrity posts are pulled and merged on-the-fly during timeline read to avoid 80M write storms."*

---

## ⚖️ 3. Strong Consistency (CP) vs Eventual Consistency (AP)
- **Options**: Linearizable consistency (Raft/Spanner) vs High availability with eventual convergence (Cassandra/DynamoDB).
- **Trade-off**: Guaranteed zero staleness at the cost of higher write latency and failure during network partitions vs 100% write availability with brief read staleness windows.
- **Decision Matrix**:
  - Use **CP** for inventory allocation, bank account balances, and distributed locks.
  - Use **AP** for social media likes, view counters, newsfeeds, and product reviews.
- **Interview Defense**: *"For seat reservations, I choose a CP model using Spanner/Raft. It is far better to return a temporary 503 error during a partition than to double-sell a single airline seat to two different customers."*

---

## ⚖️ 4. Server-Sent Events (SSE) vs WebSockets
- **Options**: Unidirectional HTTP/2 SSE vs Full-Duplex TCP WebSockets.
- **Trade-off**: Lightweight, proxy-friendly, HTTP/2 native streaming (SSE) vs Bi-directional low-latency socket communication with stateful connection management (WebSockets).
- **Decision Matrix**:
  - Use **SSE** for LLM token streaming (ChatGPT), live stock tickers, and news feeds (Server $\to$ Client only).
  - Use **WebSockets** for real-time multiplayer games, collaborative drawing (Figma), and chat applications (WhatsApp).
- **Interview Defense**: *"I select Server-Sent Events (SSE) over WebSockets for AI chat streaming because the client only sends one prompt and receives a stream of tokens. SSE runs natively over HTTP/2, eliminates socket connection tracking servers, and works seamlessly through corporate firewalls."*

---

## ⚖️ 5. Two-Phase Commit (2PC) vs Saga Pattern
- **Options**: Synchronous Atomic Distributed Transactions (2PC) vs Asynchronous Event-Driven Compensations (Saga).
- **Trade-off**: Strict global atomicity with blocking resource locks and high deadlock risk (2PC) vs Non-blocking, resilient, eventually consistent execution with complex compensating rollbacks (Saga).
- **Decision Matrix**:
  - Avoid **2PC** across microservices (it reduces availability to the product of all service availabilities $A = A_1 \times A_2 \times A_3$).
  - Use **Sagas (Orchestrated with Temporal)** for multi-service business workflows (Order $\to$ Payment $\to$ Inventory $\to$ Shipping).
- **Interview Defense**: *"I reject 2PC because holding database locks across network RPCs causes cascading connection pool exhaustion. I implement an Orchestrated Saga with asynchronous compensating transactions to guarantee forward or backward recovery."*

---

## ⚖️ 6. Cache-Aside vs Write-Through vs Write-Behind
- **Options**: Application manages cache (Cache-Aside) vs Cache updates DB synchronously (Write-Through) vs Cache updates DB asynchronously in batches (Write-Behind).
- **Trade-off**: Resilient lazy loading with potential cache miss latency (Cache-Aside) vs Guaranteed cache freshness with write latency penalty (Write-Through) vs Ultra-fast writes with risk of data loss on cache crash (Write-Behind).
- **Decision Matrix**:
  - Use **Cache-Aside** for general read-heavy web applications.
  - Use **Write-Through** for read-heavy critical data that must never be stale.
  - Use **Write-Behind** for high-frequency counters (video view counters, analytics beacons).
- **Interview Defense**: *"I choose Cache-Aside for product catalog reads to prevent cold cache bloat, combined with singleflight mutex locks to eliminate cache stampedes on viral items."*

---

## ⚖️ 7. B+ Tree Indexing vs Log-Structured Merge-tree (LSM-Tree)
- **Options**: In-place disk updates with balanced trees (PostgreSQL/MySQL) vs Append-only sequential disk writes with background compaction (Cassandra/RocksDB).
- **Trade-off**: Fast point reads ($\mathcal{O}(\log N)$ disk seek) with write amplification (B+ Tree) vs Massive write throughput with read amplification and background compaction CPU overhead (LSM-Tree).
- **Decision Matrix**:
  - Use **B+ Trees** for read-heavy or transactional point-update workloads (70% Reads, 30% Writes).
  - Use **LSM-Trees** for write-heavy ingestion ($> 100\text{k writes/sec}$, append-only time-series, logs).
- **Interview Defense**: *"I choose RocksDB's LSM-Tree for our time-series ingestion engine because LSM converts random writes into sequential disk appends, maximizing NVMe SSD write throughput."*

---

## ⚖️ 8. HNSW Vector Search vs Inverted File Product Quantization (IVF-PQ)
- **Options**: Graph-based ANN (HNSW) vs Inverted File Index with Vector Quantization (IVF-PQ).
- **Trade-off**: Ultra-high recall ($> 98\%$) and sub-10ms query speed with high RAM footprint (HNSW) vs $4\times - 8\times$ lower RAM consumption and faster index build with slightly lower recall ($90\%-95\%$) (IVF-PQ).
- **Decision Matrix**:
  - Use **HNSW** for low-latency, mission-critical RAG customer support and code completion ($< 100\text{M}$ vectors).
  - Use **IVF-PQ** for billion-scale web image/video retrieval where RAM cost dominates.
- **Interview Defense**: *"I select HNSW for our AI Coding Assistant because inline code completion demands $< 15\text{ms}$ vector retrieval and maximum precision. We apply INT8 Scalar Quantization to halve RAM overhead without degrading recall."*

---

## ⚖️ 9. Pull-Based Monitoring (Prometheus) vs Push-Based Telemetry (StatsD/OTel)
- **Options**: Central server scrapes HTTP endpoints (Pull) vs Microservice agents push metrics to central collector (Push).
- **Trade-off**: Centralized health verification, firewall-friendly, auto-rate control (Pull) vs Low client overhead, optimal for short-lived serverless functions (Push).
- **Decision Matrix**:
  - Use **Pull (Prometheus)** for persistent microservice container clusters and Kubernetes nodes.
  - Use **Push (OpenTelemetry Collector / StatsD)** for ephemeral AWS Lambda functions, batch jobs, and UDP metrics.
- **Interview Defense**: *"I use Prometheus pull scraping for core microservices so the monitoring system controls scrape frequency and cannot be overwhelmed by client traffic spikes. Ephemeral Lambda jobs push to an OpenTelemetry collector sidecar."*

---

## ⚖️ 10. Client-Side Service Discovery vs Server-Side Discovery
- **Options**: Client queries registry (Eureka/Consul) and load balances directly vs Client calls a reverse proxy / Load Balancer (AWS ALB / K8s CoreDNS).
- **Trade-off**: Zero extra network proxy hops with language-specific client logic (Client-Side) vs Language-agnostic, simple client architecture with an additional network proxy hop (Server-Side).
- **Decision Matrix**:
  - Use **Client-Side** (gRPC / Envoy xDS) for ultra-high-throughput internal microservice RPCs where eliminating a 2ms proxy hop is critical.
  - Use **Server-Side** (K8s Service / ALB) for external client ingress and polyglot microservice environments.
- **Interview Defense**: *"I use Kubernetes Server-Side discovery with CoreDNS and Envoy for standard services to keep microservices decoupled from discovery logic, but use gRPC client-side load balancing for ultra-low latency billing RPCs."*

---

## ⚖️ 11. Monolithic Database vs Database-per-Service
- **Options**: Shared central database for all services vs Strict private database per microservice.
- **Trade-off**: Easy cross-domain joins and ACID transactions with tight coupling and single point of failure (Monolithic DB) vs Domain isolation, independent scaling, and fault boundary with complex cross-service event choreography (Database-per-Service).
- **Decision Matrix**:
  - Use **Monolithic DB** for early-stage startups and small monolithic applications.
  - Use **Database-per-Service** for large engineering teams requiring independent deployability and failure isolation.
- **Interview Defense**: *"I enforce Database-per-Service to prevent teams from creating hidden SQL coupling across domain boundaries. Data sharing across services occurs strictly via versioned gRPC APIs or Kafka domain events."*

---

## ⚖️ 12. Synchronous REST/gRPC vs Asynchronous Message Queues
- **Options**: Direct blocking HTTP/gRPC request-response vs Decoupled event publishing via Kafka/RabbitMQ.
- **Trade-off**: Immediate confirmation with temporal coupling and cascading failure risk (Synchronous) vs Resilient temporal decoupling and traffic smoothing with eventual consistency (Asynchronous).
- **Decision Matrix**:
  - Use **Synchronous (gRPC)** when the caller cannot proceed without the result (User login token verification, checkout price quote).
  - Use **Asynchronous (Kafka)** for notifications, search indexing, video transcoding, and analytics.
- **Interview Defense**: *"Order placement uses synchronous gRPC to validate credit card authorization with Stripe, but offloads invoice generation, email dispatch, and inventory analytics asynchronously to Kafka."*

---

## ⚖️ 13. Content-Defined Chunking (FastCDC) vs Fixed-Size Chunking
- **Options**: Variable-length chunk boundaries determined by rolling hash (FastCDC) vs Fixed 4MB blocks.
- **Trade-off**: Robust to byte insertions with slight compute overhead (FastCDC) vs Fast zero-compute chunking that breaks completely upon inserting a single byte at the start of a file (Fixed-Size).
- **Decision Matrix**:
  - Use **FastCDC** for document and file sync platforms (Dropbox, Google Drive, Docker layer caching).
  - Use **Fixed-Size** for raw block storage volumes (AWS EBS) and video streaming chunks (HLS).
- **Interview Defense**: *"I select FastCDC for cloud file sync because inserting a 1-sentence paragraph into a 100MB document changes only 1 chunk, saving 99% of re-upload bandwidth."*

---

## ⚖️ 14. Twitter Snowflake 64-bit ID vs 128-bit UUIDv4 vs UUIDv7
- **Options**: 64-bit roughly time-ordered integer (Snowflake) vs 128-bit random string (UUIDv4) vs 128-bit time-ordered integer (UUIDv7).
- **Trade-off**: Compact, index-friendly, highly performant 8-byte integer (Snowflake) vs Non-monotonic B-Tree fragmentation (UUIDv4) vs Larger 16-byte storage with built-in time-ordering (UUIDv7).
- **Decision Matrix**:
  - Use **Snowflake** for primary keys in distributed databases (Cassandra/Postgres) where compact 64-bit integers optimize index page density.
  - Use **UUIDv7** when a decentralized 128-bit ID is needed without managing worker node IDs.
  - Avoid **UUIDv4** as primary keys on clustered B-Tree indexes.
- **Interview Defense**: *"I use 64-bit Snowflake IDs because sequential IDs prevent random B-Tree disk fragmentation and fit inside standard 8-byte integer columns, doubling database index cache density."*

---

## ⚖️ 15. Redis In-Memory Caching vs Memcached
- **Options**: Single-threaded rich data structures with persistence (Redis) vs Multi-threaded pure key-value memory cache (Memcached).
- **Trade-off**: Rich structures (Hashes, Sorted Sets, Bitmaps), Pub/Sub, Lua scripts, and RDB/AOF persistence (Redis) vs Simple multi-threaded horizontal core scaling for pure static blobs (Memcached).
- **Decision Matrix**:
  - Use **Redis** for leaderboards (`ZSET`), rate limiters, session stores, and complex data structures.
  - Use **Memcached** for massive multi-threaded caching of static pre-rendered HTML/JSON blobs.
- **Interview Defense**: *"I choose Redis because its native Sorted Sets allow $\mathcal{O}(\log N)$ timeline pagination and Lua scripts allow atomic rate-limiting checks without network round-trip race conditions."*

---

## ⚖️ 16. Push CDN vs Pull CDN
- **Options**: Origin explicitly pushes assets to edge servers ahead of time (Push) vs Edge POP fetches on-demand on first cache miss (Pull).
- **Trade-off**: Zero cache miss latency for users with high origin compute/storage push overhead (Push) vs Minimal maintenance and on-demand caching with initial cache miss latency (Pull).
- **Decision Matrix**:
  - Use **Push CDN** for major scheduled software patch releases and popular movie pre-releases.
  - Use **Pull CDN** for user-generated content (Instagram photos, YouTube videos, web blogs).
- **Interview Defense**: *"I implement a Pull CDN with Origin Shielding for photo sharing to dynamically cache only the top 20% most popular photos, eliminating wasted edge storage on rarely viewed content."*

---

## ⚖️ 17. Row-Oriented Storage vs Column-Oriented Storage
- **Options**: Row-based storage (PostgreSQL, MySQL) vs Columnar storage (ClickHouse, Parquet, Snowflake).
- **Trade-off**: Fast single-record point CRUD operations ($\mathcal{O}(1)$ row writes) (Row-Oriented) vs $10\times - 100\times$ faster aggregations (`SUM`, `AVG`) and $4\times$ better compression across billions of rows (Columnar).
- **Decision Matrix**:
  - Use **Row-Oriented (PostgreSQL)** for OLTP transactional applications (Orders, Users, Payments).
  - Use **Column-Oriented (ClickHouse/Iceberg)** for OLAP analytics dashboards and clickstream aggregations.
- **Interview Defense**: *"I separate OLTP from OLAP: PostgreSQL handles transactional checkout mutations, while Debezium CDC streams events to ClickHouse for real-time merchant analytics aggregations."*

---

## ⚖️ 18. Stateful WebSocket Cluster vs Stateless REST Polling
- **Options**: Persistent full-duplex TCP connections vs Periodic client HTTP polling (e.g. every 2s).
- **Trade-off**: Sub-millisecond latency and 2-byte framing overhead with complex stateful connection management and rebalance storms (WebSockets) vs Simple load balancing and autoscaling with heavy HTTP header overhead and polling latency (REST Polling).
- **Decision Matrix**:
  - Use **WebSockets** for chat applications (WhatsApp), collaborative tools (Google Docs), and multiplayer games.
  - Use **REST / Short Polling** for infrequent status checks (Order delivery status updated once every 2 minutes).
- **Interview Defense**: *"I use WebSockets for ride-hailing live driver tracking to stream coordinates every 4s with 2-byte framing overhead. Polling would waste 1KB HTTP headers on every ping across 5M drivers."*

---

## ⚖️ 19. Edge TLS Termination vs End-to-End mTLS Encryption
- **Options**: Terminate TLS at Load Balancer and use plain HTTP internally vs Enforce mutual TLS (mTLS) across all internal microservice hops.
- **Trade-off**: High performance, simplified internal microservice routing, and lower CPU overhead (Edge TLS) vs Zero-Trust enterprise compliance with cryptographic verification and CPU crypto overhead (End-to-End mTLS).
- **Decision Matrix**:
  - Use **Edge TLS** for non-regulated low-risk web applications.
  - Use **End-to-End mTLS** (via Istio/Linkerd) for banking, healthcare (HIPAA), and payment (PCI-DSS) platforms.
- **Interview Defense**: *"I implement Edge TLS termination at the Cloudflare WAF for public ingress, and enforce internal mTLS via Envoy service mesh sidecars to guarantee Zero-Trust compliance for financial services."*

---

## ⚖️ 20. Master-Replica Async Replication vs Multi-Master Active-Active
- **Options**: Single writable master with async read replicas vs Multiple writable masters across global datacenters.
- **Trade-off**: Simple conflict-free writes with cross-region write latency (Master-Replica) vs Local sub-10ms writes globally with complex multi-master write conflicts (Multi-Master CRDT / LWW).
- **Decision Matrix**:
  - Use **Single-Master** for relational ACID databases where write conflicts are unacceptable.
  - Use **Multi-Master (DynamoDB Global Tables / CockroachDB)** for global shopping carts, user sessions, and bookmarking.
- **Interview Defense**: *"I choose Single-Master with read replicas for payment ledgers to eliminate write merge collisions, and use Multi-Master DynamoDB for global user session state."*

---

## ⚖️ 21. Optimistic Concurrency Control (OCC) vs Pessimistic Locking
- **Options**: Validate version on commit (`WHERE version = 1`) vs Exclusive row locking (`SELECT ... FOR UPDATE`).
- **Trade-off**: High throughput and zero lock contention under low conflict rates with retry overhead under high contention (OCC) vs Guaranteed conflict prevention with thread blocking and deadlock risk (Pessimistic).
- **Decision Matrix**:
  - Use **OCC** for user profile updates, document revisions, and e-commerce shopping cart updates.
  - Use **Pessimistic Locking** for high-contention flash sales and limited seat reservations where conflict rate exceeds 50%.
- **Interview Defense**: *"I use Optimistic Concurrency Control with atomic version increments for order state updates, but use Redis distributed locks for flash sale inventory decrement to prevent OCC retry storms."*

---

## ⚖️ 22. Singleflight Mutex vs Probabilistic Early Expiration (XFetch)
- **Options**: Mutex lock ensures only 1 thread re-computes missing cache key vs Background thread recomputes cache before TTL based on access probability.
- **Trade-off**: Zero redundant DB queries with brief read blocking on cold miss (Singleflight) vs Completely non-blocking reads with complex probabilistic background refresh math (XFetch).
- **Decision Matrix**:
  - Use **Singleflight** for general cache miss protection.
  - Use **XFetch** for ultra-hot viral content where even a 50ms read pause under heavy load is unacceptable.
- **Interview Defense**: *"I combine both: Singleflight protects against cold key stampedes, while XFetch asynchronously refreshes hot home timeline caches before they expire."*

---

## ⚖️ 23. Dead-Letter Queue (DLQ) vs Infinite Exponential Retries
- **Options**: Isolate unprocessable messages to a separate DLQ topic after 3 failed retries vs Retry indefinitely with exponential backoff.
- **Trade-off**: Unblocks primary queue and isolates poison pills with requirement for manual/automated DLQ recovery (DLQ) vs Preserves strict FIFO processing with risk of total queue blockage on poison pills (Infinite Retry).
- **Decision Matrix**:
  - Use **DLQ** for asynchronous decoupled processing (Order fulfillment, email dispatch).
  - Use **Infinite Retries with Circuit Breaking** only for critical synchronous financial accounting pipelines.
- **Interview Defense**: *"I route failed order events to a Dead-Letter Queue after 3 retries with full jitter. This prevents a single malformed poison-pill payload from halting our entire 50-worker consumer fleet."*

---

## ⚖️ 24. Pre-computed Feed Cache vs Dynamic On-Demand Joins
- **Options**: Pre-generate personalized feed lists in Redis vs Query and rank friend posts dynamically on every page load.
- **Trade-off**: Sub-30ms read latency with background fan-out compute and storage overhead (Pre-computed) vs Zero background write overhead with slow 500ms+ multi-table database join latency (Dynamic Joins).
- **Decision Matrix**:
  - Use **Pre-computed Feeds** for social platforms with heavy read-to-write ratios ($100:1$).
  - Use **Dynamic Joins** for private enterprise intranets with low read traffic and strict real-time access filters.
- **Interview Defense**: *"I pre-compute home timelines in Redis Sorted Sets so 99% of feed reads execute in $< 20\text{ms}$. Inactive users' feeds are evicted from Redis and generated on-demand only upon login."*

---

## ⚖️ 25. Kafka In-Sync Replicas `acks=all` vs `acks=1` vs `acks=0`
- **Options**: All ISR replicas must acknowledge vs Only partition leader acknowledges vs Fire-and-forget without waiting for ack.
- **Trade-off**: Zero data loss durability with higher producer write latency (acks=all) vs Low latency with risk of data loss on leader crash before replication (acks=1) vs Maximum throughput with zero delivery guarantee (acks=0).
- **Decision Matrix**:
  - Use **`acks=all` + `min.insync.replicas=2`** for financial payments, order events, and user authentication logs.
  - Use **`acks=1`** for general microservice domain events.
  - Use **`acks=0`** for loss-tolerant high-frequency metrics and clickstream telemetry.
- **Interview Defense**: *"Payment transaction events are published with `acks=all` and idempotent producers (`enable.idempotence=true`) to guarantee zero message loss during broker failovers."*

---

## ⚖️ 26. Continuous Batching (vLLM) vs Static Batching in AI Inference
- **Options**: Iteration-level scheduling inserting/evicting tokens dynamically on every step vs Request-level fixed batching.
- **Trade-off**: $3\times - 5\times$ higher GPU throughput with complex scheduling algorithms (Continuous) vs Simple architecture with massive GPU idle bubbles caused by variable sequence lengths (Static).
- **Decision Matrix**:
  - Use **Continuous Batching (vLLM)** for all production LLM conversational serving platforms.
  - Use **Static Batching** only for fixed-length offline batch embedding generation.
- **Interview Defense**: *"I deploy vLLM with Continuous Batching and PagedAttention to eliminate idle GPU compute bubbles and increase concurrent chat token throughput by 4x."*

---

## ⚖️ 27. Hybrid RAG (Dense + Sparse) vs Pure Dense Vector Search
- **Options**: Combines Vector ANN (Milvus) + Keyword BM25 (Elasticsearch) via Reciprocal Rank Fusion vs Dense Vector Search alone.
- **Trade-off**: Higher retrieval recall and accuracy on exact entity names/SKUs with dual index storage overhead (Hybrid) vs Simpler pipeline with poor performance on exact keyword queries (Pure Dense).
- **Decision Matrix**:
  - Use **Hybrid RAG** for enterprise customer support, legal search, and e-commerce catalogs.
  - Use **Pure Dense Vector Search** for cross-lingual semantic search and image similarity.
- **Interview Defense**: *"I use Hybrid RAG combining Milvus HNSW dense search with Elasticsearch BM25 sparse search so queries for exact order numbers and acronyms are matched accurately alongside conceptual queries."*

---

## ⚖️ 28. Direct Client-to-S3 Uploads (Presigned URLs) vs Backend Proxying
- **Options**: Client uploads file directly to S3 via Presigned URL vs Client streams file through backend API servers to S3.
- **Trade-off**: Zero backend server bandwidth saturation and horizontal S3 scaling (Direct Presigned) vs Full server-side file inspection before upload with severe backend network and thread starvation (Backend Proxying).
- **Decision Matrix**:
  - Use **Direct Presigned S3 Uploads** for large files ($> 1\text{MB}$: video uploads, photos, documents).
  - Use **Backend Proxying** only for tiny binary blobs ($< 50\text{KB}$) requiring instant synchronous validation.
- **Interview Defense**: *"I generate 5-minute S3 Presigned URLs for video uploads. This eliminates 100 TB/day of streaming bandwidth from our application servers and leverages AWS S3's native multi-part parallel upload scaling."*

---

## ⚖️ 29. Bloom Filters vs Exact Database Key Lookups
- **Options**: In-memory probabilistic set membership check vs Direct disk/database index lookup.
- **Trade-off**: Sub-microsecond memory check with 1% false positive rate (Bloom Filter) vs 100% deterministic precision with disk I/O and latency penalty (Exact DB Lookup).
- **Decision Matrix**:
  - Use **Bloom Filters** in Web Crawlers (Seen URL filter), LSM-Tree SSTables, and Cache Penetration defense.
  - Use **Exact DB Lookups** for final authorization decisions and user account existence checks.
- **Interview Defense**: *"I place a Counting Bloom Filter in front of our web crawler frontier to filter out 99% of previously seen URLs in memory before issuing any database or network requests."*

---

## ⚖️ 30. Centralized API Gateway vs Decentralized Sidecar Service Mesh
- **Options**: Central reverse proxy cluster (Kong, Spring Cloud Gateway) vs Envoy sidecar proxies deployed alongside every pod (Istio, Linkerd).
- **Trade-off**: Simple operational governance, central SSL termination, and single entry point (Central Gateway) vs End-to-end mTLS Zero-Trust security, fine-grained traffic shifting, and decentralized resilience with high Kubernetes operational complexity and memory overhead (Service Mesh).
- **Decision Matrix**:
  - Use **Central API Gateway** for external client edge ingress, public rate limiting, and consumer authentication.
  - Use **Service Mesh Sidecars** for internal microservice-to-microservice mTLS, canary routing, and distributed tracing in large multi-team microservice clusters ($> 50$ services).
- **Interview Defense**: *"I use a hybrid approach: a Central API Gateway at the edge for external authentication, rate limiting, and SSL termination, paired with Istio Envoy sidecars internally for service-to-service mTLS and automated distributed tracing."*
