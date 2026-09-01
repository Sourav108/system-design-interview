# System Design Interview Curriculum — Single Source of Truth

> **Master Index & Status Matrix**
> Tracks all modules, dependencies, building blocks, case studies, and Java implementations.
> Statuses: `TODO`, `IN PROGRESS`, `COMPLETE`.

---

## 📚 01. Introduction & Foundations

| ID | Topic | Status | Dependencies | Building Block | Case Studies | Implementation |
|---|---|:---:|---|---|---|---|
| `INT-01` | What is System Design? | `COMPLETE` | None | None | All | None |
| `INT-02` | System Design vs Low-Level Design (HLD vs LLD) | `COMPLETE` | `INT-01` | None | All | None |
| `INT-03` | System Design vs Coding Interviews | `COMPLETE` | `INT-01` | None | All | None |
| `INT-04` | How to Approach Ambiguous Problems | `COMPLETE` | `INT-01` | None | All | None |
| `INT-05` | How to Communicate Architecture | `COMPLETE` | `INT-04` | None | All | None |
| `INT-06` | How to Reason About Trade-Offs | `COMPLETE` | `INT-05` | None | All | None |
| `INT-07` | Architecture Diagramming Standards | `COMPLETE` | `INT-05` | None | All | None |

---

## 🎯 02. Interview Fundamentals (The DESIGN-FLOW Framework)

| ID | Topic | Status | Dependencies | Building Block | Case Studies | Implementation |
|---|---|:---:|---|---|---|---|
| `FND-01` | The DESIGN-FLOW 45-Minute Game Plan | `COMPLETE` | `INT-04` | All | All | None |
| `FND-02` | Step 1: Requirements Scoping | `COMPLETE` | `FND-01` | None | All | None |
| `FND-03` | Step 2 & 3: Constraints & Scale Estimation | `COMPLETE` | `FND-01` | `BB-01` to `BB-40` | All | None |
| `FND-04` | Step 4 & 5: High-Level Architecture & Data Model | `COMPLETE` | `FND-01` | `BB-03`, `BB-10` | All | None |
| `FND-05` | Step 6 & 7: Communication & Reliability | `COMPLETE` | `FND-01` | `BB-11`, `BB-13` | All | `IMP-01`, `IMP-04` |
| `FND-06` | Step 8, 9 & 10: Bottlenecks, Trade-Offs & Deep Dives | `COMPLETE` | `FND-01` | All | All | None |
| `FND-07` | Interview Pacing & Time Management | `COMPLETE` | `FND-01` | None | All | None |

---

## 🌐 03. Distributed Systems Fundamentals

| ID | Topic | Status | Dependencies | Building Block | Case Studies | Implementation |
|---|---|:---:|---|---|---|---|
| `DST-01` | Network Abstractions (TCP/UDP, HTTP/2, HTTP/3, gRPC) | `COMPLETE` | None | `BB-01`, `BB-02`, `BB-19` | All | `IMP-09` |
| `DST-02` | Partial Failure & Failure Models (Crash-Stop vs Byzantine) | `COMPLETE` | `DST-01` | `BB-21`, `BB-31` | All | `IMP-05` |
| `DST-03` | Distributed State Management | `COMPLETE` | `DST-02` | `BB-03`, `BB-04`, `BB-10` | All | `IMP-03` |
| `DST-04` | Replication Strategies (Leader-Follower, Multi-Leader, Quorum) | `COMPLETE` | `DST-03` | `BB-03`, `BB-04` | `CS-01`, `CS-06` | `IMP-03` |
| `DST-05` | Partitioning & Sharding (Consistent Hashing, Range, Hash) | `COMPLETE` | `DST-04` | `BB-04`, `BB-10`, `BB-18` | `CS-06`, `CS-09` | `IMP-02`, `IMP-03` |
| `DST-06` | Consistency Models (Linearizable, Sequential, Eventual, Causal) | `COMPLETE` | `DST-04` | `BB-03`, `BB-04` | `CS-11`, `CS-15` | `IMP-08` |
| `DST-07` | CAP & PACELC Theorems in Practice | `COMPLETE` | `DST-06` | `BB-03`, `BB-04`, `BB-10` | All | None |
| `DST-08` | Time, Clocks & Ordering (NTP, Lamport, Vector Clocks, TrueTime) | `COMPLETE` | `DST-06` | `BB-06`, `BB-24` | `CS-11`, `CS-13` | `IMP-02` |
| `DST-09` | Distributed Consensus (Paxos, Raft, Zab) | `COMPLETE` | `DST-08` | `BB-21`, `BB-31` | `CS-14`, `CS-15` | `IMP-05` |
| `DST-10` | Idempotency & Exactly-Once Semantics | `COMPLETE` | `DST-06` | `BB-36` | `CS-15` | `IMP-08` |

