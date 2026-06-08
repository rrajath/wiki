---
title: Distributed Systems — Index
type: concept
created: 2026-05-12
updated: 2026-06-08
sources:
  - raw/notes/20250220100347-databases.org
tags: [distributed-systems, system-design, databases]
---

# Distributed Systems

Notes captured during system design interview preparation. Covers databases, distributed primitives, and architectural patterns.

## Sub-pages

- [[CAP Theorem]] — consistency, availability, partition tolerance trade-offs
- [[DynamoDB]] — AWS NoSQL database: data model, indexing, single-table design, DAX, streams
- [[Cassandra]] — write-optimized wide-column NoSQL; LSM tree, gossip, consistent hashing
- [[Redis]] — in-memory data store: cache, locks, leaderboards, rate limiting, pub/sub
- [[PostgreSQL]] — RDBMS: ACID, replication, GIN/GiST/PostGIS indexes, JSONB
- [[Database Indexing]] — B-tree, hash, geospatial, inverted indexes; composite and covering indexes
- [[Distributed Primitives]] — consistent hashing, gossip protocol, LSM tree, WAL, bloom filter, phi accrual, hinted handoff, saga pattern
- [[Load Balancer]] — L4 vs L7, sticky sessions
- [[Flink]] — stream processing: operators, windows, watermarks, state management
- [[System Design Numbers]] — key metrics and scale triggers for capacity planning
- [[gRPC]] — binary RPC for internal service-to-service communication
- [[WebSockets]] — persistent full-duplex TCP connection for real-time bidirectional communication
- [[WebRTC]] — peer-to-peer UDP communication; STUN/TURN for NAT traversal
- [[Sharding]] — partitioning data across machines; shard key selection, strategies, hot spots
- [[Kafka]] — distributed event streaming; topics, partitions, consumer groups, fault tolerance
- [[Zookeeper]] — distributed coordination: service discovery, config management, leader election, distributed locks
- [[Real-time Updates]] — overview of all real-time patterns: polling, SSE, WebSockets, WebRTC, and server-side routing
    - [[Simple Polling]] — baseline: client polls server at regular intervals
    - [[Long Polling]] — server holds request open until data is ready
    - [[Server Sent Events]] — persistent HTTP stream; server→client only

- [[Scaling Reads]] — optimize DB, read replicas, sharding, application caching, CDN
- [[Scaling Writes]] — vertical scaling, write-optimized DBs, sharding, queues, batching, hierarchical aggregation
    - [[Cache Stampede]] — thundering herd on cache expiry; probabilistic early refresh, background refresh
    - [[Cache Invalidation]] — TTL, write-through, cache key versioning, deleted items cache
- [[Dealing with Contention]] — race conditions and five tools to prevent lost updates
    - [[Conditional Writes]] — atomic UPDATE with WHERE predicate; simplest fix
    - [[Pessimistic Locking]] — FOR UPDATE; locks rows across app decision logic
    - [[Optimistic Concurrency Control]] — version columns; detect collision at write time
    - [[Database Isolation Levels]] — four levels; SERIALIZABLE for write skew
    - [[Distributed Locks]] — Redis TTL / DB column / ZooKeeper for cross-transaction holds

## Database comparison

| Database | CAP | Best for | Avoid when |
|----------|-----|----------|------------|
| [[PostgreSQL\|PostgreSQL]] | CP | Complex relationships, ACID, mixed workloads | Extreme write throughput, multi-region |
| [[DynamoDB\|DynamoDB]] | AP | Simple key-value, managed scaling | Complex queries, multi-table joins |
| [[Cassandra\|Cassandra]] | AP | High write throughput, global scale | Strict consistency, complex queries |
| [[Redis\|Redis]] | AP | Caching, locks, real-time features | Persistence-critical data |
