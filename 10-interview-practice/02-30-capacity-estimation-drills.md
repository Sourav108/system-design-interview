# 30 Back-of-the-Envelope Capacity Estimation Drills

A comprehensive collection of **30 complete back-of-the-envelope capacity estimation drills** with real-world problem statements, core assumptions, step-by-step mathematical calculations, and hardware/infrastructure sizing conclusions.

---

### Drill 01: Twitter / X — QPS, Storage & Timeline Cache
- **Problem**: 300M Daily Active Users (DAU). Each user views their timeline 5 times/day. 10% of users post 2 tweets/day (average 300 bytes text).
- **Assumptions**: 1 Day = $86,400\text{ seconds} \approx 10^5\text{ seconds}$. Peak multiplier = $3\times$.
- **Calculations**:
  - **Daily Reads**: $300\text{M} \times 5 = 1.5\text{ Billion timeline reads/day}$.
  - **Average Read QPS**: $\frac{1.5 \times 10^9}{86,400} \approx \mathbf{17,360\text{ QPS}}$ (Peak: $\mathbf{52,000\text{ QPS}}$).
  - **Daily Writes**: $300\text{M} \times 0.10 \times 2 = 60\text{ Million tweets/day}$.
  - **Average Write QPS**: $\frac{60 \times 10^6}{86,400} \approx \mathbf{694\text{ QPS}}$ (Peak: $\mathbf{2,100\text{ QPS}}$).
  - **Daily Storage**: $60\text{M} \times 300\text{ bytes} = \mathbf{18\text{ GB/day}}$.
  - **5-Year Storage**: $18\text{ GB} \times 365 \times 5 \approx \mathbf{32.8\text{ TB}}$ (with index + 3x replication $\approx \mathbf{118\text{ TB}}$).
  - **Timeline Cache Sizing**: Cache top 800 tweet IDs (8 bytes each) for 300M users: $300\text{M} \times 800 \times 8\text{ bytes} \approx \mathbf{1.92\text{ TB RAM}}$ in Redis.

---

### Drill 02: Instagram — Photo Storage & Global Egress Bandwidth
- **Problem**: 50M new photos uploaded per day (2MB high-res original + 500KB thumbnails). 1 Billion photo views per day.
- **Assumptions**: 95% of photo views are offloaded by Edge CDNs.
- **Calculations**:
  - **Daily Ingestion**: $50\text{M} \times 2.5\text{ MB} = \mathbf{125\text{ TB/day}}$.
  - **5-Year Blob Storage**: $125\text{ TB/day} \times 365 \times 5 \approx \mathbf{228\text{ Petabytes}}$ in Amazon S3.
  - **Total Egress Bandwidth**: $\frac{10^9 \text{ views} \times 500\text{ KB}}{86,400\text{ s}} \approx \mathbf{5.78\text{ GB/sec}} = \mathbf{46.2\text{ Gbps}}$.
  - **Origin Egress (5% Cache Miss)**: $46.2\text{ Gbps} \times 0.05 = \mathbf{2.31\text{ Gbps}}$ on origin storage.

---

### Drill 03: YouTube / Netflix — Video Streaming Concurrent Egress
- **Problem**: 1 Billion video views/day. Average video watch time = 5 minutes ($300\text{ seconds}$). Stream bitrate = 2 Mbps (1080p compressed).
- **Calculations**:
  - **Total Daily Watch Seconds**: $10^9 \times 300\text{ s} = 3 \times 10^{11}\text{ viewer-seconds/day}$.
  - **Concurrent Streaming Viewers**: $\frac{3 \times 10^{11}}{86,400} \approx \mathbf{3,472,222\text{ concurrent streams}}$.
  - **Total Egress Bandwidth**: $3.47\text{M streams} \times 2\text{ Mbps} = \mathbf{6.94\text{ Terabits per second (Tbps)}}$.
  - **Hardware Infrastructure**: Requires global edge caching across 300+ Edge POPs; single origin cannot sustain 7 Tbps egress.

---

