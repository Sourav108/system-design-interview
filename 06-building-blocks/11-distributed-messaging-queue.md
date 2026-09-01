# Building Block 11: Distributed Messaging Queue (RabbitMQ & SQS)

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry

---

## 1. Problem
Synchronous HTTP/gRPC calls between microservices cause tight temporal coupling, cascading latency, and service crashes when downstream consumers experience traffic spikes.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
Message queues decouple producers from consumers asynchronously. Producers push messages to a durable queue; consumers pull/consume at their own processing speed with guaranteed delivery.

## 4. Mental Model
A physical postal mailbox where senders drop letters and postal workers process them at their own sustainable pace.

## 5. Core Concepts
Point-to-Point Queue, Producer/Consumer, Message Acknowledgments (`ACK`/`NACK`), Dead Letter Queue (DLQ), Prefetch Limits, Backpressure, Poison-Pill isolation, AMQP protocol.

## 6. Architecture
```mermaid
flowchart LR
    Producer[Producer Service] -->|Enqueue Message| Broker[Message Queue: RabbitMQ / SQS]
    Broker -->|Prefetch = 10| Worker1[Consumer Worker 1]
    Broker -->|Prefetch = 10| Worker2[Consumer Worker 2]
    Worker1 -->|Processed: ACK| Broker
    Worker2 -.->|Failed 3 Times: NACK .-> DLQ[(Dead Letter Queue)]
```

## 7. Request/Data Flow
1. Producer publishes message to broker. 2. Broker persists message to disk/RAM. 3. Dispatches message to available worker. 4. Worker processes message and sends `ACK`. 5. Broker deletes message. 6. On failure/timeout: worker sends `NACK` or broker re-enqueues.

## 8. Data Model
Message Envelope: `message_id (UUID)`, `payload (BYTE[])`, `headers (MAP)`, `retry_count (INT)`, `timestamp (INT64)`.

## 9. API Design
AMQP / AWS SQS API: `SendMessage(payload)`, `ReceiveMessage(visibility_timeout)`, `AckMessage(receipt_handle)`.

## 10. Algorithms
Fair Dispatch (Prefetch Count = 1), Exponential Backoff Retry with Dead-Letter routing.

## 11. Scaling
Scale consumers horizontally by adding worker nodes; scale broker clusters using Quorum Queues (Raft).

## 12. Partitioning
Queues partitioned across broker nodes using consistent hashing on routing keys.

## 13. Replication
RabbitMQ Quorum Queues use Raft consensus (replicated across 3 or 5 broker nodes).

## 14. Consistency
At-least-once delivery with message acknowledgments; consumers must implement idempotent processing.

## 15. Failure Scenarios
Consumer crash mid-processing (un-acked message automatically redelivered), Broker crash (Raft leader failover), Poison pill message.

## 16. Recovery
Dead Letter Queue isolates poison pills after 3 failed retries without blocking the primary queue.

## 17. Observability
Queue Depth (Messages Ready), Consumer Lag, Message Ingestion Rate (msg/s), Redelivery / DLQ Rate.

## 18. Security
TLS connection encryption, SASL authentication, queue-level IAM access policies.

## 19. Performance
Prefetch tuning (avoiding worker buffer starvation), batch acknowledgment, memory-resident queue buffers.

## 20. Trade-offs
Point-to-Point Queue (destructive read, single consumer per message) vs Pub/Sub (broadcast stream, multiple independent consumer groups).

## 21. When to Use
Asynchronous background tasks, video transcoding jobs, email/push dispatch, heavy image processing.

## 22. When NOT to Use
High-throughput event streaming where millions of events must be replayed or fanned out to multiple independent subscribers (use Kafka instead).

## 23. Implementation Strategy
Configure RabbitMQ with Spring Cloud Stream, enabling Quorum Queues and explicit manual acknowledgment with DLQ routing.

## 24. Practical Exercise
Implement a Java worker listening to a RabbitMQ queue with Testcontainers, simulate an unhandled exception, and verify automatic DLQ routing.

## 25. Interview Questions
1. How does RabbitMQ handle consumer backpressure? 2. What is the difference between an un-acked message and a dead-lettered message? 3. Why is prefetch limit tuning critical?

## 26. Common Mistakes
Setting prefetch limit to 0 (unbounded), causing a single slow worker to consume all queue memory and crash.

## 27. Quick Revision
Queue = Asynchronous decoupling -> Consumer pulls with prefetch -> ACK deletes -> Poison pills route to DLQ.

## 28. Related Building Blocks
`BB-12` (Pub/Sub / Kafka), `BB-23` (Notification Service), `BB-27` (Job Queue)

## 29. Related Case Studies
`CS-10` (Web Crawler), `CS-14` (CI/CD Deployment), `CS-16` (Online Judge)
