# Step 5: Data Modeling & Storage Selection

Data modeling in System Design is not just writing SQL tables; it is choosing the correct **storage paradigm** and defining the **sharding key** that prevents hot partitions.

---

## 1. Storage Engine Decision Matrix

```mermaid
flowchart TD
    DataReq{What is your primary data & access pattern?}
    DataReq -->|ACID Transactions, Financial Ledgers, Complex Joins| Relational[Relational SQL: PostgreSQL / MySQL]
    DataReq -->|High Write QPS, Append-Only Time-Series, Wide Columns| LSM[Distributed Wide-Column: Cassandra / ScyllaDB]
    DataReq -->|Sub-millisecond Key Lookups, Session Cache| KV[Distributed KV / In-Memory: Redis / DynamoDB]
    DataReq -->|Full-Text Fuzzy Search, Inverted Index| SearchEngine[Search Index: Elasticsearch / OpenSearch]
    DataReq -->|Unstructured Images, Videos, Binary Blobs| ObjectStore[Blob / Object Store: Amazon S3 / GCS]
    DataReq -->|Semantic Vectors, High-Dimensional Embeddings| VectorDB[Vector Search: HNSW / Milvus / Pinecone]
```

---

## 2. Defining Schemas & Sharding Keys

When presenting a data model in an interview:
1. **Define Core Entities**: Name fields, types, and primary keys.
2. **Identify the Sharding / Partition Key**: Choose a partition key with high cardinality and even write distribution (e.g. `user_id` or `hash(tenant_id)`).
3. **Analyze Access Patterns**: Ensure that 95%+ of queries can be satisfied from a single partition without requiring costly cross-partition scatter-gather queries.
