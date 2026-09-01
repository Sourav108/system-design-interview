# Bandwidth & Network Throughput Estimation

Bandwidth estimation calculates the inbound (ingress) and outbound (egress) network load on your load balancers, gateways, and CDN edge infrastructure.

---

## 1. The Bandwidth Formulas

$$\text{Ingress Bandwidth} = \text{Write QPS} \times \text{Average Upload Payload Size}$$

$$\text{Egress Bandwidth} = \text{Read QPS} \times \text{Average Download Payload Size}$$

---

## 2. Converting Bits vs Bytes

- **Storage** is measured in **Bytes** ($1\text{ Byte} = 8\text{ bits}$, uppercase **B**).
- **Network Bandwidth** is measured in **bits per second** ($1\text{ Gbps} = 125\text{ MB/s}$, lowercase **b**).

---

## 3. Worked Interview Example: Video Streaming (YouTube / Netflix)

- **Assumptions**:
  - $1\text{ Billion}$ video views per day.
  - Average video duration watched = $5\text{ minutes}$.
  - Average video stream bitrate (1080p compressed) = $2\text{ Mbps} = 250\text{ KB/sec}$.
- **Calculations**:
  - **Average Concurrent Streams**:
    $$\text{Daily Watch Seconds} = 10^9 \times 300\text{ sec} = 3 \times 10^{11}\text{ viewer-seconds/day}$$
    $$\text{Concurrent Viewers} = \frac{3 \times 10^{11}}{86,400} \approx \mathbf{3.47\text{ Million concurrent streams}}$$
  - **Total Egress Bandwidth**:
    $$3.47\text{M streams} \times 2\text{ Mbps} = \mathbf{6.94\text{ Tbps (Terabits per second)}}$$
  - **Architectural Defense**:
    - A single datacenter cannot handle 7 Tbps egress directly from origin servers.
    - **Mitigation**: Deploy **95%+ caching at Global CDN Edge POPs** (Akamai/Cloudflare). Origin servers only handle $5\%$ ($~350\text{ Gbps}$).
