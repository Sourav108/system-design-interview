# Mock Interview 02: Staff Engineer — Design a Multi-Region Real-Time Ride Matching Platform (Uber)

> **Candidate Level**: Staff Engineer
> **Interviewer**: Principal Systems Architect
> **Duration**: 45 Minutes
> **Target Outcome**: Strong Hire

---

## Transcript & Commentary

### 1. High-Level Architecture & Ingestion Bottlenecks
- **Candidate**: "With 5M active drivers streaming GPS every 4s, we have 1.25M writes/sec. Writing raw GPS pings to disk is an anti-pattern. We ingest GPS pings via Netty WebSocket Gateways, buffer in Kafka partitioned by city ID, and update an in-memory Redis Google S2 geospatial index (Level 13 cells)."
- **Interviewer**: "How do you prevent two riders from matching the same driver concurrently?"
- **Candidate**: "We use Redlock distributed locking with a 15-second lease on the `driver_id`. The matching engine acquires the lock atomically before dispatching the offer. Storage mutations enforce monotonic fencing tokens to prevent zombie writes."

### 2. Dynamic Surge Pricing Pipeline
- **Candidate**: "Dynamic surge pricing is computed per S2 cell in real time using Apache Flink sliding windows over supply (available drivers in cell) vs demand (ride requests in cell), caching the surge multiplier in Redis with a 30s TTL."
- **Interviewer**: *[Excellent staff-level domain depth: connects geospatial primitives, streaming state, and concurrency locks.]*
