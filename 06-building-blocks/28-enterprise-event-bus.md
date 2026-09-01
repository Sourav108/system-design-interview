# Building Block 28: Enterprise Event Bus (Schema Registry & CloudEvents)

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry

---

## 1. Problem
In microservice architectures, evolving event schemas (adding/removing JSON fields) silently breaks downstream consumer services, causing deserialization crashes and corrupted business logic.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
An Enterprise Event Bus enforces strict, versioned event contracts using a central Schema Registry (Avro/Protobuf) and standardized metadata envelopes (CloudEvents), guaranteeing backward and forward compatibility.

## 4. Mental Model
A formal international treaty defining standard passport and visa specifications that all border control checkpoints can reliably verify.

## 5. Core Concepts
Schema Registry, Avro / Protocol Buffers Binary Serialization, Schema Compatibility Modes (BACKWARD, FORWARD, FULL), CloudEvents Specification, Event Routing, Dead-Letter Schema Topics.

## 6. Architecture
```mermaid
flowchart TD
    Producer[Producer Microservice] -->|1. Register / Validate Schema v2| Registry[Confluent Schema Registry]
    Producer -->|2. Serializes Binary Avro Payload (5x smaller than JSON)| Kafka[(Kafka Event Bus: Topic 'order-events')]

    Kafka --> Consumer[Consumer Microservice]
    Consumer -->|3. Fetches Schema v2 by Schema ID (Cached)| Registry
    Consumer -->|4. Deserializes Avro Binary Payload ✅| Process[Execute Business Logic]
```

## 7. Request/Data Flow
1. Producer defines schema in Avro/Protobuf IDL. 2. Registers schema with Schema Registry; registry validates compatibility. 3. Producer serializes event to binary with 5-byte header (Magic Byte + Schema ID). 4. Publishes to Kafka. 5. Consumer retrieves Schema ID from registry (caches locally) and deserializes binary payload.

## 8. Data Model
CloudEvent Envelope: `id (UUID)`, `source (URI)`, `specversion (1.0)`, `type (com.example.order.created)`, `data (AVRO/PROTOBUF)`.

## 9. API Design
Schema Registry REST API: `POST /subjects/{subject}/versions`, `GET /schemas/ids/{id}`.

## 10. Algorithms
Schema Evolution Compatibility Checking (Validating default values on new fields for Backward compatibility).

## 11. Scaling
Scale event throughput on Kafka; scale Schema Registry via local in-memory client caching (99.99% cache hit).

## 12. Partitioning
Schemas partitioned by Subject Name: `{topic-name}-value`.

## 13. Replication
Schema Registry stores schemas in a compact, replicated Kafka topic (`_schemas`) with Raft leader election.

## 14. Consistency
Strong consistency on schema registrations; immutable schema IDs.

## 15. Failure Scenarios
Schema evolution incompatibility rejected at registration time; unparseable payload routed to Schema Dead-Letter Topic.

## 16. Recovery
Schema rollback protection; local disk cache of registered schemas if registry is temporarily unreachable.

## 17. Observability
Schema Registration Rate, Schema Cache Hit Ratio (> 99.9%), Event Serialization/Deserialization Latency (p99 < 0.1ms).

## 18. Security
Schema Registry ACLs, RBAC on schema modification, TLS encryption for schema registry HTTP traffic.

## 19. Performance
Avro/Protobuf binary serialization is $5\times - 10\times$ smaller than verbose JSON and $10\times$ faster to deserialize.

## 20. Trade-offs
Strict Binary Schema (High performance, contract safety, requires registry) vs Loose JSON (Flexible, zero registry, prone to breaking changes).

## 21. When to Use
Core enterprise event streams, financial transaction events, inter-microservice choreography, data lakehouse CDC ingestion.

## 22. When NOT to Use
Simple monolithic applications or internal one-off prototypes.

## 23. Implementation Strategy
Configure Spring Boot Kafka with Confluent Schema Registry, Avro Maven plugin, and `FULL` compatibility mode.

## 24. Practical Exercise
Define an Avro schema for an Order event, generate Java classes, add a new field with a default value, and test backward-compatible consumption.

## 25. Interview Questions
1. Explain BACKWARD vs FORWARD vs FULL schema compatibility. 2. How does the 5-byte Avro wire format header work? 3. Why is binary Avro superior to JSON for high-throughput event streaming?

## 26. Common Mistakes
Adding a new mandatory field without a default value in a schema evolution step (instantly breaks all existing consumers!).

## 27. Quick Revision
Event Bus = CloudEvents envelope + Avro binary serialization + Schema Registry compatibility enforcement = Zero breaking changes.

## 28. Related Building Blocks
`BB-12` (Kafka), `BB-11` (Queue), `BB-30` (Audit Log)

## 29. Related Case Studies
`CS-05` (Uber), `CS-06` (Twitter / X), `CS-15` (Payment Gateway)
