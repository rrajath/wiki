---
title: Load Balancer
type: concept
created: 2026-05-12
updated: 2026-05-28
sources:
  - raw/notes/20250222100446-load_balancer.org
tags: [distributed-systems, load-balancer, networking, system-design]
---

# Load Balancer

Distributes incoming network traffic across multiple servers to ensure no single server bears too much load.

## L4 vs L7

| Layer | Operates at | Sees | Use when |
|-------|-------------|------|----------|
| **L4** | Transport (TCP/UDP) | IP addresses, ports | Sticky sessions or persistent connections (WebSockets) |
| **L7** | Application (HTTP/HTTPS) | URLs, headers, cookies | Most HTTP APIs; content-based routing |

**Rule of thumb**: if you need sticky sessions or persistent connections (e.g., WebSockets), use L4. Otherwise, use L7.

### L7 details

- **Terminates** the incoming client connection and opens a **new** connection to the backend server — the backend sees the LB's IP, not the client's.
- Can route based on URL path, HTTP headers, cookies, etc.
- More **CPU-intensive** due to packet inspection.
- Better suited for HTTP-based traffic with content-aware routing needs.

### L4 details

- Operates at the TCP level — **faster** but unaware of application content.
- A TCP connection established through an L4 LB stays pinned to the same backend for the life of that session.
- Required for **WebSockets** and other persistent connections.

## Health checks

Load balancers continuously probe backends to detect failures. Unhealthy nodes are removed from rotation automatically.

## Load balancing algorithms

| Algorithm | Behavior |
|-----------|----------|
| **Round robin** | Requests distributed sequentially across servers |
| **Random** | Server selected at random |
| **Least connections** | Route to the server with fewest active connections |
| **Least response time** | Route to the server with the lowest latency |
| **IP hash** | Client IP is hashed to a consistent server (soft sticky sessions at L7) |
