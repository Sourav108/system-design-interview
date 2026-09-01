# Building Block 04: Distributed Key-Value Store (LSM-Tree & NoSQL)

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry

---

## 1. Problem
Relational databases suffer from write amplification and high join overhead at massive write scales ($> 100,000\text{ writes/sec}$). Applications need ultra-fast $\mathcal{O}(1)$ key lookups and high-throughput ingestion.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
Key-Value stores discard relational joins and rigid table schemas in exchange for massive horizontal write scalability, tunable consistency, and single-digit millisecond latency.

## 4. Mental Model
A massive, horizontally partitioned in-memory and on-disk dictionary spanning hundreds of servers.

## 5. Core Concepts
LSM-Tree (Log-Structured Merge-tree), MemTable, SSTables (Sorted String Tables), Bloom Filters, Compaction (Size-Tiered vs Leveled), Consistent Hashing, Tunable Consistency ($R + W > N$).

## 6. Architecture
```mermaid
flowchart TD
    Client[Client Write: PUT key, val] --> MemTable[In-Memory MemTable (SkipList)]
    Client -. Append-Only .-> WAL[(Write-Ahead Log on Disk)]
    MemTable -- Flush when Full --> L0[L0 SSTable on Disk]
    L0 -- Compaction --> L1[L1 SSTable on Disk]
    L1 -- Compaction --> L2[L2 SSTable on Disk]
```

## 7. Request/Data Flow
1. Write: Appends to WAL on disk, inserts into in-memory MemTable (SkipList). Returns success. 2. Read: Checks MemTable -> Immutable MemTable -> Bloom Filters -> L0 to LN SSTables via binary search.

## 8. Data Model
Key-Value pairs: `Key (BYTE[]) -> Value (BYTE[], JSON, Protocol Buffer, Document)`.

## 9. API Design
`GET(key)`, `PUT(key, value)`, `DELETE(key)` (writes a tombstone marker).

## 10. Algorithms
SkipList for in-memory MemTable, Bloom Filter for fast negative disk lookups, Leveled Compaction algorithm.

## 11. Scaling
Consistent Hashing on primary key across $N$ storage nodes; virtual nodes ensure balanced cluster load.

## 12. Partitioning
Hash partition of `key` determines replica placement on the consistent hash ring.

## 13. Replication
Dynamo-style Leaderless replication with Quorum reads/writes ($R + W > N$), Read Repair, and Hinted Handoff.

## 14. Consistency
Tunable: Eventual Consistency ($R=1, W=1$) to Strong Linearizability ($R+W > N$ with Quorum).

## 15. Failure Scenarios
Node crash (Hinted Handoff stores writes temporarily), Disk corruption, Read amplification during heavy uncompacted writes.

## 16. Recovery
Anti-entropy background sync using Merkle trees to reconcile diverging replica partitions.

## 17. Observability
Read/Write latency (p99 < 5ms), MemTable flush rate, Compaction backlog, Bloom filter false positive rate.

## 18. Security
TLS communication, node-to-node authentication, client API tokens, data-at-rest encryption.

## 19. Performance
LSM-Trees convert random writes into sequential disk appends; Bloom filters bypass 99% of unnecessary disk seeks.

## 20. Trade-offs
Write Throughput (LSM wins) vs Read Latency (B-Tree wins on pure point reads due to zero compaction overhead).

## 21. When to Use
User sessions, shopping carts, metadata caches, high-frequency IoT sensor telemetry, URL mapping stores.

## 22. When NOT to Use
Complex multi-table relational joins, foreign key constraints, ad-hoc aggregation queries.

## 23. Implementation Strategy
Deploy Cassandra or DynamoDB with consistent partition keys, appropriate compaction strategies, and quorum read/write profiles.

## 24. Practical Exercise
Implement a mini LSM-Tree in Java using a `ConcurrentSkipListMap` (MemTable) and binary SSTable file writer with Bloom filter.

## 25. Interview Questions
1. How does an LSM-Tree achieve superior write performance compared to a B-Tree? 2. What is the role of a Bloom Filter in read optimization? 3. Explain how SSTable Compaction works.

## 26. Common Mistakes
Choosing a low-cardinality partition key (e.g. `status = 'ACTIVE'`) causing massive single-node hot partition burnout.

## 27. Quick Revision
LSM = In-memory MemTable + Append-only WAL + On-disk SSTables + Bloom Filters + Background Compaction.

## 28. Related Building Blocks
`BB-03` (RDBMS), `BB-10` (Distributed Cache), `BB-18` (Sharded Counter)

## 29. Related Case Studies
`CS-09` (TinyURL), `CS-11` (WhatsApp / Discord), `CS-06` (Twitter / X)