---

## ⚡ 04. Non-Functional Requirements (NFRs)

| ID | Topic | Status | Dependencies | Building Block | Case Studies | Implementation |
|---|---|:---:|---|---|---|---|
| `NFR-01` | Availability (99.9% to 99.999%, MTBF, MTTR, SLA/SLO/SLI) | `COMPLETE` | `DST-04` | `BB-02`, `BB-03` | All | None |
| `NFR-02` | Reliability & Fault Tolerance (Redundancy, Graceful Degradation) | `COMPLETE` | `NFR-01` | `BB-13`, `BB-21` | All | `IMP-10` |
| `NFR-03` | Scalability (Horizontal vs Vertical, Stateless vs Stateful) | `COMPLETE` | `DST-05` | `BB-02`, `BB-10` | All | `IMP-01` |
| `NFR-04` | Latency & Performance (p50, p90, p99, p99.9 tail latencies) | `COMPLETE` | `DST-01` | `BB-05`, `BB-10` | All | `IMP-03` |
| `NFR-05` | Throughput & Bandwidth (IOPS, QPS, Egress/Ingress Saturation) | `COMPLETE` | `NFR-04` | `BB-11`, `BB-14` | `CS-01`, `CS-08` | `IMP-04` |
| `NFR-06` | Durability & Data Loss Prevention (WAL, RAID, Multi-AZ) | `COMPLETE` | `DST-04` | `BB-03`, `BB-14` | `CS-15` | None |
| `NFR-07` | Observability (Metrics, Distributed Tracing, Structured Logging) | `COMPLETE` | None | `BB-07`, `BB-16`, `BB-29` | All | None |
| `NFR-08` | Security & Compliance (TLS, OAuth2, RBAC, Encryption At Rest) | `COMPLETE` | None | `BB-19`, `BB-25`, `BB-30` | All | `IMP-09` |

---

## 🧮 05. Capacity Estimation (Back-of-the-Envelope Math)

| ID | Topic | Status | Dependencies | Building Block | Case Studies | Implementation |
|---|---|:---:|---|---|---|---|
| `CAP-01` | Powers of Two & Latency Numbers Every Engineer Should Know | `COMPLETE` | None | None | All | None |
| `CAP-02` | Traffic & QPS Estimation (Read/Write Ratios, Peak Multipliers) | `COMPLETE` | `CAP-01` | `BB-02`, `BB-13` | All | `IMP-01` |
| `CAP-03` | Storage Capacity & Growth Trajectory (5-Year Projections) | `COMPLETE` | `CAP-01` | `BB-03`, `BB-14` | All | None |
| `CAP-04` | Bandwidth & Network Throughput Estimation | `COMPLETE` | `CAP-02` | `BB-05`, `BB-14` | `CS-01`, `CS-08` | None |
| `CAP-05` | Memory & Distributed Cache Sizing (80-20 Pareto Rule) | `COMPLETE` | `CAP-02` | `BB-10` | All | `IMP-03` |
| `CAP-06` | Server Cluster & Core Count Estimation | `COMPLETE` | `CAP-02` | `BB-02`, `BB-19` | All | None |

