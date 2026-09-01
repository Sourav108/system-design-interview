# Observability: Metrics, Logs & Distributed Tracing

Observability is the degree to which you can infer the internal state and root causes of anomalies in a distributed system based solely on its external telemetry outputs.

---

## 1. The 3 Pillars of Observability

```mermaid
flowchart TD
    Obs[Observability Telemetry]
    Obs --> M[1. Metrics: Aggregated numeric time-series counters and gauges: Prometheus]
    Obs --> L[2. Structured Logs: Detailed contextual event payloads: JSON / ELK / Loki]
    Obs --> T[3. Distributed Tracing: End-to-end request lifecycle graphs: OpenTelemetry / Jaeger]
```

---

## 2. The RED Method for Microservices

For every microservice endpoint, track the **RED** metrics:
- **R (Rate)**: Requests per second received by the service.
- **E (Errors)**: Number of requests failing per second (HTTP 5xx / exceptions).
- **D (Duration)**: Time taken to process each request (p50, p95, p99 latency histograms).

---

## 3. Distributed Tracing Architecture

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant GW as API Gateway (TraceID: abc-123, SpanID: s1)
    participant Auth as Auth Service (SpanID: s2, ParentSpan: s1)
    participant Order as Order Service (SpanID: s3, ParentSpan: s1)
    participant DB as Postgres (SpanID: s4, ParentSpan: s3)

    Client->>GW: Request (TraceContext injected into HTTP headers)
    GW->>Auth: Verify Token (propagates traceparent header)
    Auth-->>GW: Token Valid
    GW->>Order: Create Order
    Order->>DB: INSERT INTO orders
    DB-->>Order: OK
    Order-->>GW: 201 Created
    GW-->>Client: 201 Created
```
