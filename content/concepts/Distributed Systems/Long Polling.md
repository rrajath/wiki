---
title: Long Polling
type: concept
created: 2026-06-05
updated: 2026-06-05
sources:
  - raw/notes/20260605182700-long_polling.org
tags: [distributed-systems, networking, real-time, system-design, polling]
---

# Long Polling

An upgrade over [[Simple Polling]] that reduces latency by having the **server hold the HTTP request open** until new data is available, then respond and let the client immediately re-issue the request.

## How it works

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server

    C->>S: GET /updates (timeout=30s)
    Note over S: holds connection open...
    Note over S: data arrives
    S->>C: 200 OK (data)
    C->>S: GET /updates (timeout=30s)
    Note over S: holds connection open...
```

1. Client sends request
2. Server blocks the response until data is ready or a timeout elapses
3. Server sends response; client immediately makes another request

The "call-back" cycle means there is always a small gap between when the server gets new data and when the client actually receives it (the re-request RTT).

## Trade-offs

| | |
|---|---|
| **HTTP compatible** | Builds on standard HTTP; no special infrastructure |
| **Lower latency than simple polling** | Client gets data as soon as it arrives, not at the next interval |
| **Stateless server-side** | No persistent connection state between requests |

| | |
|---|---|
| **Re-request latency** | The round-trip to re-issue the request adds a small lag after each delivery |
| **Server resource pressure** | Held connections consume server threads/file descriptors |
| **Browser connection limits** | Browsers cap concurrent connections per domain; long polling eats into that budget |
| **Monitoring friction** | Long-lived requests look like hangs in dashboards and load-balancer logs |

## When to use

- Updates are infrequent and [[Simple Polling]]'s interval latency is unacceptable
- A long async process needs to signal completion as soon as it finishes (e.g. payment processing, job queues)
- Full [[WebSockets]] or [[Server Sent Events]] infrastructure is impractical

## Comparison

| Mechanism | Latency | Connection state | Complexity |
|---|---|---|---|
| [[Simple Polling\|Simple Polling]] | interval-bounded | none | minimal |
| **Long Polling** | low (held until data) | held per request | low |
| [[Server Sent Events\|SSE]] | near real-time | persistent (server→client) | low |
| [[WebSockets\|WebSockets]] | near real-time | persistent (bidirectional) | medium |

See also: [[Real-time Updates]], [[concepts/Distributed Systems/index|Distributed Systems]]
