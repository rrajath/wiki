---
title: System Design Numbers
type: concept
created: 2026-05-12
updated: 2026-06-01
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

## Common interview mistakes

### Premature sharding

The single biggest mistake is assuming sharding is always necessary. A few worked examples:

- **Yelp**: 10M businesses × 1 KB = 10 GB; even 10× for reviews = 100 GB — fits comfortably on a single database.
- **Leetcode leaderboard**: 100k competitions × 100k users × 4 B (float rating) = 400 GB — still fits on a single large instance or cache.

Reach for [[concepts/Distributed Systems/Sharding|Sharding]] only when data genuinely won't fit on the largest available single instance.

### Overestimating latency

SSD / database row lookups on indexed columns happen in **sub-millisecond to a few milliseconds**. There is no need to add a caching layer for indexed lookups. Reserve caching for non-indexed or aggregation-heavy queries.

### Over-engineering write throughput

At 5 k writes/sec, candidates often add a message queue. But:

- A well-tuned **PostgreSQL** instance handles **20 k+ writes/sec** for simple inserts.
- Write capacity is actually limited by: complex multi-table transactions, write amplification from excessive indexes, cascading updates, or heavy concurrent reads competing with writes.

**When message queues are justified**: guaranteed delivery against downstream failures, event-sourcing patterns, spikes exceeding 20 k WPS on a single Postgres instance, or decoupling producers from consumers. Before reaching for a queue, first try batch writes, schema/index optimization, connection pooling, or async commits for non-critical writes.

---

Source: [hellointerview.com — Numbers to Know](https://www.hellointerview.com/learn/system-design/deep-dives/numbers-to-know)
