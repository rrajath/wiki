---
title: System Design Numbers
type: concept
created: 2026-05-12
updated: 2026-05-12
sources:
  - raw/notes/20250214215540-numbers_to_know_for_system_design.org
tags: [distributed-systems, system-design, capacity-planning, performance]
---

# Numbers to Know for System Design

Key performance metrics and scale triggers for common system components. Useful for capacity estimation in system design interviews.

## Reference table

| Component | Key Metric | Scale Trigger |
|-----------|-----------|---------------|
| **Caching** | ~1ms latency; 100k+ ops/sec; up to 1TB memory | Hit rate < 80%; latency > 1ms; memory > 80%; cache thrashing |
| **Databases** | Up to 50k TPS; sub-5ms read latency (cached); 64TiB+ storage | Write throughput > 10k TPS; read latency > 5ms uncached; geo-distribution needs |
| **App Servers** | 100k+ concurrent connections; 8–64 cores @ 2–4GHz; 64–512GB RAM (up to 2TB) | CPU > 70%; response latency > SLA; connections near 15k/instance; memory > 80% |
| **Message Queues** | Up to 1M msgs/sec per broker; sub-5ms end-to-end latency; up to 50TB storage | Throughput near 800k msgs/sec; partition count ~200k/cluster; growing consumer lag |

## Key numbers to memorize

- **Cache read latency**: ~1 millisecond (Redis in practice: microseconds).
- **DB read latency (cached)**: sub-5ms.
- **DB write throughput limit**: ~10k TPS before scaling is needed.
- **App server connections**: ~15k per instance before hitting limits.
- **Kafka throughput**: ~800k msgs/sec per broker before needing to scale.

## Usage in interviews

Use these numbers during the capacity estimation step:
1. Estimate QPS (queries per second) from user count and access patterns.
2. Compare against these thresholds to determine when you need multiple instances, caching layers, read replicas, etc.
3. Justify technology choices: "We expect 50k writes/sec, which exceeds a single PostgreSQL instance's comfortable write ceiling — use Cassandra or shard PostgreSQL."

Source: [hellointerview.com — Numbers to Know](https://www.hellointerview.com/learn/system-design/deep-dives/numbers-to-know)
