# Module 06: System Design Building Blocks

The comprehensive library of **40 reusable distributed system infrastructure building blocks**, each structured according to the master **29-section architecture template**.

---

## 🧱 Catalog of 40 Reusable Building Blocks

| ID | Building Block Name | Primary Domain | Core Technology / Pattern |
|---|---|---|---|
| [**01**](./01-dns.md) | **DNS (Domain Name System & GeoDNS)** | Ingress | Anycast BGP Routing, GeoDNS, TTL Caching |
| [**02**](./02-load-balancer.md) | **Load Balancer (L4 vs L7 & Reverse Proxy)** | Ingress | Envoy, NGINX, Maglev Hashing, Health Checks |
| [**03**](./03-distributed-database.md) | **Distributed Database (Relational & ACID)** | Storage | PostgreSQL, Spanner, WAL, MVCC, 2PC |
| [**04**](./04-distributed-key-value-store.md) | **Distributed Key-Value Store (LSM-Tree)** | Storage | Cassandra, DynamoDB, MemTable, SSTables, Bloom |
| [**05**](./05-cdn.md) | **CDN (Content Delivery Network & Edge Caching)** | Edge | Cloudflare, CloudFront, PoPs, Origin Shielding |
| [**06**](./06-distributed-id-sequencer.md) | **Distributed ID Generator / Sequencer** | Invariant | Twitter Snowflake, 64-bit Bitwise Layout, Clock Drift |
| [**07**](./07-distributed-monitoring.md) | **Distributed Monitoring & Alerting Engine** | Telemetry | Prometheus, TSDB, Gorilla Compression, Alertmanager |
| [**08**](./08-server-side-error-monitoring.md) | **Server-Side Error Monitoring & Exception Aggregator** | Telemetry | Sentry, Stack Normalization, Signature Fingerprinting |
| [**09**](./09-client-side-error-monitoring.md) | **Client-Side Error Monitoring & Crash Reporter** | Telemetry | Minidumps, Symbolication, dSYM, ProGuard Mapping |
| [**10**](./10-distributed-cache.md) | **Distributed Cache (Redis & Multi-Tier)** | Caching | Redis Cluster, Cache-Aside, Singleflight, XFetch |
| [**11**](./11-distributed-messaging-queue.md) | **Distributed Messaging Queue (RabbitMQ & SQS)** | Messaging | RabbitMQ, AMQP, Prefetch Limits, Backpressure, DLQ |
| [**12**](./12-distributed-pub-sub.md) | **Distributed Pub/Sub (Apache Kafka)** | Messaging | Kafka Commit Log, Partitioning, ISR, Consumer Groups |
| [**13**](./13-distributed-rate-limiter.md) | **Distributed Rate Limiter** | Reliability | Token Bucket, Sliding Window Counter, Redis Lua |
| [**14**](./14-distributed-blob-object-store.md) | **Distributed Blob / Object Store (Amazon S3)** | Storage | S3 Architecture, Erasure Coding, Multipart Upload |
| [**15**](./15-distributed-search.md) | **Distributed Search (Elasticsearch & Inverted Index)** | Search | Elasticsearch, Lucene Inverted Index, BM25 Scoring |
| [**16**](./16-distributed-logging-pipeline.md) | **Distributed Logging Pipeline (ELK & Vector)** | Telemetry | Vector, FluentBit, Kafka Buffer, Index Lifecycle (ILM) |
| [**17**](./17-distributed-task-scheduler.md) | **Distributed Task Scheduler** | Compute | Timing Wheels, Priority Delay Queue, Redis ZSET |
| [**18**](./18-sharded-counter.md) | **Sharded Counter (Distributed High-Write Counter)** | Primitive | Write Dispersion across $N$ slots, Parallel Aggregation |
| [**19**](./19-api-gateway.md) | **API Gateway & Reverse Proxy** | Ingress | Kong, Spring Cloud Gateway, JWT Auth, Non-blocking Netty |
| [**20**](./20-service-discovery.md) | **Service Discovery & Registry (Consul & Eureka)** | Routing | Consul, Eureka, SWIM Gossip Protocol, Heartbeats |
| [**21**](./21-distributed-lock-manager.md) | **Distributed Lock Manager (Redlock & Zookeeper)** | Coordination | Redlock, Zookeeper Ephemeral Nodes, Fencing Tokens |
| [**22**](./22-distributed-configuration-service.md) | **Distributed Configuration Service (etcd & Consul)** | Coordination | etcd, Raft Consensus, gRPC Watchers, Dynamic Reload |
| [**23**](./23-notification-service.md) | **Multi-Channel Notification Service** | Messaging | APNs, FCM, Twilio, Priority Queues, Vendor Fallbacks |
| [**24**](./24-unique-id-generator.md) | **Unique ID Generator (Base62 & UUIDv7)** | Primitive | Base62 Bijective Mapping, UUIDv7, URL-Safety |
| [**25**](./25-token-and-auth-service.md) | **Token & Authentication Service (OAuth2 & JWT)** | Security | OAuth2.0, RS256 Asymmetric JWTs, JWKS Public Keys |
| [**26**](./26-feature-flag-service.md) | **Feature Flagging Service (Flipt & Unleash)** | Operations | MurmurHash3 Percentage Bucketing, In-Memory SDK |
| [**27**](./27-distributed-job-queue.md) | **Distributed Job Queue (Work Stealing & Retries)** | Compute | Work Stealing, Visibility Timeout, Exponential Jitter |
| [**28**](./28-enterprise-event-bus.md) | **Enterprise Event Bus (Schema Registry & CloudEvents)** | Messaging | Schema Registry, Avro Binary Format, Schema Evolution |
| [**29**](./29-metrics-aggregation-pipeline.md) | **Metrics Aggregation Pipeline (OTel & StatsD)** | Telemetry | OpenTelemetry Collector, T-Digest Quantiles, UDP Sockets |
| [**30**](./30-immutable-audit-log.md) | **Immutable Audit Log (Append-Only & Merkle Tree)** | Security | SHA-256 Hash Chains, Merkle Trees, S3 Object Lock |
| [**31**](./31-leader-election-service.md) | **Leader Election Service (Raft & Bully Algorithm)** | Coordination | Raft Leader Election, Term Fencing, Majority Quorum |
| [**32**](./32-geospatial-indexing.md) | **Geospatial Indexing (Geohash, Quadtree & S2)** | Indexing | Google S2 Hilbert Curves, Geohash, 9-Neighbor Query |
| [**33**](./33-timeline-and-feed-generator.md) | **Timeline & Feed Generation Engine** | Feeds | Hybrid Fan-Out, Redis Sorted Sets, Celebrity Caching |
| [**34**](./34-recommendation-pipeline.md) | **Real-Time Recommendation Pipeline** | Compute | Two-Tower Embeddings, ANN Retrieval, Real-Time Features |
| [**35**](./35-file-synchronization-engine.md) | **File Synchronization Engine (Rsync & Merkle Trees)** | Sync | FastCDC Content-Defined Chunking, Merkle Tree Diff |
| [**36**](./36-payment-idempotency-engine.md) | **Payment Idempotency & Financial Ledger Engine** | Financial | Idempotency Keys, Redis Locks, Double-Entry Bookkeeping |
| [**37**](./37-websocket-gateway.md) | **WebSocket Gateway (Stateful Connection Manager)** | Real-Time | Netty Full-Duplex TCP, Redis Pub/Sub Backplane |
| [**38**](./38-streaming-processing-pipeline.md) | **Real-Time Stream Processing (Apache Flink)** | Streaming | Apache Flink, Watermarks, RocksDB Checkpoints, CEP |
| [**39**](./39-vector-search-index.md) | **Vector Search Index (HNSW & Embeddings)** | AI/ML | HNSW Multi-Layer Graphs, Cosine Similarity, Milvus |
| [**40**](./40-ai-inference-gateway.md) | **AI Inference Gateway & Model Router** | AI/ML | Semantic Prompt Caching, Token Rate Limiting, vLLM |
