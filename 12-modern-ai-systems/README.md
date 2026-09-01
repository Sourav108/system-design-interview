# Module 12: Modern AI & LLM Systems Design

A comprehensive, cutting-edge curriculum covering the architecture, serving, memory management, vector indexing, and scaling of **Large Language Models (LLMs) and Generative AI Systems**.

---

## 📚 Lessons in this Module

1. [**01: LLM Inference Fundamentals**](./01-llm-inference-fundamentals.md) — Prefill (compute-bound) vs Decode (memory-bound) phases, TTFT vs ITL metrics.
2. [**02: KV Cache & Memory Management**](./02-kv-cache-and-memory-management.md) — KV Cache sizing formulas, VRAM allocation math, and PagedAttention architecture.
3. [**03: Continuous Batching & vLLM**](./03-continuous-batching-and-vllm.md) — Static vs dynamic vs continuous batching and iteration-level scheduling.
4. [**04: Token Streaming Architectures**](./04-token-streaming-architectures.md) — Server-Sent Events (SSE) vs WebSockets vs gRPC streaming protocols.
5. [**05: Semantic Prompt Caching**](./05-semantic-prompt-caching.md) — Exact SHA-256 vs Vector Cosine similarity prompt caching in Redis.
6. [**06: AI Inference Gateway**](./06-ai-inference-gateway.md) — Model routing, token-per-minute rate limiting, and multi-provider circuit breaking.
7. [**07: Vector Search & HNSW**](./07-vector-search-and-hnsw.md) — Hierarchical Navigable Small World (HNSW) graphs, Cosine vs L2 distance, Product Quantization.
8. [**08: Production RAG Architecture**](./08-rag-systems-architecture.md) — Semantic chunking, Dense + Sparse hybrid search, Cross-Encoder reranking, guardrails.
9. [**09: GPU Cluster Capacity Estimation**](./09-gpu-cluster-capacity-estimation.md) — Sizing VRAM for weights, KV cache, activations, and Tensor Parallelism.
10. [**10: AI Agent Architectures**](./10-ai-agent-architectures.md) — ReAct reasoning loop, function calling, tool execution sandboxes, and episodic memory.