### Drill 04: TinyURL / Bitly — 5-Year Storage & 80-20 Cache RAM
- **Problem**: 100M new URLs shortened per month ($500\text{ bytes}$ metadata). 100:1 read-to-write ratio. Default 5-year retention.
- **Calculations**:
  - **Total 5-Year URLs**: $100\text{M/month} \times 12 \times 5 = \mathbf{6\text{ Billion URLs}}$.
  - **Total 5-Year Disk Storage**: $6\text{B} \times 500\text{ bytes} \approx \mathbf{3\text{ TB}}$ (Fits comfortably on partitioned DynamoDB/PostgreSQL).
  - **Read QPS**: $\frac{100\text{M} \times 100}{30 \times 86,400} \approx \mathbf{3,858\text{ QPS}}$ (Peak: $\mathbf{12,000\text{ QPS}}$).
  - **Daily Read Volume**: $3,858 \times 86,400 \times 500\text{ bytes} \approx \mathbf{166.6\text{ GB/day}}$.
  - **80-20 Pareto Cache Sizing**: $166.6\text{ GB} \times 0.20 \times 1.25\text{ (Redis overhead)} \approx \mathbf{41.6\text{ GB RAM}}$ in Redis.

---

### Drill 05: Uber / Lyft — 5M Drivers Real-Time GPS Ingestion
- **Problem**: 5M active drivers streaming GPS coordinates (100-byte JSON payload) every 4 seconds.
- **Calculations**:
  - **Ingestion Write QPS**: $\frac{5,000,000\text{ drivers}}{4\text{ seconds}} = \mathbf{1,250,000\text{ GPS pings/sec}}$.
  - **Network Ingress Bandwidth**: $1.25\text{M pings/sec} \times 100\text{ bytes} = \mathbf{125\text{ MB/sec}} = \mathbf{1\text{ Gbps}}$.
  - **Live Driver Spatial Grid RAM**: $5\text{M drivers} \times 64\text{ bytes (ID + Lat + Lon + S2 Cell)} \approx \mathbf{320\text{ MB RAM}}$ in Redis!

---

### Drill 06: WhatsApp / Discord — 100B Messages & 50M Concurrent Sockets
- **Problem**: 100 Billion messages sent per day ($500\text{ bytes}$ per message). 50 Million peak concurrent open WebSocket connections.
- **Calculations**:
  - **Message Ingestion QPS**: $\frac{100 \times 10^9}{86,400} \approx \mathbf{1,157,400\text{ msgs/sec}}$ (Peak: $\mathbf{3.5\text{M msgs/sec}}$).
  - **Network Bandwidth**: $1.157\text{M} \times 500\text{ bytes} \approx \mathbf{578.7\text{ MB/sec}} = \mathbf{4.63\text{ Gbps}}$.
  - **WebSocket Gateway Sizing**: Single Netty server node handles $100,000\text{ active TCP sockets}$.
    $$\text{Gateway Nodes Needed} = \frac{50,000,000}{100,000} = \mathbf{500\text{ Gateway Instances}}.$$

---

### Drill 07: Google Search Autocomplete — Prefix Trie In-Memory Sizing
- **Problem**: 100 Million unique search queries. Average length = 15 characters. Storing top 5 suggestion IDs (8 bytes each) at each Trie node.
- **Calculations**:
  - **Node Storage**: $15\text{ bytes string} + (5 \times 8\text{ bytes cached IDs}) + 24\text{ bytes pointer overhead} \approx \mathbf{79\text{ bytes/node}}$.
  - **Total Trie RAM**: $100\text{M} \times 79\text{ bytes} \approx \mathbf{7.9\text{ GB RAM}}$.
  - **Deployment**: Entire global Trie easily fits into memory on a single 32GB server; replicated across 50 nodes to handle 300,000 QPS.

---

### Drill 08: Distributed Web Crawler — 5B Pages Storage & Bloom Filter
- **Problem**: Crawling 5 Billion web pages per month. Average compressed HTML = 25KB. Sizing a Bloom Filter with 1% false positive probability ($p=0.01$).
- **Calculations**:
  - **Continuous Crawl Rate**: $\frac{5 \times 10^9}{30 \times 86,400} \approx \mathbf{1,929\text{ pages/sec}}$ (Peak: $\mathbf{5,000\text{ pages/sec}}$).
  - **Monthly Raw Storage**: $5\text{B} \times 25\text{ KB} = \mathbf{125\text{ TB/month}} \implies \mathbf{1.5\text{ PB/year}}$ in S3.
  - **Bloom Filter Bits Formula**: $m = -\frac{n \ln p}{(\ln 2)^2} \approx n \times 9.6\text{ bits} \approx 10\text{ bits/URL}$.
  - **Bloom Filter Memory**: $\frac{5 \times 10^9 \times 10\text{ bits}}{8 \times 10^9\text{ bits/GB}} \approx \mathbf{6.25\text{ GB RAM}}$.

