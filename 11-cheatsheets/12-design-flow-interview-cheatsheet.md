# The 45-Minute DESIGN-FLOW Interview Playbook

| Time Allocation | Step | Core Objectives |
|---|---|---|
| **00:00 - 05:00** | **1. Scope & Requirements** | Clarify Functional & Non-Functional Requirements, establish scale constraints. |
| **05:00 - 10:00** | **2. Capacity Estimation** | Calculate Read/Write QPS, 5-year storage, bandwidth, and 80-20 cache sizing. |
| **10:00 - 15:00** | **3. High-Level Architecture** | Draw end-to-end Mermaid diagram connecting Clients, CDN, Gateway, DB, Cache. |
| **15:00 - 25:00** | **4. Data Model & Deep Dive** | Define table schemas, partition keys, caching patterns, and core algorithms. |
| **25:00 - 35:00** | **5. Scaling & Bottlenecks** | Address hot keys, replication lag, partitioning, database bottlenecks. |
| **35:00 - 40:00** | **6. Reliability & Failure Modes** | Detail circuit breakers, DLQs, idempotency, split-brain defenses. |
| **40:00 - 45:00** | **7. Summary & Trade-offs** | Defend key architectural trade-offs and address interviewer follow-ups. |
