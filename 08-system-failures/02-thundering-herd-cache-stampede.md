# Production Outage Post-Mortem 02: Thundering Herd & Cache Stampede

> **Severity**: SEV-1 Outage
> **Impacted Domain**: Distributed Reliability & Fault Tolerance
> **Post-Mortem Framework**: Cause $\to$ Symptoms $\to$ Detection $\to$ Mitigation $\to$ Recovery $\to$ Prevention

---

## 1. Incident Summary
A viral celebrity post's cache key expired simultaneously for 200,000 active viewers. 50,000 concurrent requests experienced a cache miss within 500ms and simultaneously hit the primary PostgreSQL database, causing connection pool exhaustion and database CPU spike to 100%.

---

## 2. Root Cause Analysis
Uniform fixed TTL ($3600\text{s}$) on hot cache keys without concurrency locks (Singleflight) or probabilistic early expiration (XFetch).

---

## 3. Symptoms & Blast Radius
- **Database CPU**: Spiked from $15\%$ to $100\%$ in 2 seconds.
- **Database Connection Pool**: All 1,000 PgBouncer connections saturated.
- **HTTP 500 Internal Server Errors**: 85,000 failed user requests in 60 seconds.

---

## 4. Detection & Time to Detect (TTD)
- **Time to Detect (TTD)**: $45\text{ seconds}$ (Database connection pool exhaustion alert).

---

## 5. Immediate Mitigation (Stop the Bleeding)
1. Manually warmed the viral post key in Redis with a 24-hour TTL.
2. Temporarily enabled query caching on PgBouncer.

---

## 6. Full Recovery & Time to Recovery (TTR)
- **Time to Recovery (TTR)**: $8\text{ minutes}$.

---

## 7. Architectural Prevention
```mermaid
flowchart TD
    Reqs[50,000 Concurrent Requests] --> CacheCheck{Redis Cache Miss}
    CacheCheck --> Mutex{Singleflight Mutex Lock}
    Mutex -->|1 Single Request| FetchDB[(Query PostgreSQL DB)]
    Mutex -.->|Remaining 49,999 Reqs Wait for Lock .-> ReadRedis[Read from Redis once populated ✅]
    FetchDB --> Populate[Write to Redis with Jittered TTL]
    Populate --> ReadRedis
```
- Implement **Singleflight Mutex Locking** (only 1 thread queries DB on miss; all other threads wait and read cached result).
- Add **Jittered TTLs** (randomize TTL $3600\text{s} \pm \text{rand}(0, 300\text{s})$) to prevent synchronized expirations.
- Implement **XFetch Probabilistic Early Expiration** (background thread asynchronously refreshes cache before expiration).

---

## 8. Code & Configuration Fix
```java
public class SingleflightCache<K, V> {
    private final ConcurrentHashMap<K, CompletableFuture<V>> inFlight = new ConcurrentHashMap<>();

    public V get(K key, Function<K, V> dbLoader) {
        V cached = redis.get(key);
        if (cached != null) return cached;

        // Singleflight: Only 1 thread executes loader
        CompletableFuture<V> future = inFlight.computeIfAbsent(key, k ->
            CompletableFuture.supplyAsync(() -> {
                try {
                    V val = dbLoader.apply(k);
                    int jitteredTtl = 3600 + ThreadLocalRandom.current().nextInt(300);
                    redis.setex(k, jitteredTtl, val);
                    return val;
                } finally {
                    inFlight.remove(k);
                }
            })
        );
        return future.join();
    }
}
```

---

## 9. Key Lessons Learned
1. Never let multiple concurrent threads query the database for the same missing cache key.
2. Always add random jitter to cache TTLs.
3. Use probabilistic early expiration (XFetch) for mission-critical hot keys.
