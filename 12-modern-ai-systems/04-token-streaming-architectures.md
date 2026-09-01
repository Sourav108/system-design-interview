# Token Streaming Architectures: Server-Sent Events (SSE) vs WebSockets

Waiting 10–20 seconds for an LLM to generate an entire 500-word essay causes poor user experience. Streaming tokens in real time provides an instant perception of speed.

---

## 1. Protocol Comparison for AI Streaming

| Protocol | Connection Type | Framing Overhead | Suitability for LLM Streaming |
|---|---|---|---|
| **Server-Sent Events (SSE)** | Unidirectional HTTP/2 | 2 bytes (`data: \n\n`) | **Optimal**: Native browser `EventSource`, HTTP proxy friendly, automatic reconnect. |
| **WebSockets** | Full-Duplex TCP | 2 bytes frame | **Overkill**: Unnecessary bi-directional complexity unless client continuously streams audio/video. |
| **gRPC Streaming** | HTTP/2 Multiplexed | Protobuf binary | **Optimal for Inter-Service**: High-throughput service-to-service backend RPCs. |

---

## 2. Server-Sent Events (SSE) Wire Format

```http
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: {"choices": [{"delta": {"content": "Hello"}}]}

data: {"choices": [{"delta": {"content": " world"}}]}

data: [DONE]
```
