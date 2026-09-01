# Building Block 27: Distributed Job Queue (Work Stealing & Retries)

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry

---

## 1. Problem
Heavy computational operations (PDF report generation, video encoding, fraud detection scoring) block HTTP request threads, causing timeout errors for users.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
A Distributed Job Queue decouples job submission from background execution across a dedicated fleet of worker nodes, providing automatic retries, backpressure, progress tracking, and work-stealing concurrency.

## 4. Mental Model
A manufacturing factory conveyor belt where assembly line workers pull tasks from bins, process them, and log completions.

## 5. Core Concepts
Job State Machine (QUEUED -> RUNNING -> COMPLETED / FAILED), Visibility Timeout, Work Stealing, Exponential Backoff with Jitter, Heartbeats, Dead-Letter Queues, Priority Lanes.

## 6. Architecture
```mermaid
flowchart TD
    Client[Client App] -->|1. Submit Job: POST /v1/jobs| JobAPI[Job Ingestion API]
    JobAPI --> JobDB[(Job State DB: PostgreSQL)]
    JobAPI --> JobQueue[(Priority Broker: Redis / RabbitMQ)]

    subgraph WorkerFleet["Distributed Worker Fleet"]
        Worker1[Worker 1 (Pulls High Priority)]
        Worker2[Worker 2 (Work Stealing Pool)]
    end

    JobQueue --> Worker1
    JobQueue --> Worker2
    Worker1 -->|2. Heartbeat every 10s| JobDB
    Worker1 -->|3. Finished: State = COMPLETED| JobDB
```

## 7. Request/Data Flow
1. Client submits job payload. 2. Job saved in DB with state `QUEUED`. 3. Job ID enqueued in priority message queue. 4. Worker pulls job, updates state to `RUNNING`. 5. Worker streams heartbeats. 6. On success: state set to `COMPLETED`. 7. On error/timeout: retry with exponential backoff.

## 8. Data Model
Job Entity: `job_id (UUID)`, `type (STRING)`, `priority (1-10)`, `status (QUEUED/RUNNING/COMPLETED/FAILED)`, `payload (JSON)`, `attempts (INT)`, `max_retries (INT)`.

## 9. API Design
`POST /v1/jobs`, `GET /v1/jobs/{id}/status`, `POST /v1/jobs/{id}/cancel`.

## 10. Algorithms
Work Stealing Algorithm (idle workers steal tasks from busy queues), Exponential Backoff with Jitter.

## 11. Scaling
Scale job submissions via API Gateway + Redis; scale workers dynamically based on queue backlog depth.

## 12. Partitioning
Jobs partitioned by job type and priority level across broker queues.

## 13. Replication
Job metadata persisted in multi-AZ PostgreSQL with 3x replication.

## 14. Consistency
At-least-once execution; worker tasks must be idempotent to tolerate worker crash re-executions.

## 15. Failure Scenarios
Worker crash during execution (Visibility timeout expires -> job re-assigned to another worker), Poison pill payload crashing all workers.

## 16. Recovery
Zombie job reclamation via background sweeper checking expired heartbeats; DLQ isolation after max retry exhaustion.

## 17. Observability
Queue Backlog Depth, Job Wait Time in Queue, Job Execution Duration, Worker CPU Saturation, Retry Rate.

## 18. Security
Worker execution sandboxing, payload sanitization, restricting worker container network access.

## 19. Performance
Prefetching small batches of jobs into worker memory; zero-copy streaming for large file payloads.

## 20. Trade-offs
Dedicated Priority Lanes (High/Medium/Low: prevents low-priority batch jobs from starving critical user-facing tasks).

## 21. When to Use
Asynchronous batch processing, report generation, video transcoding, webhook dispatching, machine learning batch inference.

## 22. When NOT to Use
Low-latency synchronous request-response workflows (< 100ms).

## 23. Implementation Strategy
Build a distributed job processor in Java using Spring Boot, Redis Streams, and resilient thread pool workers with heartbeat tracking.

## 24. Practical Exercise
Implement a Java worker executing long-running jobs with periodic heartbeat updates, simulate worker crash (kill process), and verify job recovery.

## 25. Interview Questions
1. How does Visibility Timeout prevent duplicate execution in distributed job queues? 2. What is the Work-Stealing algorithm? 3. How do you prevent poison-pill jobs from bringing down an entire worker fleet?

## 26. Common Mistakes
Failing to implement heartbeats on long-running jobs, causing visibility timeout expiration and duplicate concurrent execution.

## 27. Quick Revision
Job Queue = Submit to queue -> Workers pull with visibility timeout -> Heartbeats track health -> DLQ catches poison pills.

## 28. Related Building Blocks
`BB-11` (Queue), `BB-17` (Task Scheduler), `BB-21` (Distributed Lock)

## 29. Related Case Studies
`CS-01` (YouTube), `CS-14` (CI/CD Deployment), `CS-16` (Online Judge)
