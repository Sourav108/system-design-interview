# Idempotency & Exactly-Once Processing

In distributed systems communicating over unreliable networks, true physical "exactly-once transmission" is impossible. Instead, systems achieve **exactly-once processing semantics** by combining **At-Least-Once Delivery** with **Idempotency**.

---

## 1. The Core Equation of Distributed Reliability

$$\text{At-Least-Once Delivery (Retries)} + \text{Idempotent Consumer (Deduplication)} \iff \mathbf{Exactly\text{-}Once\text{ Semantics}}$$

---

## 2. The Idempotency Key Architecture

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant Gateway as API Gateway
    participant IdemFilter as Idempotency Filter
    participant Redis as Redis / DB Idempotency Store
    participant CoreSvc as Payment Core Service
    participant DB as Ledger DB

    Client->>Gateway: POST /v1/payments (Idempotency-Key: "uuid-12345")
    Gateway->>IdemFilter: Intercept Request
    IdemFilter->>Redis: SET "idem:uuid-12345" "PROCESSING" NX EX 120

    alt First Request (Key was Absent)
        Redis-->>IdemFilter: Key Created (Lock Acquired ✅)
        IdemFilter->>CoreSvc: Execute Charge Transaction
        CoreSvc->>DB: Insert Ledger Entry
        CoreSvc-->>IdemFilter: Return Charge Result {status: "SUCCESS", txId: "tx_999"}
        IdemFilter->>Redis: SET "idem:uuid-12345" "{status: SUCCESS, txId: tx_999}" EX 86400
        IdemFilter-->>Client: 201 Created {status: "SUCCESS", txId: "tx_999"}
    else Duplicate Retry (Key Already Exists)
        Redis-->>IdemFilter: Returns Cached Result {status: "SUCCESS", txId: "tx_999"}
        IdemFilter-->>Client: 200 OK {status: "SUCCESS", txId: "tx_999"} (Zero Duplicate Charge! 🛡️)
    end
```

---

## 3. Distributed Transactions: Two-Phase Commit (2PC) vs Saga

- **Two-Phase Commit (2PC)**: Synchronous, blocking distributed transaction with a coordinator. If the coordinator crashes mid-commit, all nodes remain locked indefinitely. Not suitable for microservices.
- **Saga Pattern**: Sequence of local transactions where each step publishes an event triggering the next step. If a step fails, the Saga executes **compensating transactions** in reverse order to rollback state.
