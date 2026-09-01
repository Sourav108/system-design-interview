# Load Balancing & Traffic Routing

| Layer / Mechanism | Protocol Level | Routing Criteria | Performance | Use Case |
|---|---|---|---|---|
| **Anycast BGP** | IP / BGP Network | BGP routing distance to nearest Edge POP | Terabits/sec (Hardware) | Global DNS & CDN edge steering |
| **Layer 4 (L4)** | Transport (TCP/UDP) | IP 5-tuple (Src/Dst IP, Src/Dst Port, Protocol) | Millions QPS (Kernel-level) | Ingress load distribution across L7 proxies |
| **Layer 7 (L7)** | Application (HTTP/gRPC) | URL Path, Headers, Cookies, JWT, Body payload | 50k - 100k QPS (User-space) | API Gateways, path routing, auth filters |

---

## Load Balancing Algorithms
- **Round Robin**: Distributes requests sequentially. Best for homogeneous servers and equal request times.
- **Weighted Round Robin**: Routes traffic proportionally based on assigned server compute capacity (vCPU/RAM).
- **Least Connections**: Routes to backend node with lowest active TCP connections. Optimal for variable-duration requests.
- **Power of Two Random Choices (P2C)**: Randomly picks 2 healthy nodes and chooses the least loaded one. Avoids thundering herd.
- **Consistent Hashing**: Hashes request key to consistent ring. Guarantees cache affinity and zero connection disruption on scale-out.
