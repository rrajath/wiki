---
title: Sharding
type: concept
created: 2026-05-26
updated: 2026-05-26
sources:
  - raw/notes/20260523071945-sharding.org
tags: [distributed-systems, system-design, databases, sharding, scalability]
---

# Sharding

Sharding is a strategy for splitting data **across multiple machines**. It is distinct from partitioning, which splits data into multiple tables on the *same* machine.

## Partitioning vs Sharding

- **Horizontal partitioning** — split by rows (e.g., orders by year)
- **Vertical partitioning** — split by columns (frequently-accessed columns moved to a separate table)
- **Sharding** — horizontal partitioning extended across multiple physical machines

Two decisions define a sharding design:
1. **What to shard by** — the field/column (shard key) that groups records
2. **How to distribute** — the rule that assigns groups to specific shards

## Choosing a Shard Key

A bad shard key causes uneven distribution, hot spots, and scatter-gather queries (hitting every shard). A good shard key has three properties:

| Property | Explanation |
|----------|-------------|
| High cardinality | Many unique values (e.g., `user_id` rather than `country`) |
| Even distribution | Values spread uniformly across shards |
| Query alignment | Most frequent queries hit a single shard |

## Sharding Strategies

**Range-based sharding** — groups records by a continuous value range (e.g., order date). Simple and intuitive, but risks skew: users almost always query recent orders, leaving older shards idle.

**Hash-based sharding** — applies a hash function to the shard key to assign a partition. Produces even distribution. The downside is that adding or removing shards requires re-assigning data — this is where [[Distributed Systems/Distributed Primitives|Consistent Hashing]] helps.

**Directory-based sharding** — a lookup table records which shard each record lives on. Maximum flexibility (you can relocate a hot user to a dedicated shard), but the lookup adds latency and the directory service is a single point of failure.

## Challenges

### Hot Spots

A hot spot occurs when many requests target the same shard — e.g., a shard key of `user_id` where Taylor Swift's ID is queried constantly, or a date-based shard where the "today" shard gets all writes.

Mitigations:
1. **Isolate hot keys** — move them to a dedicated shard
2. **Compound shard key** — e.g., `(user_id, date)` distributes one user's records across multiple shards
3. **Dynamic shard splitting** — some databases split a shard automatically when it grows too large

### Cross-Shard Operations

Queries that don't align with the shard key must scatter to every shard and aggregate the results. Example: "top 10 most popular posts" when sharding by `user_id`.

Mitigations:
1. **Cache results** — expensive first query; fast subsequent ones
2. **Denormalize** — store related data together (duplicates data but avoids scatter)
3. **Accept the cost** — if the query is rare, scatter-gather may be acceptable

### Distributed Transactions

A single-database transaction is straightforward. With sharding, a transaction that spans two shards requires a distributed transaction protocol (e.g., two-phase commit or a saga pattern) because the shards don't share a transaction log. See [[Distributed Systems/Distributed Primitives]].

## When to Propose Sharding

Sharding is not the first solution. Establish why a single database won't work, then bring it up when hitting one of these limits:

- **Storage**: "500M users × 5 KB = 2.5 TB. A single Postgres instance handles this today, but at 10× growth we need to shard."
- **Write throughput**: "50K writes/sec at peak exceeds what a single primary can sustain."
- **Read throughput**: "100M DAU with multiple queries each — read replicas alone won't be enough."

Formula: **identify the bottleneck → explain why a single database won't scale → propose sharding**.

## Interview Walkthrough (Social Media Example)

1. **Propose the shard key** — "Most queries are user-centric (feed, followers, likes), so shard by `user_id`."
2. **Choose the distribution strategy** — "Hash-based sharding with consistent hashing for even distribution and easier rebalancing."
3. **Name the trade-offs** — "Global queries (trending posts) hit all shards. Mitigate by pre-computing trending content with a background job and caching it."
4. **Address growth** — "Start with 64 shards. Consistent hashing means adding shards only moves a fraction of data."

## Related Pages

- [[Distributed Systems/Distributed Primitives]] — consistent hashing, which makes hash-based resharding tractable
- [[Distributed Systems/CAP Theorem]] — sharding affects consistency/availability trade-offs
- [[Distributed Systems/System Design Numbers]] — capacity thresholds that trigger the sharding conversation
