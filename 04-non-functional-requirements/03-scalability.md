# Scalability in Distributed Systems

Scalability is the ability of a system to handle linearly growing workloads gracefully by adding computational, storage, or network resources.

---

## 1. Vertical Scaling (Scale Up) vs Horizontal Scaling (Scale Out)

```mermaid
flowchart TD
    subgraph ScaleUp["Vertical Scaling (Scale Up)"]
        S1["Single Server: 4 Cores, 16GB RAM"] --> S2["Upgrade Hardware: 64 Cores, 512GB RAM, 10Gbps NIC"]
        S2 --> Limit["Physical Hardware Limit & Exponential Cost Ceiling"]
    end

    subgraph ScaleOut["Horizontal Scaling (Scale Out)"]
        H1["1 Node (4 Cores)"] --> H2["Add Nodes Horizontally Behind Load Balancer"]
        H2 --> H3["100 Nodes (400 Cores Total, Linear Cost, Zero Downtime)"]
    end
```

---

## 2. The 3 Scaling Dimensions: The Scale Cube (Abbott & Fisher)

```mermaid
flowchart TD
    Cube[The Scale Cube]
    Cube --> X[X-Axis: Horizontal Duplication of Stateless Services]
    Cube --> Y[Y-Axis: Functional Decomposition into Microservices]
    Cube --> Z[Z-Axis: Data Partitioning / Sharding by User / Region]
```

1. **X-Axis Scaling (Duplication)**: Run $N$ identical stateless clones of an application behind an L7 load balancer.
2. **Y-Axis Scaling (Decomposition)**: Split a large monolithic code base into distinct bounded services (`User Service`, `Order Service`, `Payment Service`).
3. **Z-Axis Scaling (Data Sharding)**: Partition state across independent database clusters by customer ID or geographic country (`US Shard`, `EU Shard`, `APAC Shard`).
