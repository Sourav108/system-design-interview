# Partial Failure & Failure Models

The defining characteristic of distributed computing is **partial failure**: one component fails while the rest of the system continues to operate in an unknown state.

---

## 1. Classical Distributed Failure Taxonomy

```mermaid
flowchart TD
    FM[Distributed Failure Models]
    FM --> CS[1. Crash-Stop: Node halts permanently; other nodes detect via timeout]
    FM --> CR[2. Crash-Recovery: Node halts, restarts, recovers state from WAL/Disk]
    FM --> OM[3. Omission Fault: Node drops incoming or outgoing network packets]
    FM --> BYZ[4. Byzantine Fault: Node behaves arbitrarily, sends corrupt or malicious messages]
```

---

## 2. The Anatomy of a Timeout: The Tri-State Uncertainty

When Node A sends a network request to Node B and receives no response within a timeout period, Node A is left in **tri-state uncertainty**:

```mermaid
sequenceDiagram
    autonumber
    participant A as Node A (Client)
    participant N as Network
    participant B as Node B (Server)

    A->>N: Request Sent
    alt Scenario 1: Request Lost in Transit
        N--x B: Packet Dropped (Server never saw it)
    else Scenario 2: Server Crashed During Execution
        N->>B: Request Delivered
        Note over B: Crashes mid-execution!
    else Scenario 3: Response Lost in Transit
        N->>B: Request Delivered & Executed Successfully ✅
        B->>N: Response Sent
        N--x A: Response Dropped (Client assumes failure!)
    end
    Note over A: Timeout Triggered! What happened to the state on Node B?
```

---

## 3. Fault Detection: Heartbeats, Leases & Phi Accrual

To distinguish between a slow node and a dead node:
- **Periodic Heartbeats**: Nodes broadcast keep-alive pings every $T_{\text{heartbeat}}$ seconds. Missing $K$ consecutive heartbeats marks the node as dead.
- **Leases**: A bounded time-based token granted by a coordinator. If the lease expires without renewal, the node loses authority to mutate state.
- **$\Phi$-Accrual Failure Detector (Cassandra / Akka)**: Rather than binary alive/dead decisions, it calculates a continuous suspicion probability metric $\Phi$ based on historical heartbeat intervals.
