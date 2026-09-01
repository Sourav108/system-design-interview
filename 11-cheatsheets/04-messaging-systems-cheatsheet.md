# Messaging Systems & Queues Comparison

| Feature | Apache Kafka | RabbitMQ | AWS SQS | Redis Streams |
|---|---|---|---|---|
| **Model** | Distributed Append-Only Commit Log | Message Broker (Smart broker, dumb consumer) | Fully Managed Cloud Queue | In-Memory Stream Log |
| **Consumption** | Pull-based (Consumer Groups read offsets) | Push-based (Broker pushes to workers) | Pull-based (`ReceiveMessage`) | Pull-based Consumer Groups |
| **Message Deletion** | Retained for days/weeks (Replayable) | Deleted immediately upon `ACK` | Deleted after receipt handle ACK | Retained until trimmed (`XTRIM`) |
| **Throughput** | **Millions msg/sec** (Zero-copy I/O) | $50\text{k} - 100\text{k msg/sec}$ | Moderate (Auto-scales) | **Millions msg/sec** (In-memory) |
| **Ordering** | Strict per partition key | Strict per single queue | Best-effort (or FIFO queue) | Strict per stream ID |
| **Best Use Case** | Event streaming, CDC, analytics, Kafka Streams | Complex task routing, RPC, background jobs | Serverless cloud task dispatch | Low-latency in-memory event streaming |
