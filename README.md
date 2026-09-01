# System Design Interview — Implementation-First Curriculum

An original, implementation-first, deep-dive System Design curriculum engineered for **SDE2**, **Senior Backend Engineer**, and **Staff Engineer** technical interview preparation.

---

## 🎯 Learning Philosophy

```
UNDERSTAND → VISUALIZE → IMPLEMENT → REUSE → APPLY → EXPLAIN → DEFEND TRADE-OFFS
```

This repository is **NOT** a collection of abstract, hand-wavy bullet points. Every topic bridges theoretical distributed systems foundations with concrete, runnable Java/Spring Boot implementations, enterprise architectural case studies, and interview defense strategies.

---

## 📐 The DESIGN-FLOW Interview Framework

We structure all 45-minute System Design interviews around the **DESIGN-FLOW** framework:

```mermaid
flowchart LR
    R[1. Requirements] --> C[2. Constraints]
    C --> S[3. Scale]
    S --> A[4. Architecture]
    A --> D[5. Data Model]
    D --> COM[6. Communication]
    COM --> REL[7. Reliability]
    REL --> B[8. Bottlenecks]
    B --> T[9. Trade-offs]
    T --> DD[10. Deep Dive]
```

1. **Requirements**: Functional boundaries, user workflows, out-of-scope boundaries.
2. **Constraints**: SLOs, availability targets (99.99%), latency bounds (p99 < 50ms), consistency requirements.
3. **Scale**: Peak QPS, read/write ratio, ingress/egress bandwidth, storage trajectory (5-year).
4. **Architecture**: High-level component decomposition, API gateway, stateless services, storage tier.
5. **Data Model**: SQL vs NoSQL schema, primary keys, sharding keys, indexing, access patterns.
6. **Communication**: Sync (gRPC, REST) vs Async (Kafka, RabbitMQ), WebSocket, Long-Polling.
7. **Reliability**: Replication, failover, idempotency, circuit breakers, rate limiting.
8. **Bottlenecks**: Single points of failure (SPOF), hot partitions, cache stampedes, slow queries.
9. **Trade-offs**: Latency vs Consistency (CAP/PACELC), throughput vs durability, cost vs performance.
10. **Deep Dive**: Low-level algorithms, concurrency control, custom partitioning, distributed locks.

---

## 🗺️ Curriculum Structure

| Module | Focus Area | Description |
|---|---|---|
| **`01-introduction/`** | Foundations & Communication | What is System Design, HLD vs LLD, ambiguity navigation, architecture diagramming. |
| **`02-interview-fundamentals/`** | The 45-Minute Game Plan | The DESIGN-FLOW framework, pacing, active listening, trade-off defense. |
| **`03-distributed-systems/`** | Core Distributed Theory | Network models, partial failures, replication, partitioning, consensus, CAP/PACELC, logical clocks. |
| **`04-non-functional-requirements/`** | System Invariants | Availability, reliability, scalability, latency, throughput, durability, maintainability, security. |
| **`05-capacity-estimation/`** | Back-of-the-Envelope Math | QPS, peak multipliers, bandwidth, memory, cache sizing, storage growth, server count math. |
| **`06-building-blocks/`** | 40 Reusable Components | DNS, LBs, Caches, Sequencers, Rate Limiters, Queues, Pub/Sub, Vector Search, AI Gateway. |
| **`07-case-studies/`** | 20 Real-World Systems | Video Streaming, Rideshare, Social Feeds, Proximity Search, Chat, Search Autocomplete, Payment Gateway. |
| **`08-system-failures/`** | Production Post-Mortems | Cascading failures, retry storms, thundering herds, split-brain, hot keys, clock drift. |
| **`09-implementations/`** | Runnable Code (Java/Spring) | Production-grade reference code: Rate Limiter, Distributed Cache, Task Scheduler, Idempotency. |
| **`10-interview-practice/`** | Question Bank & Mocks | 100 System Design questions, 30 estimations, 30 trade-offs, 20 failures, 10 full mock interviews. |
| **`11-cheatsheets/`** | Last-Minute Revision | High-density cheat sheets: DB selection, caching patterns, Kafka, replication, consistency. |
| **`12-modern-ai-systems/`** | AI Infrastructure & LLMs | LLM inference serving, RAG pipelines, Vector Search, AI Gateways, GPU sizing, streaming tokens. |
| **`roadmap/`** | Study Roadmaps & Graphs | 6-Week SDE2/Senior Roadmap, Dependency Graph, Coverage Matrix. |

---

## 🧭 Single Source of Truth

Track full progress, dependencies, and component mappings in [**CURRICULUM.md**](./CURRICULUM.md).
