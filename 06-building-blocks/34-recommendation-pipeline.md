# Building Block 34: Real-Time Recommendation Pipeline

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry / AI Infrastructure

---

## 1. Problem
Users browsing media or e-commerce platforms expect personalized content recommendations selected from a catalog of 100 million items in $< 100\text{ms}$ based on their real-time clicks and historical preferences.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
Running heavy deep-learning neural network ranking models across 100 million items in real-time is computationally impossible. Modern recommendation architectures use a multi-stage funnel: Candidate Generation (Retrieval) -> Scoring / Filtering -> Heavy Re-Ranking.

## 4. Mental Model
A talent scouting audition funnel: 100,000 applicants filtered down to 500 semi-finalists by scouts, and top 10 finalists selected by master judges.

## 5. Core Concepts
Multi-Stage Funnel (Candidate Generation -> Filtering -> Ranking -> Re-ranking / Diversity), Two-Tower Neural Network Embeddings, Vector Search (Approximate Nearest Neighbors), Real-Time Feature Store (Feast), Flink Stream Aggregations.

## 6. Architecture
```mermaid
flowchart TD
    UserActivity["Real-Time User Action (Watched Video X)"] --> Stream[Kafka Activity Stream]
    Stream --> Flink[Apache Flink Stream Processor]
    Flink --> FeatureStore[(Real-Time Feature Store: Redis)]

    UserOpensApp[User Opens Home Screen] --> RecoEngine[Recommendation Orchestrator]
    RecoEngine -->|1. Fetch Real-Time User Profile| FeatureStore

    subgraph MultiStageFunnel["Recommendation Funnel (100M -> 20 Items)"]
        Stage1["1. Candidate Retrieval: Vector Search (HNSW) -> 1,000 Candidates"]
        Stage2["2. Business Filtering: Remove Watched/Blocked -> 500 Candidates"]
        Stage3["3. Heavy ML Ranking: GBDT / Deep Learning Model -> 50 Ranked Items"]
        Stage4["4. Re-Ranking & Diversity: De-duplicate categories -> Top 20 Items"]
        Stage1 --> Stage2 --> Stage3 --> Stage4
    end
    RecoEngine --> MultiStageFunnel
    MultiStageFunnel --> ReturnFeed[Return Personalized Feed ✅ (p99 < 80ms)]
```

## 7. Request/Data Flow
1. User action ingested into Kafka. 2. Flink updates real-time user features in Redis. 3. User requests feed. 4. Retrieval stage uses Approximate Nearest Neighbors (ANN) vector search to pull top 1,000 candidate items. 5. Filters out watched items. 6. ML ranker scores 500 items. 7. Diversity layer selects top 20 items.

## 8. Data Model
User Profile Feature: `user_id (STRING)`, `embedding_vector (FLOAT[256])`, `recent_clicks (LIST)`, `demographics (MAP)`.

## 9. API Design
`POST /v1/recommendations { user_id, context, limit: 20 }`.

## 10. Algorithms
Two-Tower Model (User Tower & Item Tower Dot Product), HNSW Vector Indexing, Maximal Marginal Relevance (MMR for Diversity).

## 11. Scaling
Candidate retrieval scales on Vector DB cluster; ML inference scales horizontally on GPU worker clusters (Triton Inference Server).

## 12. Partitioning
Vector embeddings partitioned by Item Category and HNSW shard.

## 13. Replication
Feature Store replicated across multi-AZ Redis Cluster; Embeddings stored in Milvus / Pinecone.

## 14. Consistency
Eventual consistency; user feature updates propagate in $< 500\text{ms}$.

## 15. Failure Scenarios
Cold-start problem for new users/items, Feature store timeout, Model inference latency spikes.

## 16. Recovery
Fallback to popular/trending items if personalized candidate retrieval times out ($> 50\text{ms}$).

## 17. Observability
Recommendation CTR (Click-Through Rate), Funnel Stage Latencies (Retrieval < 10ms, Ranking < 40ms), Feature Freshness Lag.

## 18. Security
Filtering inappropriate/NSFW content in the business logic filtering stage; privacy-safe anonymized feature storage.

## 19. Performance
Asynchronous candidate pre-fetching; caching offline candidate pools for low-activity users.

## 20. Trade-offs
Retrieval (Fast, coarse, millions $\to$ thousands) vs Ranking (Heavy, precise, thousands $\to$ tens).

## 21. When to Use
YouTube video recommendations, Netflix movie rows, TikTok For You page, Amazon 'Customers Also Bought'.

## 22. When NOT to Use
Purely deterministic sorted business listings (e.g. chronological transaction statements).

## 23. Implementation Strategy
Build a recommendation pipeline in Java orchestrating an HNSW vector search retrieval step with a real-time Redis feature store and fallback rules.

## 24. Practical Exercise
Simulate a 2-stage recommendation funnel in Java: retrieve 1,000 candidate item embeddings via cosine similarity, filter out 200 watched items, and score remaining items.

## 25. Interview Questions
1. Explain the 4 stages of the modern recommendation funnel. 2. How does a Two-Tower Neural Network enable sub-10ms candidate retrieval? 3. How do you solve the Cold-Start problem for new users?

## 26. Common Mistakes
Attempting to run complex deep neural network scoring across the entire 100-million item database on every user request.

## 27. Quick Revision
Recommendation Pipeline = Real-time Feature Store + Vector Retrieval (100M -> 1k) -> Filter -> ML Rank (1k -> 50) -> Diversity Re-rank (Top 20).

## 28. Related Building Blocks
`BB-10` (Cache), `BB-12` (Kafka), `BB-39` (Vector Search)

## 29. Related Case Studies
`CS-01` (YouTube), `CS-07` (Newsfeed), `CS-08` (Instagram)
