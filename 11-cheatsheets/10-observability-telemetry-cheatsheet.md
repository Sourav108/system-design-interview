# Observability & Telemetry (The 3 Pillars + Golden Signals)

## 📊 The 3 Pillars of Observability
1. **Metrics**: Aggregatable numeric time-series ($p50, p95, p99$, counters, gauges). Low storage cost.
2. **Logs**: Structured JSON events recording execution details. High storage cost; forensic debugging.
3. **Traces**: Distributed request propagation tracking parent-child spans via `trace_id` (OpenTelemetry).

---

## 🚦 The 4 Golden Signals (Google SRE)
- **Latency**: Time taken to service a request (Distinguish between successful 200s and failed 500s).
- **Traffic**: Measure of demand on the system (HTTP QPS, Network I/O, concurrent sessions).
- **Errors**: Rate of requests that fail (HTTP 5xx, uncaught exceptions, circuit breaker trips).
- **Saturation**: How full the service is (CPU %, RAM %, DB Connection Pool utilization, Disk IOPS).