---

## 🧱 06. System Design Building Blocks (40 Reusable Components)

| ID | Topic | Status | Dependencies | Reusability | Case Studies | Implementation |
|---|---|:---:|---|---|---|---|
| `BB-01` | DNS (Domain Name System & Geo-Routing) | `COMPLETE` | `DST-01` | Ingress | All | None |
| `BB-02` | Load Balancer (L4 vs L7, Algorithms, Health Checks) | `COMPLETE` | `DST-01` | Ingress | All | `IMP-09` |
| `BB-03` | Distributed Database (Relational, SQL Engines, ACID, WAL) | `COMPLETE` | `DST-04` | Storage | All | None |
| `BB-04` | Distributed Key-Value Store (LSM-Trees vs B-Trees) | `COMPLETE` | `DST-05` | Storage | `CS-09`, `CS-10` | `IMP-02` |
| `BB-05` | CDN (Content Delivery Network & Edge Caching) | `COMPLETE` | `DST-01` | Edge | `CS-01`, `CS-08` | None |
| `BB-06` | Distributed ID Generator / Sequencer (Snowflake, Ticket Server) | `COMPLETE` | `DST-08` | Invariant | `CS-06`, `CS-09` | `IMP-02` |
| `BB-07` | Distributed Monitoring & Alerting Engine | `COMPLETE` | `NFR-07` | Telemetry | All | None |
| `BB-08` | Server-Side Error Monitoring & Exception Aggregator | `COMPLETE` | `NFR-07` | Telemetry | All | None |
| `BB-09` | Client-Side Error Monitoring & Crash Reporter | `COMPLETE` | `NFR-07` | Telemetry | `CS-01`, `CS-08` | None |
| `BB-10` | Distributed Cache (Cache-Aside, Write-Through, LRU/LFU, Redis) | `COMPLETE` | `DST-05` | Caching | All | `IMP-03` |
| `BB-11` | Distributed Message Queue (RabbitMQ, SQS, Backpressure) | `COMPLETE` | `DST-03` | Messaging | `CS-10`, `CS-14` | None |
| `BB-12` | Distributed Pub/Sub (Kafka, Event Streams, Log Compaction) | `COMPLETE` | `DST-04` | Messaging | `CS-06`, `CS-07` | `IMP-04` |
| `BB-13` | Distributed Rate Limiter (Token Bucket, Leaky Bucket, Sliding Log)| `TODO`| `BB-10` | Reliability | All | `IMP-01` |
| `BB-14` | Distributed Blob / Object Store (S3, Chunking, Erasure Coding) | `COMPLETE` | `DST-04` | Storage | `CS-01`, `CS-08` | None |
| `BB-15` | Distributed Search (Inverted Index, Elasticsearch, Lucene) | `COMPLETE` | `DST-05` | Search | `CS-02`, `CS-04` | None |
| `BB-16` | Distributed Logging Pipeline (ELK, Vector, Kafka Log Ingestion) | `COMPLETE` | `BB-12` | Telemetry | All | None |
| `BB-17` | Distributed Task Scheduler (Quartz, Priority Queues, Delay Queue)| `TODO`| `BB-10` | Compute | `CS-10`, `CS-16` | `IMP-06` |
| `BB-18` | Sharded Counter (Distributed Increment, Write Splitting) | `COMPLETE` | `DST-05` | Primitive | `CS-01`, `CS-06` | None |
| `BB-19` | API Gateway (Authentication, SSL Termination, Routing, Kong) | `COMPLETE` | `BB-02`, `BB-13` | Ingress | All | `IMP-09` |
| `BB-20` | Service Discovery (Consul, Eureka, Heartbeats, Gossip) | `COMPLETE` | `DST-09` | Routing | All | None |
| `BB-21` | Distributed Lock Manager (Redlock, Chubby, Zookeeper Fencing) | `COMPLETE` | `DST-09` | Coordination | `CS-15`, `CS-16` | `IMP-05` |
| `BB-22` | Distributed Configuration Service (etcd, Spring Cloud Config) | `COMPLETE` | `DST-09` | Coordination | All | None |
| `BB-23` | Notification Service (Push, SMS, Email Fan-out Engine) | `COMPLETE` | `BB-11` | Messaging | `CS-06`, `CS-11` | `IMP-07` |
| `BB-24` | Unique ID Generator (Base62, ULID, UUIDv7) | `COMPLETE` | `BB-06` | Primitive | `CS-09` | `IMP-02` |
| `BB-25` | Token & Authentication Service (JWT, PASETO, OAuth2, Session) | `COMPLETE` | `BB-10` | Security | All | None |
| `BB-26` | Feature Flagging Service (Flipt, Unleash, Dynamic Rollouts) | `COMPLETE` | `BB-22` | Operations | All | None |
| `BB-27` | Distributed Job Queue (Dead Letter Queue, Retries, Exponential) | `COMPLETE` | `BB-11` | Compute | `CS-14`, `CS-16` | `IMP-06` |
| `BB-28` | Enterprise Event Bus (Schema Registry, CloudEvents) | `COMPLETE` | `BB-12` | Messaging | `CS-15` | `IMP-04` |
| `BB-29` | Metrics Aggregation Pipeline (Prometheus, OpenTelemetry, StatsD) | `TODO`| `DST-05` | Telemetry | All | None |
| `BB-30` | Immutable Audit Log (Append-Only Log, Cryptographic Verification) | `TODO`| `BB-14` | Security | `CS-15` | None |
| `BB-31` | Leader Election Service (Raft, Bully Algorithm, Epoch Fencing) | `COMPLETE` | `DST-09` | Coordination | `CS-14` | `IMP-05` |
| `BB-32` | Geospatial Indexing (Geohash, Quadtree, Google S2) | `COMPLETE` | `DST-05` | Indexing | `CS-03`, `CS-04`, `CS-05` | None |
| `BB-33` | Timeline & Feed Generation Engine (Push vs Pull, Fan-out on Write) | `TODO`| `BB-10`, `BB-12` | Feeds | `CS-06`, `CS-07` | None |
| `BB-34` | Real-Time Recommendation Pipeline (Collaborative Filtering, Graph) | `TODO`| `BB-12` | Compute | `CS-01`, `CS-07` | None |
| `BB-35` | File Synchronization Engine (Rsync, Chunk Hashes, Merkle Trees)| `TODO` | `BB-14` | Sync | `CS-13` | None |
| `BB-36` | Payment Idempotency Engine (Unique Token Keys, Distributed Locks)| `TODO` | `BB-21` | Financial | `CS-15` | `IMP-08` |
| `BB-37` | WebSocket Gateway (Stateful Connection Management, Redis PubSub)| `TODO` | `BB-12` | Real-Time | `CS-11`, `CS-13` | None |
| `BB-38` | Real-Time Stream Processing Pipeline (Flink, Spark Streaming) | `COMPLETE` | `BB-12` | Streaming | `CS-01`, `CS-06` | None |
| `BB-39` | Vector Search Index (HNSW, IVF, Pinecone, FAISS, Cosine Distance) | `TODO`| `DST-05` | AI/ML | `CS-17`, `CS-19`, `CS-20` | None |
| `BB-40` | AI Inference Gateway (Model Routing, Prompt Caching, Token Metering)| `TODO`| `BB-19` | AI/ML | `CS-17`, `CS-19`, `CS-20` | None |

