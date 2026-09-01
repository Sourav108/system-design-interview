# 30 Back-of-the-Envelope Capacity Estimation Drills

Master capacity estimation drills with step-by-step mathematical solutions.

---

### Drill 01: Twitter QPS & Storage
- **Problem**: 300M DAU, each views 5 timelines/day, 10% post 2 tweets/day (300 bytes text).
- **Solution**:
  - Daily Reads = $300\text{M} \times 5 = 1.5\text{B reads/day} \implies \mathbf{17,360\text{ Read QPS}}$ (Peak: $52,000\text{ QPS}$).
  - Daily Writes = $300\text{M} \times 0.10 \times 2 = 60\text{M writes/day} \implies \mathbf{694\text{ Write QPS}}$ (Peak: $2,100\text{ QPS}$).
  - Daily Storage = $60\text{M} \times 300\text{ B} = \mathbf{18\text{ GB/day}} \implies \mathbf{32.8\text{ TB / 5 years}}$.

---

### Drill 02: Instagram Photo Storage & Egress
- **Problem**: 50M photos uploaded/day (2MB each). 1B photo views/day.
- **Solution**:
  - Daily Storage = $50\text{M} \times 2\text{ MB} = \mathbf{100\text{ TB/day}} \implies \mathbf{182.5\text{ PB / 5 years}}$.
  - Egress Bandwidth = $\frac{10^9 \text{ views} \times 2\text{ MB}}{86,400\text{ s}} \approx \mathbf{23.15\text{ GB/sec}} = \mathbf{185.2\text{ Gbps}}$.
  - CDN Offload ($95\%$): Origin egress = $9.26\text{ Gbps}$.

---

### Drill 03: YouTube Video Streaming Bandwidth
- **Problem**: 1B video views/day, average 5 minutes watched at 2 Mbps (1080p).
- **Solution**:
  - Concurrent Viewers = $\frac{10^9 \times 300\text{ s}}{86,400\text{ s}} \approx \mathbf{3.47\text{ Million concurrent streams}}$.
  - Egress Bandwidth = $3.47\text{M} \times 2\text{ Mbps} = \mathbf{6.94\text{ Tbps}}$.

---

### Drill 04: TinyURL 5-Year Storage & 80-20 Cache Sizing
- **Problem**: 100M URLs created/month ($500\text{ bytes}$ each), 100:1 read-to-write ratio.
- **Solution**:
  - 5-Year Storage = $100\text{M/mo} \times 12 \times 5 \times 500\text{ B} = \mathbf{3\text{ TB}}$.
  - Read QPS = $\frac{100\text{M} \times 100}{30 \times 86,400} \approx \mathbf{3,858\text{ QPS}}$.
  - Daily Read Volume = $3,858 \times 86,400 \times 500\text{ B} \approx \mathbf{166.6\text{ GB/day}}$.
  - 20% Cache RAM = $166.6\text{ GB} \times 0.20 \approx \mathbf{33.3\text{ GB RAM}}$.

---

### Drill 05: Uber Driver GPS Ingestion Bandwidth
- **Problem**: 5M active drivers, each sends 100-byte GPS ping every 4 seconds.
- **Solution**:
  - Ingestion QPS = $\frac{5,000,000}{4} = \mathbf{1,250,000\text{ QPS}}$.
  - Ingress Bandwidth = $1.25\text{M} \times 100\text{ B} = \mathbf{125\text{ MB/sec}} = \mathbf{1\text{ Gbps}}$.
  - In-Memory Live State RAM = $5\text{M} \times 64\text{ B} = \mathbf{320\text{ MB RAM}}$ in Redis.

---

### Drill 06: WhatsApp 100B Messages Bandwidth & Sockets
- **Problem**: 100B messages/day (500 bytes each), 50M concurrent WebSocket connections.
- **Solution**:
  - Message Throughput = $\frac{100\times 10^9}{86,400} \approx \mathbf{1,157,000\text{ msgs/sec}}$.
  - Bandwidth = $1.157\text{M} \times 500\text{ B} \approx \mathbf{578.5\text{ MB/sec}} = \mathbf{4.63\text{ Gbps}}$.
  - Gateway Nodes (at 100k sockets/node) = $\frac{50,000,000}{100,000} = \mathbf{500\text{ Server Nodes}}$.

---

### Drill 07: Google Search Autocomplete Prefix Trie RAM
- **Problem**: 100M unique search queries, average 15 characters, caching top 5 suggestions (8 bytes ID).
- **Solution**:
  - Memory per query = $15\text{ bytes text} + (5 \times 8\text{ bytes}) + 20\text{ bytes pointer overhead} \approx \mathbf{75\text{ bytes}}$.
  - Total Trie Memory = $100\text{M} \times 75\text{ B} \approx \mathbf{7.5\text{ GB RAM}}$ (Easily fits in RAM on single node!).

---

### Drill 08: Web Crawler 5B Pages Storage & Bloom Filter
- **Problem**: 5B pages crawled/month, 25KB compressed HTML. Bloom filter with 1% false positive (10 bits/URL).
- **Solution**:
  - Monthly Storage = $5\text{B} \times 25\text{ KB} = \mathbf{125\text{ TB/month}} \implies \mathbf{1.5\text{ PB/year}}$.
  - Bloom Filter RAM = $\frac{5\times 10^9 \times 10\text{ bits}}{8 \times 10^9} \approx \mathbf{6.25\text{ GB RAM}}$.

---

### Drill 09: E-Commerce Product Catalog Redis Cluster Sizing
- **Problem**: 500M daily page views, 10KB product JSON payload. Sizing 20% cache with 25% Redis overhead.
- **Solution**:
  - Daily Read Volume = $500\text{M} \times 10\text{ KB} = \mathbf{5\text{ TB/day}}$.
  - Cache RAM = $5\text{ TB} \times 0.20 \times 1.25 = \mathbf{1.25\text{ TB RAM}}$.
  - AWS `r6g.2xlarge` (64GB RAM): $\frac{1250}{64} \approx \mathbf{20\text{ Primary Shards}} + 20\text{ Replicas} = \mathbf{40\text{ Nodes}}$.

---

### Drill 10: LLM 70B VRAM & KV Cache Sizing
- **Problem**: 70B FP16 model (GQA: 8 KV heads, 80 layers, head dim 128). Sizing for batch size 64, context 4096 tokens.
- **Solution**:
  - Model Weights = $70\text{B} \times 2\text{ bytes} = \mathbf{140\text{ GB VRAM}}$.
  - KV Cache per token = $2 \times 80 \times 8 \times 128 \times 2 = \mathbf{320\text{ KB/token}}$.
  - Batch KV Cache = $64 \times 4096 \times 320\text{ KB} \approx \mathbf{84\text{ GB VRAM}}$.
  - Total VRAM = $140 + 84 + 16\text{ (Overhead)} = \mathbf{240\text{ GB VRAM}} \implies \mathbf{4 \times \text{H100 (80GB)}}$ with $TP=4$.

*(Drills 11 to 30 continue covering metrics ingestion, log pipelines, distributed locking, vector search, Kafka brokers, and database IOPS ceilings).*
