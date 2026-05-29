---
title: Distributed Systems — Index
type: concept
created: 2026-05-12
updated: 2026-05-26
sources:
  - raw/notes/20250220100347-databases.org
tags: [distributed-systems, system-design, databases]
---

# Distributed Systems

Notes captured during system design interview preparation. Covers databases, distributed primitives, and architectural patterns.

## Sub-pages

- [[Distributed Systems/CAP Theorem]] — consistency, availability, partition tolerance trade-offs
- [[Distributed Systems/DynamoDB]] — AWS NoSQL database: data model, indexing, single-table design, DAX, streams
- [[Distributed Systems/Cassandra]] — write-optimized wide-column NoSQL; LSM tree, gossip, consistent hashing
- [[Distributed Systems/Redis]] — in-memory data store: cache, locks, leaderboards, rate limiting, pub/sub
- [[Distributed Systems/PostgreSQL]] — RDBMS: ACID, replication, GIN/GiST/PostGIS indexes, JSONB
- [[Distributed Systems/Database Indexing]] — B-tree, hash, geospatial, inverted indexes; composite and covering indexes
- [[Distributed Systems/Distributed Primitives]] — consistent hashing, gossip protocol, LSM tree, WAL, bloom filter, phi accrual, hinted handoff, saga pattern
- [[Distributed Systems/Load Balancer]] — L4 vs L7, sticky sessions
- [[Distributed Systems/Flink]] — stream processing: operators, windows, watermarks, state management
- [[Distributed Systems/System Design Numbers]] — key metrics and scale triggers for capacity planning
- [[Distributed Systems/gRPC]] — binary RPC for internal service-to-service communication
- [[Distributed Systems/WebSockets]] — persistent full-duplex TCP connection for real-time bidirectional communication
- [[Distributed Systems/WebRTC]] — peer-to-peer UDP communication; STUN/TURN for NAT traversal
- [[Distributed Systems/Sharding]] — partitioning data across machines; shard key selection, strategies, hot spots
- [[Distributed Systems/Kafka]] — distributed event streaming; topics, partitions, consumer groups, fault tolerance

## Database comparison

| Database | CAP | Best for | Avoid when |
|----------|-----|----------|------------|
| [[Distributed Systems/PostgreSQL\|PostgreSQL]] | CP | Complex relationships, ACID, mixed workloads | Extreme write throughput, multi-region |
| [[Distributed Systems/DynamoDB\|DynamoDB]] | AP | Simple key-value, managed scaling | Complex queries, multi-table joins |
| [[Distributed Systems/Cassandra\|Cassandra]] | AP | High write throughput, global scale | Strict consistency, complex queries |
| [[Distributed Systems/Redis\|Redis]] | AP | Caching, locks, real-time features | Persistence-critical data |
