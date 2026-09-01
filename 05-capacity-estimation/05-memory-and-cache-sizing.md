# Memory & Distributed Cache Sizing

Caching is the primary architectural tool to achieve sub-10ms read latencies and protect primary database clusters from read saturation.

---

## 1. The 80-20 Pareto Principle

In real-world distributed web applications, user access patterns follow a power-law distribution:
$$\mathbf{20\%\text{ of hot content generates } 80\%\text{ of total read traffic.}}$$

Therefore, caching the top **20% of daily read data** in fast in-memory RAM (Redis/Memcached) satisfies 80% of all requests directly from memory!

---

## 2. The Cache Sizing Formula

$$\text{Daily Read Volume} = \text{Daily Reads} \times \text{Average Record Size}$$

$$\text{Cache RAM Capacity} = \text{Daily Read Volume} \times 0.20 \times (1 + \text{Redis Memory Overhead } 0.25)$$

---

## 3. Worked Interview Example: E-Commerce Product Catalog

- **Assumptions**:
  - $500\text{M}$ product page views per day.
  - Average product details JSON payload = $10\text{ KB}$.
- **Calculations**:
  - **Daily Read Volume**: $500\text{M} \times 10\text{ KB} = \mathbf{5\text{ TB/day}}$.
  - **20% Cache Sizing**: $5\text{ TB} \times 0.20 = \mathbf{1\text{ TB RAM}}$.
  - **Redis Overhead & Headroom ($25\%$)**: $1\text{ TB} \times 1.25 = \mathbf{1.25\text{ TB RAM}}$.
  - **Redis Cluster Topology**:
    - Standard AWS `r6g.2xlarge` instance provides $64\text{ GB RAM}$.
    - Total Nodes Needed: $\frac{1250\text{ GB}}{64\text{ GB}} \approx \mathbf{20\text{ Primary Shards}} + 20\text{ Standby Replicas} = 40\text{ Nodes}$.
