# Replication Strategies & Consensus Protocols

| Strategy | Architecture | Write Path | Read Path | Pros & Cons |
|---|---|---|---|---|
| **Single-Leader** | 1 Master + N Replicas | All writes go to Leader | Reads served by Leader or Replicas | Simple, no write conflicts; Leader is SPOF for writes |
| **Multi-Leader** | N Masters (Multi-Region) | Writes accepted by local datacenter Master | Local reads | Low write latency globally; complex conflict resolution (CRDT/LWW) |
| **Leaderless** | Symmetric Quorum | Writes sent to all $N$ nodes ($W$ must ack) | Reads sent to $R$ nodes ($R + W > N$) | Highly available; requires Read Repair & Hinted Handoff |

---

## Consensus Protocols
- **Raft**: Understandable consensus protocol based on Leader Election + Log Replication + Safety invariants. Used in etcd, Consul, KRaft.
- **Paxos**: Classic distributed consensus algorithm. Used in Google Chubby and Spanner.
- **ZAB**: Zookeeper Atomic Broadcast. Primary-backup atomic broadcast protocol.
