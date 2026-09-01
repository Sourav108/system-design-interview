# Building Block 17: Distributed Task Scheduler

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry

---

## 1. Problem
Applications need to schedule and execute millions of delayed or recurring tasks (e.g. billing subscription renewal after 30 days, reminder notifications, periodic report generation) reliably across worker clusters.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
A single server's `cron` or in-memory timer fails when the server crashes, cannot coordinate across distributed workers, and cannot handle millions of dynamic delayed tasks.

## 4. Mental Model
An enterprise air traffic control scheduling system managing departures, delayed takeoffs, and gate assignments across a fleet of airplanes.

## 5. Core Concepts
Hierarchical Timing Wheels, Priority Delay Queue, Distributed Worker Stealing, Redis Sorted Sets (`ZSET`), Lock-Free Concurrency, Idempotent Execution, Task Heartbeats.

## 6. Architecture
```mermaid
flowchart TD
    Client[Application Client] -->|Schedule Task: runAt = T + 3600| SchedulerAPI[Task Scheduler API]
    SchedulerAPI --> TaskStore[(Distributed Task DB: PostgreSQL / Redis ZSET)]

    subgraph WorkerFleet["Distributed Worker Fleet"]
        Coordinator[Leader Coordinator Node]
        Worker1[Task Worker Instance 1]
        Worker2[Task Worker Instance 2]
    end

    Coordinator -->|Polls Due Tasks: score <= now()| TaskStore
    Coordinator -->|Dispatches Task via Kafka/Queue| Worker1
    Worker1 -->|Heartbeat / State = COMPLETED| TaskStore
```

## 7. Request/Data Flow
1. Client schedules task with `execution_time`. 2. Stored in Redis Sorted Set where `score = timestamp`. 3. Coordinator polls tasks where `score <= now()`. 4. Acquires distributed lock / transitions state to `RUNNING`. 5. Dispatches to worker. 6. Worker executes and marks `COMPLETED`.

## 8. Data Model
Task Entity: `task_id (UUID)`, `execution_time (INT64)`, `payload (JSON)`, `status (SCHEDULED/RUNNING/COMPLETED/FAILED)`, `retry_count (INT)`.

## 9. API Design
`POST /v1/tasks/schedule { execution_time: 1672531200, payload: {...} }`, `DELETE /v1/tasks/{id}`.

## 10. Algorithms
Hierarchical Timing Wheel ($\mathcal{O}(1)$ insertion and expiration), Redis `ZREMRANGEBYSCORE` atomic polling.

## 11. Scaling
Scale task storage via partitioned Redis ZSETs; scale task execution by adding stateless worker nodes.

## 12. Partitioning
Tasks partitioned by `hash(task_id)` or time bucket (`bucket_hour = timestamp / 3600`).

## 13. Replication
Tasks persisted to durable relational database with synchronous multi-AZ replication.

## 14. Consistency
At-least-once task execution. Tasks must be idempotent to tolerate worker crash redeliveries.

## 15. Failure Scenarios
Worker crash mid-task (detected via missing heartbeat after 60s; re-queued), Coordinator node crash (new leader elected).

## 16. Recovery
Sweeper process reclaims orphan `RUNNING` tasks whose heartbeat expired and re-schedules them.

## 17. Observability
Scheduled Tasks Count, Tasks Due Lag (Delay between scheduled time and actual start), Worker CPU Saturation, Task Failure Rate.

## 18. Security
Task payload validation, execution sandbox isolation, authorization checks on task submission.

## 19. Performance
Hierarchical Timing Wheels allow $\mathcal{O}(1)$ task scheduling compared to $\mathcal{O}(\log N)$ priority heaps.

## 20. Trade-offs
Fine-Grained Dynamic Delays (Redis ZSET / Timing Wheel) vs Coarse Recurring Batches (Quartz / Temporal).

## 21. When to Use
Subscription renewals, delayed email/SMS reminders, order payment expiration timers, asynchronous webhook retries.

## 22. When NOT to Use
Real-time low-latency inter-process messaging ($< 5\text{ms}$) where direct Kafka streams should be used.

## 23. Implementation Strategy
Implement a distributed delay scheduler in Java using Redis Sorted Sets with atomic Lua scripts for polling and lock acquisition.

## 24. Practical Exercise
Write a Java task scheduler polling a Redis ZSET across 4 worker threads, verifying that tasks execute within 10ms of scheduled time.

## 25. Interview Questions
1. How does a Hierarchical Timing Wheel achieve $\mathcal{O}(1)$ complexity? 2. How do you prevent two workers from executing the same delayed task concurrently? 3. What is task heartbeat fencing?

## 26. Common Mistakes
Polling a SQL database with `SELECT * FROM tasks WHERE run_at <= NOW()` without indexing or row locking (causes table locks and duplicate runs).

## 27. Quick Revision
Task Scheduler = Redis ZSET (score = time) -> Leader polls due tasks -> Dispatches to worker pool -> Heartbeat ensures fault recovery.

## 28. Related Building Blocks
`BB-11` (Queue), `BB-21` (Distributed Lock), `BB-27` (Job Queue)

## 29. Related Case Studies
`CS-10` (Web Crawler), `CS-14` (CI/CD Deployment), `CS-15` (Payment Gateway)
