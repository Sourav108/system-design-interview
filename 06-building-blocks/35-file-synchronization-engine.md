# Building Block 35: File Synchronization Engine (Rsync & Merkle Trees)

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry / AI Infrastructure

---

## 1. Problem
Synchronizing multi-gigabyte files across devices (Dropbox / Google Drive) over flaky mobile networks wastes massive bandwidth and battery if the entire file is re-uploaded upon modifying a single line of text.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
A File Synchronization Engine divides files into content-defined chunks, detects modified blocks using rolling hashes (Rabin-Karp / Rsync algorithm) and Merkle Trees, and transfers only the mutated delta bytes.

## 4. Mental Model
Sending an editor only the specific paragraph you edited in a 500-page book rather than reprinting and mailing the entire book.

## 5. Core Concepts
Chunking (Fixed-Size vs Content-Defined Chunking / FastCDC), Rolling Hash (Rabin-Karp), Cryptographic Hash (SHA-256), Merkle Trees, Delta Sync, Deduplication, Conflict Resolution (Forking vs 3-Way Merge).

## 6. Architecture
```mermaid
flowchart TD
    LocalFile["Local File (Edited: 100MB)"] --> FastCDC[Content-Defined Chunking: FastCDC]
    FastCDC --> ChunkHashes["Chunk Hashes: [H1, H2, H3_new, H4]"]

    ChunkHashes --> MerkleCompare{Compare Local Merkle Tree with Cloud Merkle Tree}
    MerkleCompare -->|Chunks H1, H2, H4 match Cloud| Skip[Skip Transfer (Deduplicated! 0 Bytes)]
    MerkleCompare -->|Chunk H3_new is New| UploadDelta[Upload ONLY Chunk H3_new (2MB Delta) 🚀]

    UploadDelta --> CloudStorage[(Amazon S3 Blob Storage)]
    CloudStorage --> NotifyOtherDevices[WebSocket Sync Notification to Other Devices]
```

## 7. Request/Data Flow
1. Local file watcher detects edit. 2. File chunked using Content-Defined Chunking (FastCDC). 3. Computes SHA-256 for each chunk. 4. Queries metadata server with chunk hashes. 5. Server responds with missing hashes. 6. Client uploads ONLY missing chunks to S3. 7. Server commits new file version. 8. Pushes sync event to other user devices via WebSocket.

## 8. Data Model
File Metadata: `file_id (UUID)`, `version (INT64)`, `merkle_root (HEX)`, `chunks (ARRAY of {chunk_id, sha256, offset, size})`.

## 9. API Design
`POST /v1/files/commit_chunk_manifest { file_id, version, chunk_hashes: [...] }`.

## 10. Algorithms
FastCDC (Fast Content-Defined Chunking with gear hashing), Rabin-Karp Rolling Hash, Merkle Tree diffing.

## 11. Scaling
Scale chunk storage on S3; scale metadata catalog on CockroachDB / Cassandra; scale sync notifications via WebSocket Gateway.

## 12. Partitioning
File metadata partitioned by `hash(user_id)` or `hash(file_id)`.

## 13. Replication
Chunks stored with 11 Nines durability on S3; metadata replicated across 3 Availability Zones.

## 14. Consistency
Strong consistency per file version; optimistic concurrency control (`version_id`).

## 15. Failure Scenarios
Simultaneous offline edits on two devices (creates conflict copy `filename (conflicted copy).pdf`), Network disconnect mid-sync.

## 16. Recovery
Chunk-level resumable uploads; automatic conflict copy creation on concurrent multi-device writes.

## 17. Observability
Sync Latency (p99 < 1s), Bandwidth Saved via Deduplication (Target > 80%), Chunk Ingestion Rate, Active WebSocket sync connections.

## 18. Security
Client-side envelope encryption (zero-knowledge encryption before chunk upload), TLS 1.3.

## 19. Performance
Content-Defined Chunking ensures that inserting bytes at the beginning of a file only changes 1 chunk without shifting subsequent chunk boundaries.

## 20. Trade-offs
Content-Defined Chunking (Robust to byte insertions, compute overhead) vs Fixed-Size Chunking (Fast, breaks chunk alignment on byte shift).

## 21. When to Use
Cloud storage sync (Dropbox, Google Drive, OneDrive), backup systems, container layer distribution (Docker image layers).

## 22. When NOT to Use
Append-only streaming logs where sequential chunking is unnecessary.

## 23. Implementation Strategy
Implement a content-defined chunking and Merkle tree comparison engine in Java with SHA-256 hashing to calculate sync deltas.

## 24. Practical Exercise
Write a Java test modifying a single character inside a 10MB text file, verify that FastCDC identifies exactly 1 modified chunk (256KB) and skips uploading the other 9.75MB.

## 25. Interview Questions
1. Why does Fixed-Size chunking fail when bytes are inserted at the beginning of a file? 2. How does Content-Defined Chunking (FastCDC) solve the boundary shift problem? 3. How does Dropbox resolve concurrent offline edit conflicts across two devices?

## 26. Common Mistakes
Using Fixed-Size chunking for document sync, causing 100% of chunks to change when a single byte is inserted at the top of a file.

## 27. Quick Revision
File Sync = Content-Defined Chunking (FastCDC) -> SHA-256 Merkle tree diff -> Upload ONLY modified chunks -> 80%+ bandwidth saved.

## 28. Related Building Blocks
`BB-14` (Blob Store / S3), `BB-37` (WebSocket Gateway), `BB-30` (Audit Log)

## 29. Related Case Studies
`CS-13` (Google Docs), `CS-14` (CI/CD Deployment), `CS-08` (Instagram)
