# Production Outage Post-Mortem 12: Global CDN Cache Purge & Origin Server Crash

> **Severity**: SEV-1 Outage
> **Impacted Domain**: Distributed Reliability & Fault Tolerance
> **Post-Mortem Framework**: Cause $\to$ Symptoms $\to$ Detection $\to$ Mitigation $\to$ Recovery $\to$ Prevention

---

## 1. Incident Summary
An engineer issued a global CDN cache purge (`/*`) to deploy a minor CSS hotfix. 500,000 global users immediately bypassed CDN edge caches and hit the origin media servers simultaneously, causing 100% origin network bandwidth saturation and crashing origin storage nodes.

---

## 2. Root Cause Analysis
Executing a full wildcard cache purge on production CDNs during peak traffic hours without Origin Shielding.

---

## 3. Symptoms & Blast Radius
- **Origin Down**: Origin server network bandwidth saturated at 10 Gbps (100% capacity).
- **Global Outage**: Media and video playback failed for 100% of users.

---

## 4. Detection & Time to Detect (TTD)
- **Time to Detect (TTD)**: $1\text{ minute}$ (Origin bandwidth saturation alarm).

---

## 5. Immediate Mitigation (Stop the Bleeding)
1. Enabled CDN emergency rate limiting and pre-warmed hot assets via CDN push API.

---

## 6. Full Recovery & Time to Recovery (TTR)
- **Time to Recovery (TTR)**: $25\text{ minutes}$.

---

## 7. Architectural Prevention
```mermaid
flowchart TD
    Edge[1,000 Edge CDN POPs: Cache Miss] --> Shield[CDN Origin Shield POPs (Collapses 1,000 reqs into 1)]
    Shield --> S3Origin[(Origin Storage: Handles only 1 request! 🛡️)]
```
- Always enable **CDN Origin Shielding** (collapses multi-region cache misses into a single request).
- Use **Versioned Asset URLs** (`bundle.v2.js`) rather than global cache purges.
- Enable `stale-while-revalidate` HTTP headers so edges serve stale content while refreshing.

---

## 8. Code & Configuration Fix
```java
// Cache-Control Header with Stale-While-Revalidate
response.setHeader("Cache-Control", "public, max-age=3600, stale-while-revalidate=86400");
```

---

## 9. Key Lessons Learned
1. Never execute a wildcard `/*` cache purge in production during peak traffic.
2. Origin Shielding is mandatory to protect backend storage from stampedes.
3. Versioned URLs eliminate the need for manual cache purges.
