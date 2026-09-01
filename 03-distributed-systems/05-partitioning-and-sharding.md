# Partitioning & Sharding

Partitioning (sharding) divides massive datasets into smaller, independent subsets distributed across multiple physical storage nodes to scale beyond single-machine disk and throughput limits.

---

## 1. Partitioning Strategies

```mermaid
flowchart TD
    Sharding[Sharding Strategies]
    Sharding --> Range[1. Range-Based: Keys partitioned by contiguous ranges, e.g. A-F, G-M]
    Sharding --> Hash[2. Hash-Based: Key hashed modulo N nodes: hash key mod N]
    Sharding --> CH[3. Consistent Hashing: Keys and nodes mapped onto a 2^32 circular ring]
```

---

## 2. Consistent Hashing with Virtual Nodes

Traditional `hash(key) % N` causes catastrophic cache invalidation when a node is added or removed: almost all keys remap to new servers. **Consistent Hashing** ensures that adding a node only migrates $\frac{K}{N}$ keys.

```mermaid
flowchart TD
    subgraph HashRing["Consistent Hash Ring (0 to 2^32 - 1)"]
        N1["Node A (Virtual Node A-1)"] --> K1["Key 1"]
        K1 --> N2["Node B (Virtual Node B-1)"]
        N2 --> K2["Key 2"]
        K2 --> N3["Node C (Virtual Node C-1)"]
        N3 --> N1
    end
```

- **Virtual Nodes (vnodes)**: Each physical machine is mapped to 100–250 points on the hash ring. This eliminates hot spots and ensures balanced data distribution across heterogeneous hardware.
