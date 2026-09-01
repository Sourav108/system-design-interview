# Implementation 09: Resilience4j Circuit Breaker & Bulkhead

A resilient microservice integration utilizing **Resilience4j** to protect against cascading failures using Circuit Breakers, Bulkhead thread isolation, and TimeLimiter timeouts.

---

## 💻 Core Implementation

```java
@Service
public class ResilientProductService {
    @CircuitBreaker(name = "productService", fallbackMethod = "getCachedProduct")
    @Bulkhead(name = "productService", type = Bulkhead.Type.THREADPOOL)
    @TimeLimiter(name = "productService")
    public CompletableFuture<ProductDto> getProduct(String id) {
        return CompletableFuture.supplyAsync(() -> remoteApiClient.fetchProduct(id));
    }

    public CompletableFuture<ProductDto> getCachedProduct(String id, Throwable t) {
        return CompletableFuture.completedFuture(localCache.getFallbackProduct(id));
    }
}
```
