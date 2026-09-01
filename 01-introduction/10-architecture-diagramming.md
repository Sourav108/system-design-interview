# Architecture Diagramming Standards

A whiteboard or digital architecture diagram is the visual contract between you and your interviewer. Clean, standardized diagrams prevent confusion and project senior architectural competence.

---

## 1. Visual Hierarchy & Standard Symbols

```mermaid
flowchart TD
    subgraph Clients["1. Client Layer"]
        Mobile["📱 Mobile Client (iOS / Android)"]
        Web["💻 Web Browser Client"]
    end

    subgraph Edge["2. Edge & Ingress Layer"]
        CDN["🌐 Global CDN (Static Assets / Media)"]
        LB["⚖️ Layer 7 Load Balancer (Envoy / NGINX)"]
        GW["🚪 API Gateway (Auth / Rate Limit)"]
    end

    subgraph Compute["3. Compute Tier (Stateless)"]
        SvcA["⚙️ Core API Service Cluster"]
        SvcB["⚙️ Ingestion Worker Cluster"]
    end

    subgraph CacheLayer["4. In-Memory Caching Tier"]
        RedisCluster[("⚡ Redis Distributed Cache (Cluster Mode)")]
    end

    subgraph StorageLayer["5. Persistent Storage Tier"]
        MasterDB[("🗄️ Primary Relational DB (PostgreSQL Leader)")]
        ReadReplica[("🗄️ Read Replica DB (PostgreSQL Follower)")]
        BlobStore[("🪣 Blob / Object Store (Amazon S3)")]
    end

    subgraph MessageLayer["6. Asynchronous Messaging Tier"]
        KafkaBus[("📬 Event Stream / Kafka Cluster")]
    end

    Clients --> Edge
    Edge --> Compute
    Compute --> CacheLayer
    Compute --> StorageLayer
    Compute --> MessageLayer
```

---

## 2. Best Practices for Diagramming in Interviews

1. **Separate Stateless Compute from Stateful Storage**: Always show application servers as stateless worker instances that can be scaled horizontally behind a load balancer.
2. **Explicit Storage Types**: Do not just write `"Database"`. Specify the exact paradigm: `PostgreSQL (ACID User Metadata)`, `Cassandra (Time-Series Activity Log)`, `S3 (Raw Video Files)`.
3. **Use Line Styles to Differentiate Synchronous vs Asynchronous Paths**:
   - **Solid Arrow (`-->`)**: Synchronous HTTP/REST or gRPC blocking call.
   - **Dashed Arrow (`-.->`)**: Asynchronous event publishing, CDC replication, or background worker task.
4. **Number the Execution Lifecycle**: Annotate arrows with sequence numbers ($1 	o 2 	o 3$) when explaining end-to-end request flows.
