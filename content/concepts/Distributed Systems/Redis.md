---
title: Redis
type: concept
created: 2026-05-12
updated: 2026-05-12
sources:
  - raw/notes/20250211211636-redis.org
tags: [distributed-systems, redis, cache, in-memory, database]
---

# Redis

In-memory data store. Single-threaded with atomic operations. Read latency in microseconds; write throughput ~100k ops/second. AP system.

## Data structures

| Structure | Use |
|-----------|-----|
| Strings | Simple key-value cache |
| Hashes | Object storage |
| Lists | Ordered collections, queues |
| Sets | Unique member collections |
| Sorted Sets | Ordered by score (leaderboards, priority queues) |
| Bloom Filters | Probabilistic membership testing |
| Geospatial Indexes | Proximity search |
| Time Series | Time-ordered metrics |

## Communication patterns

- **Pub/Sub**: broadcast messages to subscribers.
- **Streams**: append-only log, like Kafka topics; consumed via consumer groups (`XADD`, `XREADGROUP`, `XCLAIM`).

## Infrastructure configurations

- Single node or cluster.
- In cluster mode, Redis clients cache "hash slot" → node mappings for direct routing.
- Nodes share cluster state via [[Distributed Systems/Distributed Primitives#Gossip Protocol|gossip protocol]].
- **Key principle**: Redis doesn't solve scaling for you. How you structure your keys determines how you scale.

## Capabilities

### Cache
Most common use. Keys and values map to a distributed hashmap across nodes. TTL prevents stale reads and manages memory.

### Distributed Lock
Used for consistency on concurrent operations (Ticketmaster seat booking, Uber ride assignment).
- Simple lock: `INCR` key atomically. If response is `1`, you own the lock; otherwise someone else does. Add TTL to avoid deadlock.
- Complex locks: Redlock algorithm.
- **Note**: if your DB already provides consistency, don't add Redis locks — it adds complexity without benefit.

### Leaderboards
Sorted Sets maintain ordered data queryable in O(log n):
```
ZADD tiger_posts 500 "SomeId1"
ZREMRANGEBYRANK tiger_posts 0 -5  # keep top 5
```

### Rate Limiting
Fixed window: `INCR` on rate-limiter key. If count > N, reject. Set `EXPIRE` at window boundary W to reset.

### Proximity Search
Native geospatial with `GEOADD` and `GEORADIUS`. Uses geohashes under the hood. O(N+log(M)) search time.

### Event Sourcing
Redis Streams as append-only durable logs. Consumer groups enable distributed consumption.

## Hot key problem

When a single key receives massive traffic, the node hosting it becomes a bottleneck.

**Mitigations**:
- Client-side in-memory cache.
- Store same data under multiple keys; randomize routing.
- Add read replicas for the hot node; dynamically scale with load.

## Single-threaded atomicity

All operations serialize. Useful for race conditions:

**Example** (Tinder swipe match): key `user1:user2` = swipe result. Two simultaneous right-swipes serialize in Redis: write 1 → check inverse → write 2 → check inverse. No race condition.

## When to use Redis

- Caching (primary use case).
- Distributed locks where DB doesn't provide consistency.
- Real-time leaderboards, rate limiting, proximity search.
- Simple event streaming (if Kafka is overkill).

## When NOT to use

- Primary data store for persistence-critical data (risk of data loss).
- Complex query patterns (use a proper DB instead).
