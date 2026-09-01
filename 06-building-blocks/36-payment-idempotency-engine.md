# Building Block 36: Payment Idempotency & Financial Ledger Engine

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry / AI Infrastructure

---

## 1. Problem
Flaky mobile networks and timeout retries cause clients to submit the same payment charge multiple times. Without strict idempotency, customers get billed twice and financial ledgers become corrupted.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
A Payment Idempotency Engine guarantees that an API call with a given `Idempotency-Key` executes exactly once, returning the identical cached response for all duplicate network retries without re-charging the customer.

## 4. Mental Model
A sealed physical bank deposit slip with a unique serial number; no matter how many times you hand the same slip to the teller, the funds are deposited exactly once.

## 5. Core Concepts
Idempotency Keys, Atomic Locking (`SET NX EX`), Double-Entry Bookkeeping (Debits = Credits), Two-Phase Commit vs Saga Pattern, Webhook Retries with Exponential Jitter, State Machine Reconciliation.

## 6. Architecture
```mermaid
flowchart TD
    Client[Client Checkout] -->|POST /v1/charges (Idempotency-Key: 'uuid-99')| Gateway[API Gateway]
    Gateway --> IdemFilter{Idempotency Filter}
    IdemFilter -->|1. Atomic SET NX EX| RedisLock[(Redis Lock: 'idem:uuid-99')]

    RedisLock -->|Lock Acquired: First Request| PaymentCore[Payment Processing Service]
    PaymentCore --> BankGateway[Third-Party Bank: Stripe/Visa]
    BankGateway -->|Charge Success| BankGateway
    PaymentCore --> LedgerDB[(Double-Entry SQL Ledger DB)]
    PaymentCore -->|2. Store Final Result in Redis (TTL 24h)| RedisLock
    PaymentCore --> Response[201 Created: {status: 'PAID', tx_id: 'tx_123'}]

    RedisLock -.->|Key Exists: Duplicate Retry| CachedResult[Return Cached Response: 200 OK (0 Bank Calls! 🛡️)]
```

## 7. Request/Data Flow
1. Client generates UUID `Idempotency-Key`. 2. Idempotency filter acquires Redis lock `SET idem:key PROCESSING NX EX 120`. 3. If key exists: returns cached response immediately. 4. If key is new: calls bank gateway. 5. Records double-entry debit/credit in PostgreSQL. 6. Caches final response in Redis for 24h. 7. Returns success.

## 8. Data Model
Idempotency Record: `idempotency_key (STRING)`, `user_id (STRING)`, `request_hash (SHA256)`, `response_code (INT)`, `response_body (JSON)`, `status (PROCESSING/SUCCESS/FAILED)`, `created_at (TIMESTAMP)`.

## 9. API Design
`POST /v1/charges` with HTTP Header `Idempotency-Key: 9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d`.

## 10. Algorithms
Atomic Compare-And-Swap (CAS), Double-Entry Balanced Ledger Accounting (Sum of Debits $\equiv$ Sum of Credits).

## 11. Scaling
Scale idempotency checks on Redis Cluster; scale persistent ledger on partitioned PostgreSQL.

## 12. Partitioning
Ledger partitioned by `hash(account_id)` ensuring all transactions for a single account execute on the same shard.

## 13. Replication
PostgreSQL Multi-AZ synchronous replication with zero RPO (zero data loss).

## 14. Consistency
Strict Serializability / Linearizability for ledger mutations (CP in CAP theorem).

## 15. Failure Scenarios
Client timeout while bank charge is executing (subsequent retry waits for lock or reads cached success), Bank gateway timeout.

## 16. Recovery
Background automated reconciliation worker periodically queries bank API for in-flight transactions and updates ledger.

## 17. Observability
Duplicate Request Rate (Blocked by Idempotency), Payment Success Rate, Ledger Imbalance Anomaly Count ($= 0$), Bank Gateway Latency.

## 18. Security
PCI-DSS compliance, credit card tokenization (never store raw PAN/CVV), TLS 1.3, encrypted ledger storage.

## 19. Performance
In-memory Redis lock check avoids 99% of unnecessary database lock queries during retry storms.

## 20. Trade-offs
Strict Financial Correctness (Linearizable, double-entry ledger, synchronous lock) vs High Write Throughput.

## 21. When to Use
Payment processing, bank account transfers, e-commerce checkout, stock trading order execution.

## 22. When NOT to Use
Non-financial loss-tolerant updates (e.g. updating social media bio or post likes).

## 23. Implementation Strategy
Build a production-grade Payment Idempotency filter in Java 21 / Spring Boot using Redis atomic operations and a PostgreSQL double-entry ledger transaction.

## 24. Practical Exercise
Execute 50 concurrent threads in Java submitting identical `Idempotency-Key` charge requests, asserting that the bank gateway is called exactly once and all 50 threads receive identical successful responses.

## 25. Interview Questions
1. Explain the step-by-step lifecycle of an Idempotency Key. 2. What is Double-Entry Bookkeeping and why is it mandatory for financial systems? 3. How do you handle a scenario where the bank charges the user but your database crashes before saving the transaction?

## 26. Common Mistakes
Allowing duplicate retries to execute when the client payload changes while reusing the same `Idempotency-Key` (must validate payload hash!).

## 27. Quick Revision
Payment Idempotency = Unique Key -> Atomic Redis lock -> Process once -> Cache response 24h -> Return cached result on retry.

## 28. Related Building Blocks
`BB-03` (RDBMS), `BB-10` (Distributed Cache), `BB-21` (Distributed Lock)

## 29. Related Case Studies
`CS-15` (Payment Gateway / Stripe), `CS-05` (Uber Ride Billing), `CS-14` (CI/CD Deployment)
