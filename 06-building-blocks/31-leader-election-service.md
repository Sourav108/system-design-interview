# Building Block 31: Leader Election Service (Raft & Bully Algorithm)

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry / AI Infrastructure

---

## 1. Problem
Distributed clusters (databases, task schedulers, active-passive coordinators) require exactly one authoritative leader node to make decisions, serialize writes, and prevent conflicting concurrent actions.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
Without an automated, consensus-backed leader election mechanism, network partitions and node crashes cause either no leader (total system paralysis) or multiple leaders (Split-Brain data corruption).

## 4. Mental Model
A democratic parliament electing a Prime Minister; if the leader steps down or is incapacitated, members vote to elect a new leader using strict quorum rules.

## 5. Core Concepts
Raft Leader Election, Bully Algorithm, Zookeeper Ephemeral Sequential Nodes, Heartbeats & Randomized Election Timeouts, Split-Brain, Epoch Generation / Term Fencing.

## 6. Architecture
```mermaid
flowchart TD
    subgraph Cluster["Raft Consensus Cluster (3 Nodes)"]
        Node1["Node 1 (Leader - Term 2)"]
        Node2["Node 2 (Follower)"]
        Node3["Node 3 (Follower)"]

        Node1 -- Heartbeat / AppendEntries (every 50ms) --> Node2
        Node1 -- Heartbeat / AppendEntries (every 50ms) --> Node3
    end

    Node1 -.->|🔥 Crashes! Heartbeat Lost| Node2
    Node2 -->|Election Timeout 150-300ms expires -> Becomes Candidate (Term 3)| Election[Requests Votes from Node 3]
    Node3 -->|Grants Vote| Node2
    Node2 -->|Majority Quorum 2/3 Votes Received| NewLeader[Node 2 Becomes New Leader ✅]
```

## 7. Request/Data Flow
1. Leader sends regular heartbeats. 2. Leader crashes. 3. Follower's randomized election timeout (150ms–300ms) expires. 4. Follower increments Term, transitions to Candidate, votes for itself, and broadcasts RequestVote RPC. 5. If majority votes ($N/2 + 1$) received, transitions to Leader. 6. Broadcasts heartbeats to assert authority.

## 8. Data Model
Election State: `current_term (INT64)`, `voted_for (NODE_ID)`, `state (FOLLOWER/CANDIDATE/LEADER)`, `leader_id (NODE_ID)`.

## 9. API Design
`RequestVote(term, candidate_id, last_log_index, last_log_term) -> (term, vote_granted)`.

## 10. Algorithms
Raft Leader Election Algorithm with randomized election timers to prevent split-vote ties.

## 11. Scaling
Cluster sizes strictly odd ($N = 3, 5, 7$); scale client reads via Leader-Follower query distribution.

## 12. Partitioning
Single logical coordination cluster per consensus group.

## 13. Replication
State machine replicated across all $N$ quorum nodes via Raft log entries.

## 14. Consistency
Strict Linearizable Consistency (CP in CAP theorem).

## 15. Failure Scenarios
Network Partition splits cluster into $\{N1, N2\}$ vs $\{N3\}$: $\{N1, N2\}$ maintains majority quorum and elects leader; $\{N3\}$ isolated and cannot accept writes.

## 16. Recovery
Epoch generation fencing ensures that if an old isolated leader reconnects, all storage nodes reject its stale commands.

## 17. Observability
Leader Flapping Rate, Election Duration (Time to elect < 500ms), Heartbeat Latency, Term Number increments.

## 18. Security
mTLS cluster communication, shared secret tokens for inter-node RPC voting.

## 19. Performance
Randomized election timeouts ($150\text{ms} - 300\text{ms}$) prevent simultaneous split-vote stalemates.

## 20. Trade-offs
Consensus-Backed Leader (Zookeeper/Raft: strict consistency, safe against split brain) vs Optimistic Lock Leader (Redis: faster, risk of split-brain on partition).

## 21. When to Use
Primary database master election, distributed task scheduler coordinators, Kafka controller broker election.

## 22. When NOT to Use
Fully symmetric leaderless systems (e.g. Cassandra / DynamoDB quorum writes).

## 23. Implementation Strategy
Implement a Raft leader election simulator in Java using virtual threads, randomized election timers, and socket RPCs.

## 24. Practical Exercise
Simulate a 5-node Raft cluster in Java, isolate the current leader via network partition, and assert that the remaining 4 nodes elect a new leader in < 1 second.

## 25. Interview Questions
1. How do randomized election timeouts prevent split-vote deadlocks in Raft? 2. What is Split-Brain and how does majority quorum ($N/2 + 1$) prevent it? 3. What is an Epoch / Term Fencing Token?

## 26. Common Mistakes
Deploying an even number of consensus nodes (e.g. 4 nodes), where a 2-2 network split leaves neither side with a majority.

## 27. Quick Revision
Leader Election = Randomized timeout -> Candidate requests votes -> Majority ($N/2 + 1$) wins -> Epoch fencing prevents zombie leaders.

## 28. Related Building Blocks
`BB-09` (Consensus), `BB-21` (Distributed Lock), `BB-22` (Config Service)

## 29. Related Case Studies
`CS-14` (CI/CD Deployment), `CS-15` (Payment Gateway), `CS-16` (Online Judge)
