# Step 4: High-Level Architecture

Step 4 is where you establish the end-to-end macro architecture. The key is to start with a clean separation of concerns before diving into complex database internals.

---

## 1. Standard 5-Tier Architecture

```mermaid
flowchart TD
    subgraph Tier1["Tier 1: Client & Edge"]
        Client[Client App] --> DNS[Route 53 / GeoDNS]
        DNS --> CDN[Cloudflare CDN]
        Client --> LB[L7 Envoy Load Balancer]
    end

    subgraph Tier2["Tier 2: Ingress & Gateway"]
        LB --> APIGW[API Gateway / Spring Cloud Gateway]
        APIGW --> Auth[Auth & Rate Limiter Filter]
    end

    subgraph Tier3["Tier 3: Stateless Compute"]
        Auth --> ServiceA[Order Service Cluster]
        Auth --> ServiceB[User Service Cluster]
        Auth --> ServiceC[Catalog Service Cluster]
    end

    subgraph Tier4["Tier 4: Caching & State"]
        ServiceA --> RedisCache[(Redis Distributed Cache)]
        ServiceA --> PrimaryDB[(PostgreSQL Primary / Leader)]
        PrimaryDB -. Replication .-> ReadReplica[(Read Replica)]
    end

    subgraph Tier5["Tier 5: Asynchronous Event Bus"]
        ServiceA --> KafkaTopic[(Kafka Event Stream)]
        KafkaTopic --> WorkerA[Inventory Worker]
        KafkaTopic --> WorkerB[Email Notification Worker]
    end
```

---

## 2. Essential Architectural Guidelines

1. **Stateless Compute**: Application servers must store zero user session state in local memory. All state resides in distributed caches (Redis) or persistent databases.
2. **Decouple Synchronous from Asynchronous Paths**:
   - Synchronous (Blocking): User-facing CRUD operations requiring immediate responses ($< 100	ext{ms}$).
   - Asynchronous (Non-Blocking): Notifications, search index updates, heavy analytics, batch transcoding.
3. **Trace the End-to-End Flow**: Walk through a complete request lifecycle with numbered steps ($1 	o 2 	o 3 	o 4$) to prove component integration.
