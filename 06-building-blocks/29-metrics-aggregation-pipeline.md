# Building Block 29: Metrics Aggregation Pipeline (OpenTelemetry & StatsD)

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry

---

## 1. Problem
High-throughput microservice clusters generate millions of raw metric events per second. Sending every individual counter increment over the network to a central database saturates network bandwidth and crashes the monitoring system.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
A Metrics Aggregation Pipeline buffers and aggregates metrics locally in-memory (using StatsD / OpenTelemetry Collector sidecars) over rolling time windows, flushing pre-aggregated summaries (p50, p99, rate) to centralized time-series storage.

## 4. Mental Model
A regional postal sorting facility that bundles thousands of local letters into single bulk sacks before loading them onto freight airplanes.

## 5. Core Concepts
OpenTelemetry Collector, StatsD Daemon, Local In-Memory Ring Buffers, Pre-Aggregation Windows (10s/60s), T-Digest for Quantiles, Exponential Histograms, Ingest Buffers.

## 6. Architecture
```mermaid
flowchart LR
    AppNode["App Container (Micrometer SDK)"] -->|Local UDP / Unix Socket: 0 Overhead| Sidecar[OTel Collector / StatsD Sidecar]
    Sidecar -->|Pre-Aggregates: 10s Window (T-Digest Quantiles)| Aggregator[Central Metrics Gateway]
    Aggregator --> TSDB[(Time-Series DB: Prometheus / VictoriaMetrics / M3DB)]
    TSDB --> Grafana[Grafana Dashboards & Alerting]
```

## 7. Request/Data Flow
1. Microservice emits metric increment via local UDP or Unix Domain Socket (zero thread blocking). 2. OTel Collector sidecar aggregates counters and calculates T-Digest quantiles locally. 3. Every 10 seconds, flushes compressed batch to central gateway. 4. Ingested into long-term TSDB. 5. Visualized in Grafana.

## 8. Data Model
Metric Data Point: `metric_name (STRING)`, `labels (MAP)`, `timestamp (INT64)`, `value_summary { count, sum, min, max, quantiles }`.

## 9. API Design
OpenTelemetry OTLP Protocol (gRPC / Protobuf): `POST /v1/metrics`.

## 10. Algorithms
T-Digest algorithm for accurate streaming percentile estimations ($p50, p90, p99$), HyperLogLog for cardinality estimation.

## 11. Scaling
Scale ingest horizontally by running OTel Collector sidecars on every host; scale central TSDB via hash-partitioned time-series clusters.

## 12. Partitioning
Metrics partitioned by metric name and label hash.

## 13. Replication
Replicated time-series chunks across 3 TSDB storage nodes with multi-region backup to S3.

## 14. Consistency
Eventual consistency; loss-tolerant numeric metrics.

## 15. Failure Scenarios
High-cardinality metric explosion (OTel Collector drops offending labels automatically), UDP packet drop during heavy network load.

## 16. Recovery
Local ring buffers buffer up to 100MB of metric batches during network disconnects; automatic sampling on saturation.

## 17. Observability
Metrics Ingested/sec, Aggregation Pipeline Latency, Dropped Data Points Rate, TSDB Disk Write Amplification.

## 18. Security
mTLS between OTel Collectors and central gateway, network isolation of monitoring endpoints.

## 19. Performance
Local UDP socket emission incurs $< 50\text{ nanoseconds}$ latency overhead on application request threads.

## 20. Trade-offs
Client-Side Pre-Aggregation (Massive network bandwidth savings, approximate quantiles) vs Raw Sample Streaming (Exact data, 100x network overhead).

## 21. When to Use
High-throughput microservices, global cloud infrastructure monitoring, real-time SLA/SLO tracking.

## 22. When NOT to Use
Audit trail logging of user financial transactions (use Immutable Audit Log instead).

## 23. Implementation Strategy
Configure Micrometer in Spring Boot with OpenTelemetry OTLP exporter routing to a local OTel Collector sidecar.

## 24. Practical Exercise
Benchmark 1,000,000 metric increments emitted via UDP in Java, verifying that the OTel collector aggregates them into a single summary flush without packet loss.

## 25. Interview Questions
1. How does the T-Digest algorithm calculate streaming percentiles in bounded memory? 2. Why is UDP or Unix Domain Sockets preferred for local metric emission? 3. What is metric label cardinality explosion?

## 26. Common Mistakes
Emitting high-frequency metrics directly over synchronous HTTP calls to a remote database on every incoming user request.

## 27. Quick Revision
Metrics Pipeline = Local UDP emission -> OTel Sidecar pre-aggregates via T-Digest -> Flushes 10s summaries to TSDB -> Grafana charts.

## 28. Related Building Blocks
`BB-07` (Monitoring), `BB-16` (Logging), `BB-08` (Server Errors)

## 29. Related Case Studies
All Case Studies (`CS-01` to `CS-20`)