---

## 🏗️ 07. End-to-End Case Studies (20 Real-World Systems)

| ID | Case Study | Status | Key Focus Area | Primary Building Blocks |
|---|---|:---:|---|---|
| `CS-01` | YouTube / Netflix Video Streaming Platform | `COMPLETE` | Adaptive Bitrate (HLS/DASH), Chunking, CDN, Transcoding Pipeline | `BB-05`, `BB-14`, `BB-38` |
| `CS-02` | Quora / StackOverflow Knowledge Platform | `COMPLETE` | Search, Ranking, Distributed Caching, Content Moderation | `BB-10`, `BB-15`, `BB-23` |
| `CS-03` | Google Maps / Navigation Engine | `COMPLETE` | Graph Routing (A* / Dijkstra), Geospatial Tiles, S2 Cells | `BB-32`, `BB-05` |
| `CS-04` | Yelp / Proximity Search Service | `COMPLETE` | Spatial Querying, Geohash, Quadtree Sharding, Cache-Aside | `BB-32`, `BB-10`, `BB-15` |
| `CS-05` | Uber / Lyft Ride Hailing Platform | `COMPLETE` | Real-Time Driver Matching, Geospatial Tracking, WebSocket, Surge Pricing | `BB-32`, `BB-37`, `BB-12` |
| `CS-06` | Twitter / X Social Network | `COMPLETE` | Fan-out on Write vs Read, Timeline Cache, Snowflake Sequencer | `BB-06`, `BB-10`, `BB-33` |
| `CS-07` | Newsfeed Aggregator | `COMPLETE` | Activity Graphs, Hybrid Fan-out, Ranking Pipeline, Cache Invalidation | `BB-10`, `BB-33`, `BB-34` |
| `CS-08` | Instagram / Flickr Photo Sharing Platform | `COMPLETE` | Image Ingestion, Thumbnail Generation, Object Storage, Global CDN | `BB-05`, `BB-14`, `BB-10` |
| `CS-09` | TinyURL / URL Shortening Service | `COMPLETE` | Base62 Encoding, MD5/MurmurHash, Distributed Sequencer, Cache Sizing | `BB-04`, `BB-06`, `BB-24` |
| `CS-10` | Distributed Web Crawler | `COMPLETE` | Politeness, Frontier Queue, DNS Caching, Duplicate Elimination (Bloom) | `BB-01`, `BB-11`, `BB-17` |
| `CS-11` | WhatsApp / Discord Real-Time Chat Platform | `COMPLETE` | WebSocket Connections, Presence Service, Mnesia/Cassandra Chat Store | `BB-37`, `BB-04`, `BB-23` |
| `CS-12` | Typeahead / Search Autocomplete System | `COMPLETE` | Trie Data Structure, Frequency Aggregation, Top-K Heap, CDN Edge | `BB-10`, `BB-15`, `BB-05` |
| `CS-13` | Google Docs / Collaborative Document Editor | `COMPLETE` | Operational Transformation (OT) vs CRDTs, WebSocket, State Sync | `BB-37`, `BB-35` |
| `CS-14` | Distributed Deployment & CI/CD Pipeline | `COMPLETE` | Canary Rollouts, Artifact Distribution, Task Graphs, Orchestration | `BB-17`, `BB-27`, `BB-31` |
| `CS-15` | Stripe / PayPal Payment Gateway & Ledger | `COMPLETE` | Double-Entry Bookkeeping, Idempotency, 2PC/Saga, Distributed Lock | `BB-21`, `BB-30`, `BB-36` |
| `CS-16` | LeetCode / HackerRank Online Coding Platform | `COMPLETE` | Sandboxed Code Execution (Docker/gVisor), Isolation, Task Scheduler | `BB-17`, `BB-27`, `BB-21` |
| `CS-17` | ChatGPT Conversational AI Platform | `COMPLETE` | SSE Streaming Tokens, Context Buffer, Model Gateway, Vector Retrieval | `BB-39`, `BB-40`, `BB-37` |
| `CS-18` | Distributed Big Data Lakehouse Platform | `COMPLETE` | Parquet Storage, Compaction, Metastore, Partition Pruning | `BB-14`, `BB-38` |
| `CS-19` | LLM-Powered Customer Support Platform | `COMPLETE` | Multi-Turn Conversation State, Tool Calling, Vector Search, Fallbacks | `BB-39`, `BB-40`, `BB-10` |
| `CS-20` | Enterprise AI Coding Assistant (Cursor / Copilot) | `COMPLETE` | AST Chunking, Context Slicing, Real-Time Inference, Prompt Cache | `BB-39`, `BB-40` |

