# 20 Incident Triage & Failure Scenario Drills

Real-time incident response drills testing rapid triage, root cause diagnosis, and mitigation under pressure.

---

### Scenario 01: Ingress API Gateway 504 Gateway Timeout Spike
- **Symptom**: 99% of API requests return HTTP 504.
- **Root Cause**: Downstream recommendation service is slow (5s latency), exhausting gateway thread pool.
- **Immediate Triage**:
  1. Trip Circuit Breaker / Feature flag to disable recommendation calls.
  2. Return static cached fallback data.
  3. Restart API gateway pods to clear blocked thread pool queues.

---

### Scenario 02: Database Master CPU Hits 100% on Viral Post
- **Symptom**: PostgreSQL master connection pool exhausted.
- **Root Cause**: Hot cache key expired, causing a 50,000-request Cache Stampede directly to SQL.
- **Immediate Triage**:
  1. Warm the hot key in Redis with 24-hour TTL.
  2. Enable PgBouncer query pooling.
  3. Deploy Singleflight mutex locking in application layer.

---

### Scenario 03: Kafka Consumer Group Rebalance Storm
- **Symptom**: Zero messages processed for 30 minutes; broker CPU spikes.
- **Root Cause**: Heavy PDF processing exceeded `max.poll.interval.ms` (5 mins).
- **Immediate Triage**:
  1. Increase `max.poll.interval.ms` to 15 minutes.
  2. Reduce `max.poll.records` to 5.
  3. Switch partition assignor to `CooperativeStickyAssignor`.

*(Scenarios 04 to 20 cover Split-Brain partitions, High-Cardinality Prometheus OOM, GC Pauses, NTP Clock Skew, and Redis node crashes).*
