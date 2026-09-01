# Database Selection Matrix

| Database Paradigm | Top Technologies | Primary Strength | Weakness / Anti-Pattern | Best Use Cases |
|---|---|---|---|---|
| **Relational (RDBMS)** | PostgreSQL, MySQL, CockroachDB | Strong ACID transactions, complex SQL joins, schema enforcement | Horizontal write scaling beyond 20k writes/sec | Financial ledgers, user accounts, e-commerce orders |
| **Key-Value Store** | Redis, DynamoDB, Memcached | Ultra-low latency ($\mathcal{O}(1) < 1\text{ms}$), simple scaling | Complex relational joins, multi-table queries | Session state, rate limiters, user carts, hot caches |
| **Wide-Column (LSM)** | Apache Cassandra, ScyllaDB, Bigtable | Massive horizontal write scale ($> 100\text{k writes/s}$), zero SPOF | Ad-hoc analytics queries, frequent updates to same row | Time-series, IoT sensors, chat messaging history |
| **Document Store** | MongoDB, Amazon DocumentDB | Flexible dynamic JSON schema, fast document reads | Distributed ACID transactions across collections | Content management (CMS), product catalogs |
| **Time-Series (TSDB)** | Prometheus, VictoriaMetrics, InfluxDB | Double-delta timestamp & float compression (Gorilla) | High-cardinality string labels, non-time data | Server CPU/Memory metrics, IoT sensor telemetry |
| **Search Engine** | Elasticsearch, OpenSearch | Full-text fuzzy search, BM25 relevance scoring | Primary transactional system of record | Product catalog search, log aggregation (ELK) |
| **Graph Database** | Neo4j, AWS Neptune | Fast multi-hop graph traversals ($\mathcal{O}(1)$ edge hops) | High-volume single-key point writes | Social network follower graphs, fraud rings |
| **Vector Database** | Milvus, Pinecone, Qdrant | Fast HNSW Approximate Nearest Neighbor search | Exact keyword matches, scalar ACID operations | LLM RAG pipelines, semantic search, recommendations |
| **Cloud Lakehouse** | Apache Iceberg, Snowflake, BigQuery | Columnar Parquet on S3, petabyte analytical SQL | Low-latency single-row point lookups ($< 50\text{ms}$) | Big data analytics, business intelligence, data science |