---

## 💥 08. System Failure Post-Mortems (Production Outage Scenarios)

| ID | Failure Mode | Status | Trigger & Mechanics | Mitigation & Prevention |
|---|---|:---:|---|---|
| `FL-01` | Cascading Failure Across Microservices | `TODO` | Downstream Latency Spike $	o$ Thread Exhaustion $	o$ Cluster Crash | Bulkheading, Circuit Breakers, Adaptive Drop |
| `FL-02` | Client Retry Storms & Avalanche Effects | `TODO` | Brief Outage $	o$ Aggressive Unjittered Retries $	o$ Saturation | Exponential Backoff with Full Jitter, Token Bucket |
| `FL-03` | Thundering Herd & Cache Stampede | `TODO` | Hot Key Expiration $	o$ 10,000 DB Queries Simultaneously | Mutex Locking (Singleflight), Probabilistic Early Expiration (XFetch) |
| `FL-04` | Hot Key & Hot Partition Saturation | `TODO` | Celebrity Account Post $	o$ Single Node IOPS / Bandwidth Overload | Salted Keys, Local Application Caching, Read Splitting |
| `FL-05` | Distributed Split-Brain in Consensus Clusters | `TODO` | Network Partition $	o$ Two Independent Leaders Elected | Strict Quorum ($N/2 + 1$), Epoch Generation Fencing |
| `FL-06` | Network Partition & Asymmetric Disconnections | `TODO` | Node can send but not receive (unidirectional network drop) | Heartbeat Quorums, TCP Keepalive, Lease Timeouts |
| `FL-07` | Stale Data & Replication Lag Spike | `TODO` | Heavy Write Surge $	o$ Read Replicas Lag Behind Primary | Read-Your-Writes Consistency, Version Vectors, Monotonic Reads |
| `FL-08` | Duplicate Event Ingestion & Replay Havoc | `TODO` | Consumer Crash Before Offset Commit $	o$ Re-processing | Unique Transaction ID, DB Idempotency Key Constraint |
| `FL-09` | Silent Message Loss in Distributed Queues | `TODO` | In-Memory Buffer Overflow $	o$ Dropped Messages Without Error | Persistent Commit Logs, High Watermark Acks, Dead Letter Queues |
| `FL-10` | Out-of-Order Event Processing | `TODO` | Multi-Partition Ingestion $	o$ State Updates Applied In Reverse | Sequence Numbers, Per-Entity Partition Keying, State Versioning |
| `FL-11` | Clock Drift & Leap Second Distributed Glitches | `TODO` | NTP Step $	o$ Negative Time Durations $	o$ Lock Expiration Bugs | TrueTime Uncertainty Intervals, Monotonic Clocks (`CLOCK_MONOTONIC`) |
| `FL-12` | Unbounded Queue Backlog & Memory OOM | `TODO` | Producer Throughput $> $ Consumer Capacity $	o$ Heap Collapse | Bounded Queues, Drop-Oldest / Drop-Newest, Upstream Rate Limit |
| `FL-13` | Database Connection Pool Starvation | `TODO` | Slow Query in Transaction Block $	o$ Pool Saturated $	o$ 500 Spike | Connection Pool Sizing Math, Strict Transaction Timeouts |
| `FL-14` | Multi-Region Active-Active Split & Sync Collisions | `TODO` | Simultaneous Writes in US and EU $	o$ Merge Conflict | Conflict-Free Replicated Data Types (CRDTs), Last-Write-Wins (LWW) |
| `FL-15` | Bad Deployment & Blast Radius Expansion | `TODO` | Faulty Config Pushed Globally $	o$ All Regions Down Simultaneously | Canary Deployment Pipeline, Automated Rollback, Staged Cell Rollouts |

