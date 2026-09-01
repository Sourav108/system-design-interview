# Partitioning & Database Sharding

| Partitioning Strategy | Routing Mechanism | Advantages | Disadvantages / Hotspots |
|---|---|---|---|
| **Hash-Based Partitioning** | `hash(key) % num_shards` | Uniform data distribution across nodes | Range queries require scatter-gather across all shards |
| **Range-Based Partitioning** | Routes by value ranges (`A-D`, `E-H`) | Efficient range queries on single shard | Hotspots on sequential timestamp writes |
| **Consistent Hashing** | Virtual nodes on a hash ring | Adding/removing nodes moves only $K/N$ keys | Requires virtual node tuning for uniform balance |
| **Directory-Based Lookup** | Central lookup table maps ID $\to$ Shard | Dynamic shard rebalancing | Lookup service is a central bottleneck |
