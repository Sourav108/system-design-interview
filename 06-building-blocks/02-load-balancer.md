# Building Block 02: Load Balancer (L4 vs L7 & Reverse Proxy)

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry

---

## 1. Problem
A single backend server instance has physical compute and network limits. As traffic grows, requests must be evenly distributed across a dynamic pool of healthy backend instances.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
Without a load balancer, clients must track server IPs directly. A server crash causes client outages, and traffic cannot be balanced across heterogeneous compute nodes.

## 4. Mental Model
A traffic cop at a busy intersection directing cars (packets/requests) to the least congested open lane (healthy backend server).

## 5. Core Concepts
Layer 4 (Transport/TCP/UDP) vs Layer 7 (Application/HTTP) load balancing, Round Robin, Weighted Round Robin, Least Connections, IP Hash, Consistent Hashing, Passive vs Active Health Checks, TLS Termination.

## 6. Architecture
```mermaid
flowchart TD
    Client[Clients] --> LB[Layer 7 Load Balancer: Envoy / NGINX]
    LB -->|Active Health Check: /health OK| Svc1[App Instance 1 (Least Conns)]
    LB -->|Active Health Check: /health OK| Svc2[App Instance 2]
    LB -.->|Health Check Failed: 503| Svc3[App Instance 3 (Drained)]
```

## 7. Request/Data Flow
1. Client establishes TCP/TLS with LB. 2. LB inspects HTTP headers/path (L7) or TCP 5-tuple (L4). 3. Applies balancing algorithm. 4. Proxies request to selected healthy backend via connection pool. 5. Streams response back.

## 8. Data Model
In-memory routing tables: upstream cluster definitions, backend IP/port lists, active connection counts, health check states.

## 9. API Design
L7 HTTP Reverse Proxy; gRPC multiplexed HTTP/2 balancing; TCP stream proxying.

## 10. Algorithms
Weighted Round Robin, Maglev Consistent Hashing, Power of Two Random Choices (P2C with Least Loaded).

## 11. Scaling
Scale L4 via ECMP (Equal-Cost Multi-Path) hardware switches; scale L7 horizontally behind L4 Maglev load balancers.

## 12. Partitioning
Traffic partitioned by path (`/api/v1/orders` -> Cluster A) or user session hash.

## 13. Replication
Active-Standby LB pairs with VRRP / Keepalived; Active-Active multi-node clusters with Anycast BGP.

## 14. Consistency
Stateless proxying; upstream routing tables eventually consistent via service discovery sync.

## 15. Failure Scenarios
Backend node crash (auto-drained in < 5s), LB instance failure (VRRP takeover), Connection pool exhaustion.

## 16. Recovery
Automated instance draining, fast health check failure tripping (3 failed probes), dynamic DNS failover.

## 17. Observability
Active connections, connection rate, upstream response latency percentiles (p50/p95/p99), HTTP 5xx error rates.

## 18. Security
TLS 1.3 Termination, WAF (Web Application Firewall) inspection, DDoS SYN flood protection, mTLS to backends.

## 19. Performance
TCP Connection pooling, zero-copy I/O (`splice`/`epoll`), HTTP/2 multiplexing, TLS session resumption.

## 20. Trade-offs
L4 (Ultra-high throughput, no header inspection) vs L7 (Smart path routing, higher CPU overhead).

## 21. When to Use
Distributing traffic across backend clusters, SSL offloading, blue/green and canary deployments.

## 22. When NOT to Use
Static single-server applications, purely local inter-thread processing.

## 23. Implementation Strategy
Deploy Envoy Proxy or NGINX with least-connection balancing, active health checks, and Keep-Alive connection pools.

## 24. Practical Exercise
Configure NGINX with weighted round-robin across 3 backend Spring Boot instances and test live container kill recovery.

## 25. Interview Questions
1. Explain L4 vs L7 load balancing. 2. What is the Power of Two Random Choices algorithm? 3. How does connection draining work during deployments?

## 26. Common Mistakes
Using Round Robin for requests with wildly varying processing times (causes severe server overload; use Least Connections instead).

## 27. Quick Revision
L4 = fast TCP packet routing; L7 = smart HTTP application routing; Health checks isolate dead nodes instantly.

## 28. Related Building Blocks
`BB-01` (DNS), `BB-19` (API Gateway), `BB-20` (Service Discovery)

## 29. Related Case Studies
`CS-06` (Twitter / X), `CS-09` (TinyURL), `CS-15` (Payment Gateway)
