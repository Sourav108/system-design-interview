# Distributed Consensus (Paxos, Raft & Zab)

Consensus is the problem of getting a cluster of distributed nodes to agree on a single data value or sequence of state machine operations, even when some nodes crash or messages are lost.

---

## 1. The Raft Consensus Algorithm

Raft decomposes consensus into three independent sub-problems: **Leader Election**, **Log Replication**, and **Safety**.

```mermaid
stateDiagram-v2
    [*] --> Follower
    Follower --> Candidate: Election Timeout (150ms-300ms randomized)
    Candidate --> Leader: Receives Majority Votes (N/2 + 1)
    Candidate --> Follower: Discovers higher term leader
    Leader --> Follower: Steps down upon seeing higher term
```

---

## 2. Raft Quorum Log Replication Workflow

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant Leader as Raft Leader
    participant F1 as Follower 1
    participant F2 as Follower 2

    Client->>Leader: Command: SET key = "val"
    Note over Leader: 1. Append entry to local uncommitted log
    par Broadcast Log Entries
        Leader->>F1: AppendEntries RPC (Term 1, Index 5)
        Leader->>F2: AppendEntries RPC (Term 1, Index 5)
    end
    F1->>Leader: Append OK ✅
    Note over Leader: 2. Majority ACK received (Leader + F1 = 2/3 Quorum)!
    Note over Leader: 3. Commit entry to local State Machine
    Leader->>Client: Success ✅ (Committed)
    Leader->>F2: Next AppendEntries communicates commitIndex = 5
    F2->>Leader: Append OK ✅
```

---

## 3. Epoch Generation & Fencing Tokens

To prevent an old, partitioned leader (zombie leader) from executing writes after a new leader has been elected:
- Every election increments an **Epoch / Term Number**.
- Storage backends reject any write tagged with an epoch lower than the highest seen epoch (Fencing Token pattern).
