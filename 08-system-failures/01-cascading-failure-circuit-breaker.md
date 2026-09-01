# Production Outage Post-Mortem 01: Cascading Failure & Thread Pool Saturation

> **Severity**: SEV-1 Outage
> **Impacted Domain**: Distributed Reliability & Fault Tolerance
> **Post-Mortem Framework**: Cause $\to$ Symptoms $\to$ Detection $\to$ Mitigation $\to$ Recovery $\to$ Prevention

---

## 1. Incident Summary
A slow downstream recommendation microservice experienced high latency ($5\text{s}$), causing upstream API gateways to hold HTTP threads open indefinitely. Within 3 minutes, all 500 API Gateway worker threads saturated, causing complete 504 Gateway Timeout outages across the entire platform.

---

## 2. Root Cause Analysis
Synchronous blocking RPC calls without tight timeouts ($30\text{s}$ default) or Circuit Breakers. As downstream slowed, upstream caller threads piled up in the OS thread queue, starving CPU and memory until the ingress proxy crashed.

---

## 3. Symptoms & Blast Radius
- **HTTP 504 Gateway Timeouts**: $99.8\%$ failure rate on all public API endpoints.
- **Thread Pool Starvation**: Gateway active thread count spiked from $45$ to $500$ (max capacity).
- **Blast Radius**: Unrelated services (Payments, Login) suffered total outages due to shared gateway thread exhaustion.

---

## 4. Detection & Time to Detect (TTD)
- **Time to Detect (TTD)**: $2\text{ minutes}$ (Prometheus alert on Gateway HTTP 5xx error rate $> 5\%$).

---

## 5. Immediate Mitigation (Stop the Bleeding)
1. Operator tripped manual emergency kill-switch disabling recommendation widget calls.
2. Restarted API Gateway pods to flush blocked thread queues.

---

## 6. Full Recovery & Time to Recovery (TTR)
- **Time to Recovery (TTR)**: $14\text{ minutes}$ from initial alert.

---

## 7. Architectural Prevention
```mermaid
flowchart TD
    Client[Incoming Request] --> Gateway[API Gateway]
    Gateway --> CB{Resilience4j Circuit Breaker}
    CB -->|State: CLOSED (Normal)| Downstream[Downstream Recommendation Service]
    CB -->|Latency > 500ms on 50% Calls -> State: OPEN| FastFallback[Instant Fallback: Return Cached Trending Items ⚡ (2ms)]
```
- Configure **Resilience4j Circuit Breaker** on all inter-service RPC calls.
- Set strict read timeouts ($< 300\text{ms}$) on all downstream network clients.
- Separate thread pools (Bulkhead pattern) so slow non-critical widgets cannot exhaust gateway threads.

---

## 8. Code & Configuration Fix
```java
// Resilience4j Circuit Breaker & Bulkhead Configuration
@CircuitBreaker(name = "recommendationService", fallbackMethod = "getTrendingFallback")
@TimeLimiter(name = "recommendationService")
public CompletableFuture<List<Item>> getPersonalizedRecommendations(String userId) {
    return CompletableFuture.supplyAsync(() ->
        restTemplate.getForObject("http://reco-service/v1/items?user=" + userId, List.class)
    );
}

// Instant Fast Fallback
public CompletableFuture<List<Item>> getTrendingFallback(String userId, Throwable t) {
    return CompletableFuture.completedFuture(localCache.getTrendingItems());
}
```

---

## 9. Key Lessons Learned
1. Never make synchronous blocking network calls without a strict timeout ($< 500\text{ms}$).
2. Isolate critical and non-critical workloads using the Bulkhead pattern.
3. Always implement fast, graceful fallbacks for third-party or ancillary dependencies.
