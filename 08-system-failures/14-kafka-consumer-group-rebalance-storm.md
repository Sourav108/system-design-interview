# Production Outage Post-Mortem 14: Kafka Consumer Rebalance Storm & Processing Halt

> **Severity**: SEV-1 Outage
> **Impacted Domain**: Distributed Reliability & Fault Tolerance
> **Post-Mortem Framework**: Cause $\to$ Symptoms $\to$ Detection $\to$ Mitigation $\to$ Recovery $\to$ Prevention

---

## 1. Incident Summary
A batch of large PDF files caused consumer workers to take 6 minutes to process a message batch. Because `max.poll.interval.ms` was set to 5 minutes, Kafka assumed the workers were dead, triggered a consumer group rebalance, revoked partitions, and assigned them to other workers—which also took 6 minutes, causing an infinite continuous rebalance storm.

---

## 2. Root Cause Analysis
Long-running CPU-bound message processing blocking the consumer poll loop beyond `max.poll.interval.ms`.

---

## 3. Symptoms & Blast Radius
- **Processing Paralyzed**: 0 messages processed for 2 hours; consumer group stuck in infinite rebalance loop.
- **Kafka Broker CPU**: Spiked due to continuous group coordinator rebalance metadata traffic.

---

## 4. Detection & Time to Detect (TTD)
- **Time to Detect (TTD)**: $15\text{ minutes}$ (Consumer group rebalance rate alert).

---

## 5. Immediate Mitigation (Stop the Bleeding)
1. Increased `max.poll.interval.ms` to 15 minutes and reduced `max.poll.records` to 5.

---

## 6. Full Recovery & Time to Recovery (TTR)
- **Time to Recovery (TTR)**: $40\text{ minutes}$.

---

## 7. Architectural Prevention
```mermaid
flowchart TD
    Poll[Consumer Thread: poll records] --> HandOff[Hands off batch to Worker Thread Pool]
    HandOff --> FastPoll[Consumer Thread Immediately Polls Kafka Again (Never Exceeds Timeout!) ✅]
    WorkerPool[Worker Thread Pool executes heavy 6-minute PDF tasks]
```
- Decouple **Kafka polling thread** from **worker processing threads** (poll records and immediately dispatch to an internal `ThreadPoolExecutor`).
- Tune `max.poll.records = 10` and increase `max.poll.interval.ms = 900000` (15 mins).
- Use **Cooperative Sticky Assignor** (`CooperativeStickyAssignor`) to avoid stop-the-world full group rebalances.

---

## 8. Code & Configuration Fix
```java
// Kafka Consumer Configuration
props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 10);
props.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 900000); // 15 mins
props.put(ConsumerConfig.PARTITION_ASSIGNMENT_STRATEGY_CONFIG,
    org.apache.kafka.clients.consumer.CooperativeStickyAssignor.class.getName());
```

---

## 9. Key Lessons Learned
1. Never do heavy long-running computation directly on the Kafka consumer polling thread.
2. Use the Cooperative Sticky Assignor to prevent full cluster stop-the-world rebalances.
3. Balance `max.poll.records` with processing time.
