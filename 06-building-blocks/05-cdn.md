# Building Block 05: CDN (Content Delivery Network & Edge Caching)

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry

---

## 1. Problem
Users located globally experience massive latency ($150\text{ms}+$) and origin bandwidth saturation when downloading static media (images, videos, JS/CSS bundles) directly from a centralized origin datacenter.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
Physics limits packet speed over fiber. CDNs distribute hundreds of Points of Presence (POPs) at the edge of the Internet close to end users, offloading origin servers and eliminating geographic latency.

## 4. Mental Model
A global chain of local convenience stores stocking popular items locally so customers don't have to travel to the central factory.

## 5. Core Concepts
Edge PoP (Point of Presence), Anycast DNS routing, Push vs Pull CDN, Cache-Control headers (`max-age`, `s-maxage`), Origin Shielding, Dynamic Content Acceleration (DCA), Byte-Range Requests.

## 6. Architecture
```mermaid
flowchart LR
    Client[Global User] --> EdgePoP[Nearest CDN Edge PoP]
    EdgePoP -->|Cache Hit: 95%| ReturnMedia[Return Cached Asset (p99 < 15ms) ✅]
    EdgePoP -->|Cache Miss: 5%| OriginShield[CDN Origin Shield]
    OriginShield --> OriginServer[(Central Origin S3 / Server)]
```

## 7. Request/Data Flow
1. Client requests `cdn.example.com/video.mp4`. 2. DNS Anycast routes to closest Edge PoP. 3. Edge checks local SSD cache. 4. On hit: returns immediately. 5. On miss: fetches from Origin Shield / S3, stores locally, returns to user.

## 8. Data Model
Cached objects: Static files (Images, HLS video chunks, JS/CSS), HTTP response headers, ETag validation tokens.

## 9. API Design
Standard HTTP/2 and HTTP/3 GET requests with caching headers (`If-None-Match`, `Range: bytes=0-1048575`).

## 10. Algorithms
LRU / LFU cache eviction at edge nodes, Consistent Hashing for edge server proxy routing.

## 11. Scaling
Scale globally by peering with Internet Exchange Points (IXPs) and ISPs across 300+ edge locations.

## 12. Partitioning
Partitioned by URL hash across cache servers within an edge PoP.

## 13. Replication
Edge caches populated on-demand (Pull model) or pre-warmed via API (Push model).

## 14. Consistency
Eventual consistency; updates propagate via TTL expiration or explicit Cache Purge / Invalidation API calls.

## 15. Failure Scenarios
Origin downtime (CDN serves stale cached content gracefully), Edge PoP network congestion, Cache avalanche on cache purge.

## 16. Recovery
Origin Shielding collapses simultaneous cache misses into a single request to origin; automatic failover to neighboring Edge PoP.

## 17. Observability
Cache Hit Ratio (Target > 95%), Origin offload percentage, Edge TTFB (Time to First Byte < 20ms), Egress bandwidth savings.

## 18. Security
TLS termination at edge, Signed URLs / Signed Cookies for private content authorization, DDoS Layer 7 mitigation.

## 19. Performance
HTTP/3 QUIC 0-RTT connection establishment, Byte-Range chunk streaming for video seeking, Gzip / Brotli compression.

## 20. Trade-offs
Cache Freshness (Low TTL = fresh data, high origin load) vs Offload (High TTL = stale risk, massive origin savings).

## 21. When to Use
Static assets, video streaming chunks (HLS/DASH), software download binaries, high-traffic read-only API responses.

## 22. When NOT to Use
Dynamic, user-specific personalized transactional data (e.g. user bank account balances, checkout cart state).

## 23. Implementation Strategy
Integrate Cloudflare / AWS CloudFront with Origin S3 buckets, configure signed CloudFront URLs for private media.

## 24. Practical Exercise
Configure an S3 bucket with CloudFront CDN, set `Cache-Control: public, max-age=31536000, immutable`, and benchmark TTFB.

## 25. Interview Questions
1. What is the difference between Push and Pull CDNs? 2. How do Signed URLs protect premium streaming content? 3. What is Origin Shielding?

## 26. Common Mistakes
Failing to use cache-busting versioned file hashes (e.g. `bundle.v2.js`), forcing slow global CDN purge operations.

## 27. Quick Revision
CDN Edge PoPs cache static media close to users -> 95%+ origin offload -> Sub-20ms global delivery.

## 28. Related Building Blocks
`BB-01` (DNS), `BB-02` (Load Balancer), `BB-14` (Blob Store)

## 29. Related Case Studies
`CS-01` (YouTube / Netflix), `CS-08` (Instagram), `CS-12` (Search Autocomplete)