---

### Drill 09: E-Commerce Product Catalog — 500M Daily Views Redis Sizing
- **Problem**: 500M product page views per day. Average product details payload = 10KB JSON. Sizing 20% cache with 25% Redis internal pointer overhead.
- **Calculations**:
  - **Daily Read Volume**: $500\text{M} \times 10\text{ KB} = \mathbf{5\text{ TB/day}}$.
  - **20% Cache Working Set**: $5\text{ TB} \times 0.20 = \mathbf{1\text{ TB}}$.
  - **With 25% Overhead**: $1\text{ TB} \times 1.25 = \mathbf{1.25\text{ TB RAM}}$.
  - **Cluster Topology (AWS `r6g.2xlarge` with 64GB RAM)**: $\frac{1250\text{ GB}}{64\text{ GB}} \approx \mathbf{20\text{ Primary Shards}} + 20\text{ Standby Replicas} = \mathbf{40\text{ Nodes}}$.

---

### Drill 10: LLM 70B Serving — Model Weights, KV Cache & GPU Sizing
- **Problem**: Serving Llama-3-70B FP16 model (GQA: 8 KV heads, 80 layers, head dim 128). Sizing for batch size $B=64$, context length $L=4,096$ tokens.
- **Calculations**:
  - **Model Weights VRAM**: $70\text{B} \times 2\text{ bytes (FP16)} = \mathbf{140\text{ GB VRAM}}$.
  - **KV Cache Memory Per Token**: $2 \times 80 \times 8 \times 128 \times 2\text{ bytes} = \mathbf{327,680\text{ bytes}} \approx \mathbf{320\text{ KB/token}}$.
  - **Batch KV Cache VRAM**: $64 \times 4096 \times 320\text{ KB} \approx \mathbf{83.88\text{ GB VRAM}}$.
  - **Total Node VRAM Required**: $140\text{ GB} + 84\text{ GB} + 16\text{ GB (CUDA Overhead)} = \mathbf{240\text{ GB VRAM}}$.
  - **Hardware Selection**: **$4 \times \text{NVIDIA H100 (80GB)}$** with Tensor Parallelism $TP=4$ ($320\text{ GB total VRAM}$).

---

### Drill 11: Prometheus Metrics TSDB — 10M Active Series Ingestion
- **Problem**: 10 Million active metric time-series scraped every 15 seconds. Using Gorilla 1.37-byte timestamp/value compression.
- **Calculations**:
  - **Sample Ingestion Rate**: $\frac{10,000,000}{15\text{ s}} \approx \mathbf{666,666\text{ samples/sec}}$.
  - **Daily Storage Ingestion**: $666,666\text{ samples/s} \times 1.37\text{ bytes} \times 86,400\text{ s} \approx \mathbf{78.9\text{ GB/day}}$.
  - **30-Day Retention Disk**: $78.9\text{ GB} \times 30 \approx \mathbf{2.36\text{ TB SSD}}$.

---

### Drill 12: Distributed Logging Pipeline (ELK) — 500M Daily JSON Logs
- **Problem**: Microservice fleet generates 500M structured JSON logs per day. Average uncompressed log = 1KB; compressed in Elasticsearch = 300 bytes.
- **Calculations**:
  - **Ingestion Write Throughput**: $\frac{500 \times 10^6}{86,400} \approx \mathbf{5,787\text{ logs/sec}}$ (Peak: $\mathbf{20,000\text{ logs/sec}}$).
  - **Daily Index Storage**: $500\text{M} \times 300\text{ bytes} \approx \mathbf{150\text{ GB/day}}$.
  - **Accounting for 1 Primary + 1 Replica Shard**: $150\text{ GB} \times 2 = \mathbf{300\text{ GB/day}} \implies \mathbf{9\text{ TB / 30 days}}$.

