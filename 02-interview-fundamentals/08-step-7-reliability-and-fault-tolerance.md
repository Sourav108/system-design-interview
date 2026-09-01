# Step 7: Reliability & Fault Tolerance

A senior architect designs systems assuming that **everything will fail at scale**: servers will crash, networks will partition, disks will corrupt, and downstream APIs will time out.

---

## 1. Core Resiliency Patterns

```mermaid
flowchart TD
    subgraph Resiliency["Distributed Resiliency Toolkit"]
        R1["1. Multi-AZ & Cross-Region Replication (Leader-Follower / Quorum)"]
        R2["2. Circuit Breakers (Closed -> Open -> Half-Open with Resilience4j)"]
        R3["3. Exponential Backoff with Full Jitter (Prevent Retry Storms)"]
        R4["4. Idempotency Keys (Prevent Double Charging / Duplicate Mutations)"]
        R5["5. Rate Limiting & Bulkheading (Protect Core Infrastructure)"]
        R6["6. Dead-Letter Queues (DLQ) (Isolate Poison-Pill Messages)"]
    end
```

---

## 2. Exponential Backoff with Full Jitter

When retrying failed network requests, naive fixed retries cause devastating **retry storms**. Adding full random jitter spreads out client retries across time:

$$t_{	ext{sleep}} = 	ext{random}(0, \min(t_{	ext{max}}, t_{	ext{base}} 	imes 2^{	ext{attempt}}))$$

```mermaid
sequenceDiagram
    autonumber
    Client->>Service: Request 1 (Fails / Timeout)
    Note over Client: Backoff: Sleep random(0, 100ms)
    Client->>Service: Retry 1 (Fails / Timeout)
    Note over Client: Backoff: Sleep random(0, 200ms)
    Client->>Service: Retry 2 (Success ✅)
```
