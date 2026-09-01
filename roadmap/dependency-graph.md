# System Design Architecture Dependency Graph

This dependency graph maps how foundational concepts, communication primitives, storage engines, distributed coordination primitives, and specialized domain engines interconnect to form large-scale enterprise architectures.

---

## 🗺️ Master Architecture Dependency Graph

```mermaid
flowchart TD
    subgraph Layer1["1. Network & Ingress"]
        NET[Network Abstraction: TCP / HTTP / gRPC]
        DNS[DNS & GeoDNS]
        LB[Load Balancers: L4 vs L7]
        CDN[CDN Edge Caching]
        GW[API Gateway & Rate Limiter]
        NET --> DNS
        NET --> LB
        DNS --> CDN
        LB --> GW
        CDN --> GW
    end

    subgraph Layer2["2. Stateless Compute & Caching"]
        APP[Stateless Microservice Tier]
        L1C[L1 In-Memory Cache: Caffeine]
        L2C[L2 Distributed Cache: Redis Cluster]
        GW --> APP
        APP --> L1C
        L1C --> L2C
    end

    subgraph Layer3["3. Storage & Partitioning"]
        RDBMS[(Relational SQL: B-Tree / WAL)]
        NOSQL[(Key-Value / Wide-Column: LSM-Tree)]
        BLOB[(Blob / Object Store: S3)]
        SHARD[Sharding & Consistent Hashing]
        REPL[Replication & Quorum Consensus]
        APP --> RDBMS
        APP --> NOSQL
        APP --> BLOB
        RDBMS --> SHARD
        NOSQL --> SHARD
        SHARD --> REPL
    end

    subgraph Layer4["4. Asynchronous Messaging & Coordination"]
        MQ[Message Queue: RabbitMQ / SQS]
        KAFKA[Event Stream: Apache Kafka]
        DLOCK[Distributed Lock & Consensus: Raft / Redis]
        SCHED[Distributed Task Scheduler]
        APP --> MQ
        APP --> KAFKA
        APP --> DLOCK
        MQ --> SCHED
        KAFKA --> SCHED
    end

    subgraph Layer5["5. Specialized Domain Engines"]
        GEO[Geospatial Engine: Geohash / S2]
        SEARCH[Distributed Search: Elasticsearch / Inverted Index]
        FEED[Timeline & Fan-out Engine]
        PAY[Idempotent Payment & Ledger Engine]
        VEC[Vector Search: HNSW / FAISS]
        LLM[AI Inference Gateway & KV Cache]

        SHARD --> GEO
        NOSQL --> SEARCH
        L2C --> FEED
        KAFKA --> FEED
        DLOCK --> PAY
        RDBMS --> PAY
        BLOB --> VEC
        VEC --> LLM
    end

    subgraph Layer6["6. Telemetry & Resiliency"]
        OBS[Observability: Prometheus / OTel / ELK]
        CB[Resilience: Circuit Breaker & Bulkhead]
        GW --> CB
        APP --> CB
        APP --> OBS
        RDBMS --> OBS
        KAFKA --> OBS
    end
```

---

## 🔍 Domain-Specific Subsystem Flows

### A. Real-Time Geospatial Platform (Uber / Yelp)
```mermaid
flowchart LR
    Driver[Driver Mobile] --> WSGW[WebSocket Gateway]
    WSGW --> KAFKA[Kafka Location Stream]
    KAFKA --> GEO_WORKER[Geospatial Worker]
    GEO_WORKER --> S2_CACHE[Redis S2 Cell Cache]
    Rider[Rider App] --> API_GW[API Gateway]
    API_GW --> MATCH_SVC[Matching Service]
    MATCH_SVC --> S2_CACHE
```

### B. High-Volume Financial Payment Platform (Stripe)
```mermaid
flowchart LR
    Checkout[Client Checkout] --> API_GW[API Gateway]
    API_GW --> IDEM_FILTER[Idempotency Filter + Redis Lock]
    IDEM_FILTER --> PAY_SVC[Payment Processing Service]
    PAY_SVC --> LEDGER_DB[(Double-Entry SQL Ledger)]
    PAY_SVC --> BANK_GW[External Bank Gateway]
    PAY_SVC --> AUDIT_LOG[(Immutable Append-Only Audit Log)]
```

### C. Modern AI / LLM RAG & Inference Platform (ChatGPT / Cursor)
```mermaid
flowchart LR
    UserPrompt[User Prompt] --> AI_GW[AI Gateway & Semantic Cache]
    AI_GW --> EMBED_SVC[Embedding Service]
    EMBED_SVC --> VEC_DB[(Vector DB: HNSW Index)]
    VEC_DB --> RERANK[Re-ranking Model]
    RERANK --> CONTEXT_BUILDER[Context & Prompt Builder]
    CONTEXT_BUILDER --> LLM_CLUSTER[GPU Inference Cluster / vLLM]
    LLM_CLUSTER --> SSE_STREAM[SSE Token Stream to Client]
```
