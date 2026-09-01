# Module 07: End-to-End System Design Case Studies

The master curriculum of **20 real-world system design case studies**, covering social networks, distributed streaming, geospatial mapping, developer platforms, cloud lakehouses, and modern AI/LLM infrastructure.

---

## 📚 Catalog of 20 Master Case Studies

| ID | Case Study | Industry Domain | Key Architectural Patterns |
|---|---|---|---|
| [**01**](./01-youtube-video-streaming.md) | **YouTube / Netflix Video Streaming** | Media & Streaming | HLS/DASH Adaptive Bitrate, Distributed Transcoding, Origin Shielding |
| [**02**](./02-quora-qa-platform.md) | **Quora / StackOverflow Knowledge Platform** | Social Q&A | Relational Master-Replica, Elasticsearch BM25, Sharded Upvote Counters |
| [**03**](./03-google-maps-navigation.md) | **Google Maps / Navigation Platform** | Geospatial | Contraction Hierarchies Graph Routing, S2 Spatial Cells, Live Traffic Flink |
| [**04**](./04-yelp-proximity-search.md) | **Yelp / Proximity Search Service** | Spatial Search | Google S2 Hilbert Curves, 9-Neighbor Proximity, In-Memory Spatial Grid |
| [**05**](./05-uber-ride-hailing.md) | **Uber / Lyft Ride Hailing Platform** | Real-Time Transport | 1.25M GPS/s Ingestion, Redis S2 Index, Redlock Driver Lease, Surge Flink |
| [**06**](./06-twitter-social-network.md) | **Twitter / X Global Social Network** | Microblogging | Hybrid Fan-Out, Snowflake 64-bit IDs, Redis Sorted Sets Timeline Cache |
| [**07**](./07-newsfeed-aggregator.md) | **Facebook Newsfeed Aggregator** | Social Feed | TAO Graph Cache, Two-Stage ML Ranking Funnel, Candidate Pools |
| [**08**](./08-instagram-photo-sharing.md) | **Instagram / Flickr Photo Platform** | Media Sharing | Direct-to-S3 Presigned Uploads, WebP Thumbnail Workers, Sharded PostgreSQL |
| [**09**](./09-tinyurl-url-shortener.md) | **TinyURL / Bitly URL Shortener** | Web Ingress | Base62 Bijective Encoding, Snowflake IDs, Redis Cache-Aside, HTTP 302 |
| [**10**](./10-distributed-web-crawler.md) | **Distributed Web Crawler (Googlebot)** | Search Engine | Priority & Politeness Frontiers, Bloom Filter Deduplication, c-ares DNS |
| [**11**](./11-whatsapp-realtime-chat.md) | **WhatsApp / Discord Real-Time Chat** | Messaging | 50M Netty WebSockets, Signal Protocol E2EE, Cassandra Offline Queue |
| [**12**](./12-typeahead-search-autocomplete.md) | **Google / Amazon Search Autocomplete** | Search & Discovery | Prefix Trie, Top-5 Precomputed Node Caches, Client Debounce, Spark |
| [**13**](./13-google-docs-collaborative-editor.md) | **Google Docs Collaborative Editor** | Real-Time Docs | Operational Transformation (OT), Multi-Cursor Presence, S3 Snapshots |
| [**14**](./14-distributed-deployment-pipeline.md) | **Distributed CI/CD (GitHub Actions)** | Developer Tools | DAG Workflow Engine, Firecracker MicroVM Sandboxes, Canary Rollouts |
| [**15**](./15-stripe-payment-gateway.md) | **Stripe / PayPal Payment Gateway** | Financial Ledgers | Idempotency Keys, Redis Mutex Locks, Double-Entry Bookkeeping SQL |
| [**16**](./16-leetcode-online-judge.md) | **LeetCode / HackerRank Online Judge** | Dev Assessment | gVisor / cgroups v2 Sandboxes, seccomp-bpf, Redis Contest Leaderboards |
| [**17**](./17-chatgpt-conversational-ai.md) | **ChatGPT Conversational AI Platform** | Modern AI | SSE Token Streaming, PagedAttention KV Cache, Continuous Batching vLLM |
| [**18**](./18-distributed-data-lakehouse.md) | **Apache Iceberg / Snowflake Lakehouse** | Data Lakehouse | Columnar Parquet on S3, ACID Manifest Snapshots, Partition Pruning, Trino |
| [**19**](./19-llm-customer-support-agent.md) | **Autonomous LLM Support Agent & RAG** | AI & Support | Hybrid RAG (Milvus + BM25), NeMo Guardrails, Safe Tool Calling |
| [**20**](./20-ai-coding-assistant.md) | **AI Coding Assistant (Copilot / Cursor)** | Developer AI | AST Tree-sitter, Speculative Decoding, Prefix KV Cache, <150ms TTFT |
