# Distributed State Management

Managing application state across multiple machines determines whether a system can scale horizontally and recover from catastrophic node failures.

---

## 1. Stateless vs Stateful Tier Decomposition

```mermaid
flowchart TD
    subgraph StatelessCompute["Stateless Compute Tier (Horizontally Scalable)"]
        GW[API Gateway] --> App1[App Server 1]
        GW --> App2[App Server 2]
        GW --> App3[App Server 3]
    end

    subgraph SharedState["State Tier (Persistent & In-Memory)"]
        App1 --> SessionStore[(Redis Session Cache)]
        App2 --> SessionStore
        App3 --> SessionStore
        App1 --> PrimaryDB[(PostgreSQL Primary)]
        App2 --> PrimaryDB
        App3 --> PrimaryDB
    end
```

- **Stateless Tier**: Nodes hold zero in-memory client state between requests. Any request can be routed to any instance. Scale-out is trivial: add instances behind a Load Balancer.
- **Stateful Tier**: Nodes hold mutable state (databases, in-memory caches, message queues). Scaling requires **partitioning**, **replication**, and **distributed concurrency control**.

---

## 2. Write-Ahead Logging (WAL) for Crash Recovery

To guarantee durability ($D$ in ACID) without performing slow random disk writes on every transaction:

```mermaid
flowchart LR
    WriteReq[Client Write] --> AppendWAL[1. Append to Sequential WAL on Disk]
    AppendWAL --> InMemBuffer[2. Update In-Memory Buffer / MemTable]
    InMemBuffer --> Ack[3. Acknowledge Success to Client]
    InMemBuffer -. Async Flush .-> DataFiles[(4. Background Flush to Data Files / SSTables)]
```

If the server crashes at step 3, upon reboot it replays the sequential WAL from the last checkpoint, restoring memory state with zero data loss.
