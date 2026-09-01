# System Reliability Patterns

| Pattern | Problem Addressed | Core Defense Mechanism |
|---|---|---|
| **Circuit Breaker** | Cascading failure from slow downstream | Trips to OPEN state upon error threshold, fast-failing traffic |
| **Bulkhead** | Thread pool starvation across microservices | Isolates compute/thread resources into bounded independent pools |
| **Rate Limiter** | Traffic spikes & DDoS saturation | Throttles requests via Token Bucket / Sliding Window algorithms |
| **Exponential Backoff + Jitter** | Retry storms on recovering services | Multiplies backoff delay exponentially with randomized full jitter |
| **Idempotency Keys** | Duplicate financial transactions | Atomic `SET NX` locks deduplicate repeated client requests |
| **Dead-Letter Queue (DLQ)** | Poison pill unparseable payloads | Isolates failed messages to prevent infinite consumer crash loops |
| **Fencing Tokens** | Zombie leader writes after GC pauses | Monotonically increasing epoch counters reject stale commands |
