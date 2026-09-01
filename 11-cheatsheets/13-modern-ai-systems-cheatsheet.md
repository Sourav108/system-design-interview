# Modern AI & LLM Systems Design Cheat Sheet

| AI Subsystem | Core Architectural Mechanism | Performance / Metric Target |
|---|---|---|
| **Inference Phases** | Prefill (Compute-bound GEMM) vs Decode (Memory-bandwidth bound autoregressive) | TTFT $< 500\text{ms}$; ITL $< 30\text{ms}$ |
| **KV Cache** | Retains Key & Value tensors in VRAM; PagedAttention virtual paging | $320\text{ KB/token}$ for 70B GQA |
| **Batching** | Continuous Batching (Iteration-level scheduling in vLLM) | $3\times - 5\times$ higher token throughput |
| **Token Streaming** | Server-Sent Events (SSE) over HTTP/2 | Sub-millisecond token egress framing |
| **Prompt Caching** | Exact SHA-256 + Semantic Vector Cosine similarity ($\ge 0.96$) | $20\% - 35\%$ LLM API cost savings |
| **Vector Search** | Hierarchical Navigable Small World (HNSW) multi-layer graphs | $\mathcal{O}(\log N)$ ANN search $< 10\text{ms}$ |
| **RAG Pipeline** | Semantic Chunking $\to$ Hybrid (Vector + BM25) $\to$ Cross-Encoder Rerank | $> 95\%$ Grounded accuracy |
