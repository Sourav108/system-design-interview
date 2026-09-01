# Numbers Every System Design Engineer Must Know

Back-of-the-envelope capacity estimation requires instant familiarity with fundamental physical hardware limits and powers of two.

---

## 1. Powers of Two and Data Units

| Power of 2 | Exact Value | Approximation | Storage Unit | Real-World Context |
|---|---|---|---|---|
| $2^{10}$ | $1,024$ | $1\text{ Thousand } (1\text{ K})$ | $1\text{ Kilobyte (KB)}$ | Small JSON API payload ($1-2\text{ KB}$) |
| $2^{20}$ | $1,048,576$ | $1\text{ Million } (1\text{ M})$ | $1\text{ Megabyte (MB)}$ | High-resolution image ($2-5\text{ MB}$) |
| $2^{30}$ | $1,073,741,824$ | $1\text{ Billion } (1\text{ B})$ | $1\text{ Gigabyte (GB)}$ | Standard RAM size / 1080p HD movie ($1.5\text{ GB}$) |
| $2^{40}$ | $1,099,511,627,776$ | $1\text{ Trillion } (1\text{ T})$ | $1\text{ Terabyte (TB)}$ | NVMe SSD capacity / Database table partition |
| $2^{50}$ | $1,125,899,906,842,624$ | $1\text{ Quadrillion } (1\text{ P})$| $1\text{ Petabyte (PB)}$ | Large enterprise data lakehouse / YouTube storage |

---

## 2. Universal Time & Seconds Conversions

Always memorize this constant:
$$\mathbf{1\text{ Day} = 86,400\text{ seconds} \approx 10^5\text{ seconds}}$$
$$\mathbf{1\text{ Month} \approx 2.5 \times 10^6\text{ seconds}}$$
$$\mathbf{1\text{ Year} \approx 3.15 \times 10^7\text{ seconds}}$$

- **Fast Mental Math Trick**:
  - $1\text{ Million requests/day} \approx \frac{1,000,000}{100,000} \approx \mathbf{10\text{ QPS}}$
  - $100\text{ Million requests/day} \approx \mathbf{1,000\text{ QPS}}$
  - $1\text{ Billion requests/day} \approx \mathbf{10,000\text{ QPS}}$

---

## 3. Hardware Throughput Ceilings

| Hardware Subsystem | Realistic Throughput Limit | Latency Order |
|---|---|---|
| **L1 CPU Cache** | Terabytes/sec | $0.5\text{ ns}$ |
| **RAM Sequential Read** | $20 - 50\text{ GB/sec}$ | $100\text{ ns}$ |
| **NVMe SSD Sequential Read** | $3 - 7\text{ GB/sec}$ | $100\text{ }\mu\text{s}$ |
| **NVMe SSD Random IOPS** | $50,000 - 500,000\text{ IOPS}$ | $200\text{ }\mu\text{s}$ |
| **HDD Random IOPS** | $100 - 200\text{ IOPS}$ | $10\text{ ms}$ |
| **1 Gbps NIC Bandwidth** | $125\text{ MB/sec}$ | $0.5\text{ ms (LAN)}$ |
| **10 Gbps NIC Bandwidth** | $1.25\text{ GB/sec}$ | $0.5\text{ ms (LAN)}$ |
| **Redis In-Memory Key Lookup**| $100,000\text{ QPS per single thread}$ | $< 1\text{ ms}$ |
| **PostgreSQL / MySQL Write** | $2,000 - 10,000\text{ TPS per primary node}$| $5 - 20\text{ ms}$ |
| **Kafka Broker Partition Write**| $200,000\text{ messages/sec per broker}$ | $< 5\text{ ms}$ |