---

## 💻 09. Production-Grade Java Implementations

| ID | Implementation Project | Status | Tech Stack | Key Architectural Features |
|---|---|:---:|---|---|
| `IMP-01` | Distributed Rate Limiter Service | `TODO` | Java 21, Spring Boot, Redis, Lua | Token Bucket + Sliding Window Counter with atomic Lua scripts |
| `IMP-02` | Distributed Unique ID & URL Shortener | `TODO` | Java 21, Spring Boot, PostgreSQL, Redis | Snowflake 64-bit ID generator + Base62 URL shortener |
| `IMP-03` | Multi-Tier Distributed Cache Engine | `TODO` | Java 21, Spring Boot, Caffeine, Redis | L1 (In-Memory) + L2 (Redis) Cache-Aside with Singleflight |
| `IMP-04` | High-Throughput Event Streaming Service | `TODO` | Java 21, Spring Boot, Apache Kafka | Idempotent Producer + Exactly-Once Consumer with DLT |
| `IMP-05` | Distributed Lock Manager & Leader Election | `TODO` | Java 21, Spring Boot, Redis, Zookeeper | Redlock algorithm with fencing tokens and TTL renewers |
| `IMP-06` | Distributed Task Scheduler & Job Queue | `TODO` | Java 21, Spring Boot, PostgreSQL, Redis | Priority Delay Queue + Distributed Worker Stealing |
| `IMP-07` | Multi-Channel Notification Fan-Out Engine | `TODO` | Java 21, Spring Boot, Kafka, Redis | Async Fan-Out with Priority Queues and Provider Fallbacks |
| `IMP-08` | Financial Payment Idempotency Engine | `TODO` | Java 21, Spring Boot, PostgreSQL | Atomic Idempotency Filter + Double-Entry Ledger Transaction |
| `IMP-09` | API Gateway & Reverse Proxy Simulator | `TODO` | Java 21, Spring Boot WebFlux, Netty | Non-blocking reverse proxy with Auth, Rate Limiting, Routing |
| `IMP-10` | Circuit Breaker & Resiliency Simulator | `TODO` | Java 21, Spring Boot, Resilience4j | Closed $	o$ Open $	o$ Half-Open state machine with chaos test |