---

### Drill 13: Stateless API Gateway Server Sizing
- **Problem**: Peak inbound traffic = 100,000 HTTPS QPS. Single 8-vCPU container gateway instance handles 5,000 QPS (TLS termination + JWT validation).
- **Calculations**:
  - **Base Compute Nodes**: $\frac{100,000}{5,000} = 20\text{ Instances}$.
  - **With 30% Autoscaling Spike Headroom**: $20 \times 1.30 = \mathbf{26\text{ Instances}}$ distributed across 3 Availability Zones (9 instances per AZ).

---

### Drill 14: Vector Database (Milvus/Pinecone) — 100M Embeddings RAM Sizing
- **Problem**: 100M document embeddings (1536-dimensional float32 vectors). Building an HNSW vector graph ($M=16, efConstruction=200$).
- **Calculations**:
  - **Raw Vector Memory**: $100\text{M} \times 1536 \times 4\text{ bytes} = \mathbf{614.4\text{ GB RAM}}$.
  - **HNSW Graph Overhead ($20\%$)**: $614.4\text{ GB} \times 1.20 \approx \mathbf{737.3\text{ GB RAM}}$.
  - **With Scalar Quantization (INT8 - $4\times$ reduction)**: $\frac{737.3\text{ GB}}{4} \approx \mathbf{184.3\text{ GB RAM}}$.

---

### Drill 15: Background Job Queue Fleet Sizing via Little's Law
- **Problem**: Ingestion rate $\lambda = 500\text{ jobs/sec}$. Average job execution duration $W = 2.5\text{ seconds}$. Each worker container has 4 worker threads.
- **Calculations**:
  - **Little's Law Formula**: $L = \lambda \times W$.
  - **Concurrent In-Flight Jobs ($L$)**: $500\text{ jobs/sec} \times 2.5\text{ seconds} = \mathbf{1,250\text{ concurrent jobs}}$.
  - **Worker Containers Needed**: $\frac{1,250\text{ concurrent jobs}}{4\text{ threads/container}} = \mathbf{313\text{ Worker Containers}}$.

---

### Drill 16: Ad Click Telemetry — Flink Sliding Window Ingestion
- **Problem**: 1 Billion ad click events per day (200 bytes each). Processing 10-minute sliding windows (slide 10s) in Apache Flink.
- **Calculations**:
  - **Event Throughput**: $\frac{10^9}{86,400} \approx \mathbf{11,574\text{ events/sec}}$ (Peak: $\mathbf{50,000\text{ events/sec}}$).
  - **Ingress Bandwidth**: $11,574 \times 200\text{ bytes} \approx \mathbf{2.31\text{ MB/sec}} = \mathbf{18.5\text{ Mbps}}$.
  - **Flink RocksDB Window State Memory**: $50,000\text{ peak events/s} \times 600\text{ s window} \times 64\text{ bytes aggregated} \approx \mathbf{1.92\text{ GB RAM}}$.

---

### Drill 17: File Sync (Dropbox) — FastCDC Chunking & Delta Sync
- **Problem**: 100MB video file edited by modifying 1 paragraph. FastCDC average chunk size = 256KB. SHA-256 hash per chunk = 32 bytes.
- **Calculations**:
  - **Total File Chunks**: $\frac{100\text{ MB}}{256\text{ KB}} = \mathbf{400\text{ Chunks}}$.
  - **Merkle Manifest Size**: $400 \times 32\text{ bytes} = \mathbf{12.8\text{ KB}}$.
  - **Delta Transfer Bandwidth**: Only 1 modified chunk is uploaded: $\mathbf{256\text{ KB}}$ (Saves $99.75\%$ of 100MB bandwidth!).

---

### Drill 18: Collaborative Editor (Google Docs) — Operation Ingestion
- **Problem**: 10 Million concurrent active document editors. Average typing speed = 2 edits/sec per user (100 bytes per OT operation payload).
- **Calculations**:
  - **Global Ingestion QPS**: $10\text{M} \times 2\text{ edits/s} = \mathbf{20,000,000\text{ operations/sec}}$.
  - **Global WebSocket Ingress Bandwidth**: $20\text{M} \times 100\text{ bytes} = \mathbf{2\text{ GB/sec}} = \mathbf{16\text{ Gbps}}$.
  - **Coordination Servers (at 200k ops/sec per node)**: $\frac{20\text{M}}{200\text{k}} = \mathbf{100\text{ Coordinator Nodes}}$.

