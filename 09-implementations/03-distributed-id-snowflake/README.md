# Implementation 03: Distributed ID Sequencer (Twitter Snowflake)

A high-throughput, thread-safe, 64-bit **Snowflake ID Generator** in Java 21 capable of generating **4,096,000 unique IDs per second per node** with NTP clock drift protection.

---

## 🏗️ 64-Bit Bitwise Layout

```
 1 bit  |  41 bits (Timestamp in ms)  | 10 bits (Worker ID) | 12 bits (Sequence)
 [ 0 ]  | [ 0000000000...000000000 ]   | [ 0000000000 ]      | [ 000000000000 ]
```

---

## 💻 Core Implementation

```java
public class SnowflakeIdGenerator {
    private static final long EPOCH = 1704067200000L; // 2024-01-01
    private static final long WORKER_ID_BITS = 10L;
    private static final long SEQUENCE_BITS = 12L;
    private static final long MAX_SEQUENCE = (1L << SEQUENCE_BITS) - 1;

    private final long workerId;
    private long lastTimestamp = -1L;
    private long sequence = 0L;

    public SnowflakeIdGenerator(long workerId) {
        this.workerId = workerId;
    }

    public synchronized long nextId() {
        long timestamp = System.currentTimeMillis();

        if (timestamp < lastTimestamp) {
            long offset = lastTimestamp - timestamp;
            if (offset <= 5) {
                try { Thread.sleep(offset << 1); } catch (Exception e) {}
                timestamp = System.currentTimeMillis();
            } else {
                throw new IllegalStateException("Clock moved backwards! Refusing ID generation for " + offset + "ms");
            }
        }

        if (timestamp == lastTimestamp) {
            sequence = (sequence + 1) & MAX_SEQUENCE;
            if (sequence == 0) {
                while ((timestamp = System.currentTimeMillis()) <= lastTimestamp);
            }
        } else {
            sequence = 0L;
        }

        lastTimestamp = timestamp;
        return ((timestamp - EPOCH) << 22) | (workerId << 12) | sequence;
    }
}
```
