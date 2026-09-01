# Distributed Replication Strategies

Replication is the process of storing copies of the same data on multiple physical machines to ensure **high availability**, **fault tolerance**, and **low-latency geographic reads**.

---

## 1. The 3 Core Replication Models

```mermaid
flowchart TD
    subgraph SingleLeader["1. Single-Leader (Primary-Follower)"]
        Leader1[Leader / Master] -- Synchronous / Asynchronous --> Follower1[Follower Replica 1]
        Leader1 -- Replication Stream --> Follower2[Follower Replica 2]
    end

    subgraph MultiLeader["2. Multi-Leader (Active-Active)"]
        LeaderUS[US Leader] <== Bi-Directional Async Replication ==> LeaderEU[EU Leader]
        LeaderUS --> FollowerUS[US Read Replica]
        LeaderEU --> FollowerEU[EU Read Replica]
    end

    subgraph Leaderless["3. Leaderless (Dynamo-Style Quorum)"]
        Client[Client / Coordinator] --> Node1[(Node 1)]
        Client --> Node2[(Node 2)]
        Client --> Node3[(Node 3)]
    end
```

---

## 2. Leaderless Quorum Mathematics ($R + W > N$)

In Dynamo-style distributed stores (Amazon DynamoDB, Apache Cassandra):
- $N$ = Total replication factor (total copies of data).
- $W$ = Number of nodes that must acknowledge a write before returning success.
- $R$ = Number of nodes that must respond to a read before returning the latest value.

$$\text{Strong Consistency (Strict Quorum)} \iff R + W > N$$

```mermaid
flowchart LR
    subgraph QuorumOverlap["Quorum Intersection (N=3, W=2, R=2)"]
        W1[Node 1: Write ACK v2]
        W2[Node 2: Write ACK v2]
        R1[Node 2: Read v2]
        R2[Node 3: Read v1]

        W1 --- W2
        R1 --- R2
    end
```

Because $W=2$ and $R=2$ out of $N=3$, at least one node (Node 2) is guaranteed to be in both the write set and read set, ensuring the client reads the latest version $\text{v}2$.
