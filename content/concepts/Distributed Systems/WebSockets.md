---
title: WebSockets
type: concept
created: 2026-05-21
updated: 2026-06-08
sources:
  - raw/notes/20260521075825-websockets.org
tags: [distributed-systems, networking, real-time, system-design, websockets]
---

# WebSockets

A protocol providing a **persistent, full-duplex TCP connection** between client and server, enabling real-time bidirectional communication. Unlike HTTP's request-response model, WebSockets keep a single connection open for the lifetime of the session.

## How it works

WebSocket connections are bootstrapped via the HTTP upgrade mechanism — no new TCP connection is needed:

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server

    C->>S: HTTP GET (Upgrade: websocket)
    S->>C: 101 Switching Protocols
    Note over C,S: Connection is now WebSocket
    C->>S: Binary/text frame
    S->>C: Binary/text frame
    Note over C,S: Bidirectional, until closed
```

1. Client initiates a WebSocket handshake over HTTP (backed by an existing TCP connection)
2. Server responds with `101 Switching Protocols`; the TCP connection is handed off to WebSocket
3. Both sides send frames (text or binary) freely in either direction
4. Connection stays open until either side explicitly closes it

## Key properties

| Property | Value |
|---|---|
| Transport | TCP |
| Direction | Full-duplex (both sides send simultaneously) |
| Initiation | HTTP upgrade (reuses TCP connection) |
| State | Persistent (stateful) |
| Latency | Low — no per-message connection overhead |

## Trade-offs

**Advantages**:
- Full-duplex — server and client send simultaneously without waiting
- Lower latency per message — no per-request headers after the handshake
- Efficient for frequent small messages (gaming state, live prices)
- Wide browser support

**Disadvantages**:
- Stateful connections complicate horizontal scaling and load balancing — a client is pinned to one server
- Reconnection must be handled explicitly by the application
- More complex to implement than simple HTTP polling
- Requires WebSocket-aware infrastructure — every proxy, load balancer, and CDN between client and server must support the protocol; HTTP→WebSocket upgrades silently fail on unaware intermediaries

## When to use

- **Live dashboards and feeds** — stock tickers, sports scores, analytics
- **Collaborative tools** — shared document editors, whiteboards
- **Chat and notifications** — any push-to-client pattern
- **Gaming** — real-time state sync

## Comparison with alternatives

| Protocol | Direction | Persistent | Notes |
|---|---|---|---|
| HTTP polling | Client→Server | No | Simple but wasteful |
| Server-Sent Events (SSE) | Server→Client only | Yes | Good for one-way push |
| WebSockets | Bidirectional | Yes | Best for interactive real-time |
| [[WebRTC\|WebRTC]] | Peer-to-peer | Yes | No server relay; UDP-based |
| [[gRPC\|gRPC]] | Bidirectional stream | Yes | Internal services; binary |

See also: [[concepts/Distributed Systems/index|Distributed Systems]]