---

### Drill 19: Flash Sale Sharded Counter — 100k Concurrent Writes/sec
- **Problem**: Flash sale SKU receiving 100,000 concurrent likes/inventory increments per second. Single Redis core max write = 10,000 QPS.
- **Calculations**:
  - **Counter Slots Needed ($N$)**: $\frac{100,000\text{ QPS}}{10,000\text{ QPS/slot}} \times 1.5\text{ (Safety Headroom)} = \mathbf{15\text{ Counter Slots}}$.
  - **Read Aggregation Cost**: Summing 15 slots in Redis requires $15 \times \mathcal{O}(1)$ commands via `MGET` ($< 0.5\text{ms}$).

---

### Drill 20: Push Notification Fan-Out — 50M Messages in 15 Minutes
- **Problem**: Breaking news broadcast dispatched to 50M users within a 15-minute target window ($900\text{ seconds}$).
- **Calculations**:
  - **Dispatch Rate**: $\frac{50,000,000}{900\text{ seconds}} \approx \mathbf{55,555\text{ pushes/sec}}$.
  - **APNs / FCM HTTP/2 Multiplexed Sockets**: Single persistent connection handles 500 pushes/sec $\implies \frac{55,555}{500} = \mathbf{112\text{ HTTP/2 Sockets}}$.

---

### Drill 21: Parallel Video Transcoding Compute Sizing
- **Problem**: 10,000 hours of video uploaded daily. Transcoding at 4 resolutions (1080p, 720p, 480p, 360p) takes $0.5\times$ real-time on 8-vCPU instance.
- **Calculations**:
  - **Total Compute Hours Needed**: $10,000\text{ hours} \times 4\text{ streams} \times 0.5 = \mathbf{20,000\text{ CPU-hours/day}}$.
  - **Dedicated Transcoder Worker Instances (24h continuous)**: $\frac{20,000}{24} \approx \mathbf{834\text{ 8-vCPU Instances}}$ (Run on AWS EC2 Spot instances).

---

### Drill 22: Cassandra Wide-Column IoT Telemetry Sizing
- **Problem**: 1 Billion IoT sensor records/day ($200\text{ bytes}$ each). 3x replication factor. Compaction overhead = 50%.
- **Calculations**:
  - **Daily Raw Storage**: $10^9 \times 200\text{ bytes} = \mathbf{200\text{ GB/day}}$.
  - **With 3x Replication**: $200\text{ GB} \times 3 = \mathbf{600\text{ GB/day}}$.
  - **With 50% Compaction Headroom**: $600\text{ GB} \times 1.5 = \mathbf{900\text{ GB/day}} \implies \mathbf{328.5\text{ TB/year}}$.

---

### Drill 23: Elasticsearch Shard & JVM Heap Sizing
- **Problem**: 10TB of raw log data per day. Target primary shard size = 40GB. 1 Replica per primary shard.
- **Calculations**:
  - **Primary Shards Needed**: $\frac{10,000\text{ GB}}{40\text{ GB/shard}} = \mathbf{250\text{ Primary Shards}}$.
  - **Total Shards (with 1 Replica)**: $250 \times 2 = \mathbf{500\text{ Total Shards/day}}$.
  - **Elasticsearch Node Sizing (Rule: Max 20 shards per 1GB JVM Heap)**: Single node with 32GB Heap handles $\approx 600\text{ shards} \implies \mathbf{20\text{ Data Nodes}}$.

---

### Drill 24: Distributed Rate Limiter Redis Memory Overhead
- **Problem**: Protecting 10M active API client keys with Sliding Window Counter (1-minute window). Storing request count and previous window count.
- **Calculations**:
  - **Storage Per Client Key**: Redis Hash (`current_count` + `prev_count` + key metadata) $\approx \mathbf{120\text{ bytes}}$.
  - **Total Redis Cluster RAM**: $10\text{M} \times 120\text{ bytes} \approx \mathbf{1.2\text{ GB RAM}}$ (Easily fits in single small Redis shard).

