# Building Block 26: Feature Flagging Service (Flipt & Unleash)

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry

---

## 1. Problem
Deploying new code directly to 100% of production users in a single release creates severe blast radiuses when critical bugs or performance regressions occur.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
A Feature Flagging Service decouples code deployment from feature release. Engineers deploy code dark, and dynamically toggle features on/off or roll out to targeted user percentages (Canary releases) in real time without redeploying code.

## 4. Mental Model
A master light switchboard controlling lights in individual rooms, allowing operators to gradually brighten a room from 0% to 100%.

## 5. Core Concepts
Feature Toggles (Release, Experimentation, Ops, Permission), Percentage-Based Rollouts, User Targeting Rules, Client-Side Evaluation Caching, Local Flag Evaluators, A/B Testing.

## 6. Architecture
```mermaid
flowchart TD
    Admin[Product Manager / Developer] -->|Update Rollout: 'new-checkout' = 10% Canary| FlagService[Feature Flag Control Plane: Unleash / Flipt]
    FlagService --> FlagDB[(Flag Rules DB: PostgreSQL)]

    subgraph MicroserviceFleet["Microservice Instances (Local In-Memory Evaluation)"]
        Svc1["App Instance 1 (Local Rule Engine)"]
        Svc2["App Instance 2 (Local Rule Engine)"]
    end

    FlagService -- SSE / Long Polling Rule Sync --> Svc1
    FlagService -- SSE / Long Polling Rule Sync --> Svc2

    User[User Request: user_id = 9876] --> Svc1
    Svc1 -->|Evaluate: MurmurHash3(user_id) % 100 < 10| Enabled[Serve New Feature ✅ (0ms Network Latency)]
```

## 7. Request/Data Flow
1. PM enables feature for 10% of users in US region. 2. Flag rules synced to application SDKs via SSE/WebSocket. 3. User request arrives. 4. In-process SDK hashes `user_id` + `flag_key` using MurmurHash3. 5. If `hash % 100 < 10`, returns `true`. 6. Evaluates locally in < 1 microsecond.

## 8. Data Model
Feature Flag: `flag_key (STRING)`, `enabled (BOOL)`, `rollout_percentage (INT 0-100)`, `targeting_rules (JSON)`, `variants (MAP)`.

## 9. API Design
`GET /api/v1/flags/evaluate?user_id=123`, `POST /api/v1/admin/flags`.

## 10. Algorithms
MurmurHash3 deterministic percentage bucketing: `(MurmurHash3(userId + flagKey) % 100) < rolloutPercentage`.

## 11. Scaling
Zero evaluation latency on request path; control plane scales via CDN caching of flag rulesets.

## 12. Partitioning
Flag rules sharded by Project / Team namespace.

## 13. Replication
Multi-AZ PostgreSQL database for control plane rules; rules replicated to edge caches.

## 14. Consistency
Eventual consistency of flag rules (updates propagate to all instances within 1–5 seconds).

## 15. Failure Scenarios
Control plane outage (SDKs fallback gracefully to local in-memory cached rulesets), Flapping flag configurations.

## 16. Recovery
Local SDK fallback defaults hardcoded in application binaries if remote config is unreachable.

## 17. Observability
Flag evaluation latency (< 5 microseconds in memory), Active Flag count, Rule sync lag from control plane.

## 18. Security
Audit logging of all flag state mutations, RBAC permissions for production toggle activations.

## 19. Performance
In-memory evaluation inside the application process; ruleset synchronization via Server-Sent Events (SSE).

## 20. Trade-offs
In-Process SDK Evaluation (Zero network overhead, memory resident) vs Remote Evaluation API (Extra network hop per request).

## 21. When to Use
Canary deployments, dark launches, percentage-based rollouts, A/B experimentation, emergency kill switches.

## 22. When NOT to Use
High-frequency per-transaction database business state management.

## 23. Implementation Strategy
Integrate Unleash or OpenFeature Java SDK in Spring Boot with local in-memory evaluation and SSE rule synchronization.

## 24. Practical Exercise
Implement a deterministic percentage rollout evaluator in Java using MurmurHash3 and verify consistent assignment for 100,000 simulated users across 5% rollout.

## 25. Interview Questions
1. How does deterministic hashing ensure a user stays in the same Canary bucket across multiple sessions? 2. Why should feature flags be evaluated in-memory rather than via remote HTTP calls? 3. What is Feature Flag Debt and how do you manage it?

## 26. Common Mistakes
Making a remote HTTP network call to a feature flag server on every incoming user request (adds 50ms latency per request!).

## 27. Quick Revision
Feature Flag = In-memory rule evaluation -> MurmurHash3 percentage bucketing -> Zero network latency -> Instant rollback kill switch.

## 28. Related Building Blocks
`BB-22` (Config Service), `BB-19` (API Gateway)

## 29. Related Case Studies
`CS-06` (Twitter / X), `CS-14` (CI/CD Deployment), `CS-15` (Payment Gateway)
