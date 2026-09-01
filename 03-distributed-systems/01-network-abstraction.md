# Network Abstractions in Distributed Systems

At the core of all distributed systems lies the network: an asynchronous, packet-switched, unreliable communication medium connecting isolated computing nodes.

---

## 1. The Protocol Stack: Transport to Application

```mermaid
flowchart TD
    subgraph TransportLayer["Transport Layer"]
        TCP["TCP: Reliable, Ordered, Byte-Stream, Congestion Control (3-Way Handshake)"]
        UDP["UDP: Unreliable, Unordered, Datagram, Minimal Overhead (0-RTT)"]
        QUIC["QUIC: UDP-based, Multiplexed Streams, 0-RTT TLS 1.3, No Head-of-Line Blocking"]
    end

    subgraph AppLayer["Application Layer"]
        HTTP1["HTTP/1.1: Text-Based, Pipelining with Head-of-Line Blocking, Heavy Headers"]
        HTTP2["HTTP/2: Binary Framing, Single TCP Connection Multiplexing, Header Compression (HPACK)"]
        HTTP3["HTTP/3: HTTP over QUIC, Independent Stream Flow Control, Fast Roaming"]
        gRPC["gRPC: Protocol Buffers, Strongly Typed Contracts, HTTP/2 Bidirectional Streaming"]
    end

    TCP --> HTTP1
    TCP --> HTTP2
    UDP --> QUIC
    QUIC --> HTTP3
    HTTP2 --> gRPC
```

---

## 2. HTTP/1.1 vs HTTP/2 vs HTTP/3 vs gRPC

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 (QUIC) | gRPC |
|---|---|---|---|---|
| **Transport** | TCP | TCP | UDP (QUIC) | TCP (HTTP/2) |
| **Framing** | Plaintext (ASCII) | Binary Frames | Binary Frames | Binary (Protobuf) |
| **Multiplexing** | No (HOL blocking) | Yes (Application-level) | Yes (Transport-level) | Yes |
| **Header Compression** | None | HPACK | QPACK | HPACK |
| **Stream Independence** | N/A | Packet drop halts all streams | Packet drop only halts affected stream | Packet drop halts all streams |
| **Best For** | Public Web APIs | Web Browsers | Mobile / High Packet Loss | Internal Microservices |

---

## 3. Network Connection Pooling & Keep-Alive

Establishing a TCP connection requires a 3-way handshake ($\text{SYN} \to \text{SYN-ACK} \to \text{ACK}$) plus TLS negotiation ($1\text{ to }2\text{ RTTs}$). In high-throughput distributed systems:
- **Connection Reuse (`Keep-Alive`)**: Keep TCP sockets open across multiple HTTP requests.
- **Connection Pools (HikariCP, Envoy)**: Pre-allocate and maintain a warm pool of connections to downstream databases and microservices, reducing per-request latency from $> 50\text{ms}$ to $< 1\text{ms}$.