---

## 🤖 12. Modern AI System Design (LLM Infrastructure)

| ID | Module | Status | Core Architecture | Key Components |
|---|---|:---:|---|---|
| `AI-01` | LLM Inference Serving Architecture | `COMPLETE` | Continuous Batching, KV Cache Management, PagedAttention | vLLM, TensorRT-LLM, GPU Cluster |
| `AI-02` | Streaming Token Responses & Server-Sent Events | `COMPLETE` | Asynchronous Token Yielding, Chunked HTTP/2, Backpressure | SSE, WebSocket, Netty |
| `AI-03` | Multi-Turn Conversation & State Management | `COMPLETE` | Sliding Window Memory, Summary Buffers, Token Truncation | Redis, DynamoDB, Postgres |
| `AI-04` | AI API Gateway & Intelligent Model Routing | `COMPLETE` | Semantic Caching, Multi-Provider Fallbacks, Token Rate Limiting | LiteLLM, Cloudflare AI Gateway |
| `AI-05` | GPU Capacity Planning & Token Capacity Math | `COMPLETE` | Parameters, FP16/INT8/FP4 VRAM Sizing, FLOPs Calculation | GPU Memory Math, Cost Optimization |
| `AI-06` | Production Retrieval-Augmented Generation (RAG) | `COMPLETE` | Multi-Stage Retrieval, Re-ranking, Dense/Sparse Hybrid Search | HNSW Vector Index, Cross-Encoder |
| `AI-07` | Vector Search Engine & Embedding Pipelines | `COMPLETE` | Asynchronous Embedding Ingestion, HNSW Indexing, FAISS/Qdrant | Vector DB, Chunking Strategies |
| `AI-08` | AI Observability, Evals & Guardrails | `COMPLETE` | Token Cost Telemetry, Hallucination Detection, Moderation Guardrails | Langfuse, TruLens, OpenTelemetry |
| `AI-09` | Enterprise AI Coding Assistant Architecture | `COMPLETE` | Repo Graph Indexing, Context Slicing, Low-Latency Auto-complete | AST Parser, Fast Fill-In-The-Middle |
| `AI-10` | Autonomous LLM Customer Support Agent | `COMPLETE` | Tool Calling Engine, Human-In-The-Loop Handoff, Policy Guardrails | ReAct Agent, Workflow Engine |

