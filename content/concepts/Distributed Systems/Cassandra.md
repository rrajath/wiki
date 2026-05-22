---
title: Cassandra
type: concept
created: 2026-05-12
updated: 2026-05-12
sources:
  - raw/notes/20250211212147-cassandra.org
tags: [distributed-systems, database, cassandra, nosql]
---

# Cassandra

Distributed wide-column NoSQL database. Optimized for write-heavy workloads. AP system (availability over consistency). No ACID or multi-row transaction support.

## Key characteristics

- Wide-column: columns can vary per row in a table.
- Eventually consistent (tunable per query).
- Writes as append-only log (LSM tree).
- Batch-writes, flushes from memory to disk, runs compaction.

## Data model

- **Keyspace**: like a database; contains tables and replication configuration.
- **Table**: contains rows.
- **Row**: identified by a primary key; contains columns with timestamps.
- **Conflict resolution**: last write wins (each column has a timestamp).

```js
// Example: wide-column rows can have different columns
{ "row1": { "col1": 1, "col2": "2" },
  "row2": { "col1": 10, "col3": 3.0 } }
```

## Primary keys

- **Partition Key**: one or more columns determining the partition (via [[Distributed Systems/Distributed Primitives#Consistent Hashing|consistent hashing]]).
- **Clustering Key**: zero or more columns determining sorted order within a partition.

Mirrors [[Distributed Systems/DynamoDB#Keys and indexing|DynamoDB's primary key]] (partition key + sort key).

## Partitioning and replication

Uses [[Distributed Systems/Distributed Primitives#Consistent Hashing|consistent hashing]] for horizontal scaling.

**Replication strategies**:
- *SimpleStrategy*: replication factor N → replicate to next N-1 nodes clockwise on the ring. For dev/test.
- *NetworkTopologyStrategy*: replicate across data centers for physical separation. Recommended for production.

## Consistency levels (tunable)

| Level | Meaning |
|-------|---------|
| `ONE` | Single replica responds |
| `QUORUM` | Majority (n/2 + 1) must respond |
| `ALL` | All replicas must respond |

QUORUM on both reads and writes gives strong consistency without ALL's performance cost.

## Query routing

- Any node can act as query coordinator — no single point of failure.
- Nodes share cluster state via [[Distributed Systems/Distributed Primitives#Gossip Protocol|gossip protocol]].
- Routing uses [[Distributed Systems/Distributed Primitives#Consistent Hashing|consistent hashing]] + replication strategy knowledge.

## Storage model (LSM tree)

Cassandra uses a [[Distributed Systems/Distributed Primitives#Log Structured Merge Tree (LSM Tree)|Log Structured Merge Tree]]:
1. **Commit Log (WAL)**: every write lands here first for durability.
2. **Memtable**: in-memory sorted structure. Accumulates writes.
3. **SSTable**: immutable on-disk file flushed from memtable.

Read path: check memtable → use [[Distributed Systems/Distributed Primitives#Bloom Filter|bloom filter]] to identify candidate SSTables → read SSTables newest-to-oldest.

Compaction runs periodically to consolidate SSTables and remove tombstones.

## Fault tolerance

- [[Distributed Systems/Distributed Primitives#Phi Accrual Failure Detector|Phi Accrual Failure Detector]]: each node independently decides if peers are alive based on heartbeat patterns.
- [[Distributed Systems/Distributed Primitives#Hinted Handoff|Hinted Handoff]]: coordinator stores writes for offline nodes temporarily.

## Design philosophy

Unlike relational databases (entity + relationship + joins), Cassandra is **query-driven**: design your schema around your access patterns. Query efficiency is tied to data layout.

Key considerations:
- Primary key design (partition key, clustering key).
- Partition size limits.
- Data denormalization across tables to support access patterns.

## When to use

- High availability over consistency.
- High write throughput.
- Flexible/sparse schemas (wide-column).
- Clear, stable access patterns known upfront.

## When NOT to use

- Strict consistency required.
- Complex queries: multi-table joins, ad-hoc aggregations.
