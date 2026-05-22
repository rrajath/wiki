---
title: Load Balancer
type: concept
created: 2026-05-12
updated: 2026-05-12
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

*This page is a stub — expand with load balancing algorithms (round-robin, least connections, consistent hashing) as more notes are captured.*
