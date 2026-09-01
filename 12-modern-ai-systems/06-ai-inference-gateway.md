# AI Inference Gateway & Multi-Provider Resiliency

A specialized API Gateway layer sitting between frontend applications and backend LLM providers (OpenAI, Anthropic, self-hosted vLLM).

---

## 1. Gateway Responsibilities

```mermaid
flowchart TD
    ClientApp[Client App] --> AIGW[AI Inference Gateway]

    subgraph GatewayModules["Core Gateway Responsibilities"]
        M1[1. Token-Per-Minute (TPM) & RPM Rate Limiting]
        M2[2. Semantic Prompt Caching (Redis)]
        M3[3. PII Masking & Prompt Injection Defense]
        M4[4. Multi-Provider Fallback & Load Balancing]
    end
    AIGW --> GatewayModules

    GatewayModules -->|Primary| OpenAI[OpenAI GPT-4o]
    GatewayModules -.->|Fallback on 429 / 503| Anthropic[Anthropic Claude 3.5]
    GatewayModules -.->|Cost Sensitive Routing| SelfHosted[Self-Hosted vLLM GPU Cluster]
```

---

## 2. Multi-Provider Circuit Breaking
If OpenAI returns HTTP 429 (Rate Limit Exceeded) or HTTP 503 (Outage), the gateway's Resilience4j circuit breaker automatically reroutes the prompt to Anthropic Claude or a self-hosted Llama 3 model in $< 200\text{ms}$.
