# Caching Strategies & Eviction Policies

| Caching Pattern | Read Flow | Write Flow | Consistency | Best For |
|---|---|---|---|---|
| **Cache-Aside (Lazy Loading)** | App checks Cache $\to$ on miss, reads DB $\to$ writes to Cache | App writes directly to DB $\to$ deletes/invalidates Cache key | Eventual (Stale window between DB write & cache eviction) | Read-heavy general web workloads |
| **Write-Through** | App reads Cache directly | App writes to Cache $\to$ Cache synchronously writes to DB | High (Cache & DB always in sync) | Read-heavy with strict freshness |
| **Write-Behind (Write-Back)** | App reads Cache directly | App writes to Cache $\to$ Cache asynchronously batches writes to DB | Eventual (Risk of data loss on cache node crash) | Write-heavy ingestion (views/likes) |
| **Refresh-Ahead** | Cache automatically re-fetches hot keys before TTL expires | Standard write | High (Zero read miss latency) | Extremely hot predictable keys |

---

## Eviction Policies at a Glance
- **LRU (Least Recently Used)**: Discards items not accessed for the longest time. Standard for general caches.
- **LFU (Least Frequently Used)**: Discards items with lowest access counter. Best for stable popularity distributions.
- **FIFO (First In, First Out)**: Discards oldest inserted items regardless of access frequency.
- **ARC (Adaptive Replacement Cache)**: Dynamically balances between LRU and LFU based on recent hit patterns.