---

### Drill 25: Cloud Data Lakehouse (Iceberg) — 50TB Daily Ingestion
- **Problem**: 50TB of raw streaming JSON analytics data per day. Compressing to Columnar Parquet with Snappy ($4\times$ compression).
- **Calculations**:
  - **Parquet Storage Ingestion**: $\frac{50\text{ TB}}{4} = \mathbf{12.5\text{ TB/day}}$.
  - **Annual Lakehouse Storage**: $12.5\text{ TB} \times 365 \approx \mathbf{4.56\text{ Petabytes/year}}$ on S3 ($0.023/GB/mo \approx \$105,000/\text{year}$).

---

### Drill 26: CDN Edge Point of Presence (POP) Storage Sizing
- **Problem**: Regional CDN POP serves 100M active requests/day. 90% of requests are for top 100,000 hot media objects (average 2MB each).
- **Calculations**:
  - **Hot Object Working Set**: $100,000 \times 2\text{ MB} = \mathbf{200\text{ GB SSD Storage}}$.
  - **POP Server Sizing**: $4 \times 1\text{TB NVMe SSD}$ edge servers per POP provides full cache redundancy and sub-10ms local reads.

---

### Drill 27: Live Video Transmuxing Buffer Sizing
- **Problem**: 10,000 concurrent live creators streaming RTMP (4 Mbps). Server transmuxes RTMP into 6-second HLS `.ts` video chunks.
- **Calculations**:
  - **Chunk Size per Stream**: $4\text{ Mbps} \times 6\text{ s} = 24\text{ Megabits} = \mathbf{3\text{ Megabytes per chunk}}$.
  - **Total Active Memory Buffer (Last 3 chunks in RAM)**: $10,000\text{ streams} \times 3\text{ chunks} \times 3\text{ MB} = \mathbf{90\text{ GB RAM}}$ across transmuxer fleet.

---

### Drill 28: Distributed Tracing Pipeline (Jaeger) — 1% Sampling
- **Problem**: 100,000 requests/sec across 20 microservices (each request produces 20 spans, $500\text{ bytes}$ per span). Adaptive sampling set to 1% ($0.01$).
- **Calculations**:
  - **Total Spans Generated**: $100,000 \times 20 = 2,000,000\text{ spans/sec}$.
  - **Sampled Spans Ingestion**: $2\text{M} \times 0.01 = \mathbf{20,000\text{ spans/sec}}$.
  - **Daily Trace Storage**: $20,000\text{ spans/s} \times 500\text{ bytes} \times 86,400\text{ s} \approx \mathbf{864\text{ GB/day}}$ in ClickHouse.

---

### Drill 29: Financial Ledger Double-Entry Database IOPS
- **Problem**: Payment platform processes 50M completed transactions/day. Each transaction inserts 1 Charge record + 2 Double-Entry Ledger rows (Debit and Credit).
- **Calculations**:
  - **Daily SQL Inserts**: $50\text{M} \times 3 = 150\text{ Million rows/day}$.
  - **Average Insert QPS**: $\frac{150 \times 10^6}{86,400} \approx \mathbf{1,736\text{ TPS}}$ (Peak: $\mathbf{7,500\text{ TPS}}$).
  - **Database Disk IOPS**: Sustaining 7,500 synchronous WAL fsyncs/sec requires provisioned AWS EBS `io2` SSD with **$15,000\text{ IOPS}$**.

---

### Drill 30: AI Agent Episodic Long-Term Memory Sizing
- **Problem**: 1M enterprise users. Each user accumulates 500 episodic conversation memory chunks (1536-dimensional float32 vector + 500 bytes text).
- **Calculations**:
  - **Total Vectors**: $1\text{M users} \times 500 = \mathbf{500\text{ Million Vectors}}$.
  - **Vector Embedding RAM**: $500\text{M} \times 1536 \times 4\text{ bytes} \approx \mathbf{3.07\text{ TB RAM}}$.
  - **With 8-bit Scalar Quantization ($4\times$)**: $\frac{3.07\text{ TB}}{4} \approx \mathbf{768\text{ GB RAM}}$ across a 12-node Milvus cluster.