---

## 🎯 10. Interview Practice & Mock Rounds

| Category | File / Resource | Status | Content Overview |
|---|---|:---:|---|
| **SDE1** | `10-interview-practice/sde1/` | `TODO` | Core fundamentals, basic scaling, load balancing, relational schemas. |
| **SDE2** | `10-interview-practice/sde2/` | `TODO` | Primary target: Microservice partitioning, caching, Kafka, 20 core problems. |
| **Senior** | `10-interview-practice/senior/` | `TODO` | Multi-region, consensus, complex failure modes, cost & capacity planning. |
| **Mock Interviews** | `10-interview-practice/mock-interviews/` | `TODO` | 10 complete transcript mock interviews with scoring rubrics. |
| **Questions Bank** | `10-interview-practice/` | `TODO` | 100 System Design Questions, 30 Estimations, 30 Trade-offs, 20 Failures. |

---

## 📋 11. Cheat Sheets (Last-Minute Revision)

| Cheat Sheet | File | Status | Core Focus |
|---|---|:---:|---|
| Database Selection | `11-cheatsheets/database-selection.md` | `TODO` | SQL vs NoSQL vs NewSQL vs Vector vs Time-Series decision tree |
| Distributed Caching | `11-cheatsheets/caching.md` | `TODO` | Cache-aside, write-through, eviction, stampede prevention |
| Load Balancing | `11-cheatsheets/load-balancing.md` | `TODO` | L4 vs L7, hashing, health checks, round-robin, least connections |
| Messaging & Queues | `11-cheatsheets/messaging.md` | `TODO` | Push vs Pull, at-least-once vs exactly-once, backpressure |
| Apache Kafka | `11-cheatsheets/kafka.md` | `TODO` | Partitions, consumer groups, replication, offsets, log compaction |
| Consistency Models | `11-cheatsheets/consistency.md` | `TODO` | Linearizable, sequential, eventual, causal, read-your-writes |
| Replication Strategies | `11-cheatsheets/replication.md` | `TODO` | Leader-follower, multi-leader, quorum $R + W > N$ |
| Partitioning & Sharding | `11-cheatsheets/partitioning.md` | `TODO` | Consistent hashing, range-based, virtual nodes, rebalancing |
| Reliability & Resiliency | `11-cheatsheets/reliability.md` | `TODO` | Circuit breakers, retries with jitter, bulkheads, rate limiters |
| Observability & Metrics | `11-cheatsheets/observability.md` | `TODO` | RED method, USE method, distributed traces, metrics vs logs |
| Capacity Estimation Math | `11-cheatsheets/capacity-estimation.md` | `TODO` | Numbers to memorize, QPS to bandwidth to storage conversion cheatsheet |
| The 45-Min Interview | `11-cheatsheets/system-design-interview.md` | `TODO` | DESIGN-FLOW cheat sheet, time allocations, checklist |
| Modern AI Systems | `11-cheatsheets/ai-system-design.md` | `TODO` | LLM inference, VRAM math, RAG architecture, vector search |
