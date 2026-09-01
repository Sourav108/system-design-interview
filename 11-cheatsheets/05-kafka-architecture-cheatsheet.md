# Apache Kafka Architecture Cheat Sheet

```mermaid
flowchart LR
    Producer -->|hash key % 3| Partition0["Partition 0 [0][1][2][3]..."]
    Producer --> Partition1["Partition 1 [0][1][2]..."]
    Producer --> Partition2["Partition 2 [0][1]..."]

    Partition0 --> ConsumerA[Consumer 1 in Group A]
    Partition1 --> ConsumerB[Consumer 2 in Group A]
    Partition2 --> ConsumerC[Consumer 3 in Group A]
```

## Key Invariants
- **Partitions**: Unit of parallelism in Kafka. Total active consumers in a consumer group $\le$ number of topic partitions.
- **Ordering**: Kafka guarantees strict chronological message order **only within a single partition**, not across partitions.
- **In-Sync Replicas (ISR)**: Follower replicas that are caught up with the leader. Set `min.insync.replicas = 2` and `acks = all` for zero data loss.
- **Zero-Copy I/O**: Reads directly from OS Page Cache to Network Socket via `sendfile()` bypassing user-space JVM memory.
