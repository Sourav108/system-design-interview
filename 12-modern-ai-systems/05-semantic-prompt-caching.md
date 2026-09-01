# Semantic Prompt Caching & Cost Optimization

LLM API calls are expensive ($0.005 - $0.03 per 1k tokens) and slow ($1-3\text{s}$). Semantic Prompt Caching intercepts user queries and returns cached responses for semantically equivalent prompts.

---

## 1. Exact vs Semantic Caching

```mermaid
flowchart TD
    Prompt["User Prompt: 'How do I reset my password?'"] --> ExactCheck{1. Exact SHA-256 Hash Match?}
    ExactCheck -->|Hit| ReturnExact[Return Cached Answer: < 1ms, $0 ✅]

    ExactCheck -->|Miss| Embed[2. Generate 1536-dim Embedding]
    Embed --> VectorSearch[3. Vector DB ANN Search in Redis / Milvus]
    VectorSearch --> SimCheck{Cosine Similarity >= 0.96?}
    SimCheck -->|Hit: 'Steps to change password'| ReturnSemantic[Return Cached Answer: 5ms, $0 ✅]
    SimCheck -->|Miss| CallLLM[4. Route to LLM Provider ($0.03, 1500ms)]
```

---

## 2. Threshold Tuning & Guardrails
- **Cosine Threshold $\ge 0.96$**: High confidence; safe for static FAQs, documentation, and policy questions.
- **Dynamic Entities**: Never cache prompts containing dynamic time-sensitive entities (e.g. "What is the stock price of Apple right now?").
- **Cost Reduction**: Typical enterprise production workloads achieve **$20\% - 35\%$ cache hit ratios**, saving hundreds of thousands of dollars in LLM API fees.
