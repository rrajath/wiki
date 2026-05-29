---
title: PostgreSQL
type: concept
created: 2026-05-12
updated: 2026-05-28
sources:
  - raw/notes/20250218080308-postgresql.org
  - raw/notes/20250218084743-generalized_inverted_index.org
tags: [distributed-systems, database, postgresql, rdbms, sql]
---

# PostgreSQL

Relational database with strong ACID guarantees, rich indexing (B-tree, GIN, GiST, PostGIS), and excellent support for both structured and semi-structured data (JSONB). The "start here" default for most system designs.

## Key indexes

### Full-text search with GIN (Generalized Inverted Index)

Works like the index at the back of a book: for each word, store all locations where it appears. Supports word stemming, relevance ranking, multi-language, AND/OR/NOT queries.

Good enough for many applications without setting up Elasticsearch. Limitations: no sophisticated relevancy scoring, no faceted search, no "search as you type," no distributed search at very large scale.

### JSONB with GIN

Store flexible metadata in a single JSONB column and query it efficiently. Example: post metadata (location, mentions, hashtags, media) as one column rather than many nullable columns.

### Geospatial with PostGIS

Extension adding spatial data support. Uses GiST indexes (R-tree under the hood) for efficient spatial queries:
- Points, lines, polygons.
- Distance calculations (straight-line, driving).
- Spatial operations (intersection, containment).

### Covering indexes

Include extra columns in the index so Postgres can answer queries from the index alone, without fetching the actual row:

```sql
CREATE INDEX idx_posts_user_include
ON posts(user_id) INCLUDE (title, created_at);
```

Niche optimization — use when profiling reveals the extra fetch is a bottleneck.

### Partial indexes

Index only a subset of rows:

```sql
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';
```

Smaller index, faster writes, efficient queries for the common case.

## Write performance

1. **WAL (Write Ahead Log)**: write is logged to disk first → transaction is durable even if server crashes.
2. **Buffer Cache**: changes applied to in-memory data pages (dirty pages).
3. **Background Writer**: async flush of dirty pages to disk (checkpoint or memory pressure).
4. **Index Updates**: every index must be updated on write — more indexes = slower writes.

## Write performance optimizations

When raw write throughput becomes a bottleneck:

- **Vertical scaling**: NVMe disks improve WAL performance; more RAM increases buffer cache size; more CPU cores help parallel operations.
- **Batch processing**: buffer multiple writes in application memory and commit them as a single transaction. Trade-off: a crash mid-batch loses all buffered writes.
- **Write offloading**: send analytics or aggregated metrics to a queue (e.g. Kafka) and have background workers flush to Postgres asynchronously. Decouples write spikes from DB pressure.
- **Table partitioning**: split a large table into smaller partitions (most commonly by time). Different sessions can write to different partitions simultaneously; index updates touch only the relevant partition.
- **Sharding**: distribute data across multiple Postgres nodes. Postgres has no built-in sharding — use Citus or implement manually. Shard by user ID so all data for a user lives on one node.

## Replication

| Type | Consistency | Latency |
|------|-------------|---------|
| Synchronous | Strong | High |
| Asynchronous | Eventually consistent (replication lag) | Low |

- **Read scaling**: multiple read replicas; reads → replicas, writes → primary. Watch for replication lag (read-your-writes issue).
- **High availability**: promote a replica to primary if primary fails.

## Data consistency

ACID transactions. Three isolation levels:

| Level | Behavior |
|-------|----------|
| Read Committed (default) | Each query sees commits made before it ran; non-repeatable reads possible |
| Repeatable Read | Snapshot at transaction start; same query returns same results |
| Serializable | Full serial ordering; prevents all anomalies; requires retry on conflict |

Row-level locking with `SELECT ... FOR UPDATE` prevents concurrent writes to the same row.

**Optimistic Concurrency Control (OCC)**: instead of locking upfront, read the data along with a version number or timestamp. At commit time, check whether the version has changed. If it has, another transaction modified the row — retry. Lower contention than pessimistic locking for read-heavy workloads.

## When to use

Start with Postgres; switch if a specific limitation is hit:
- Complex relationships and joins.
- Strong ACID guarantees.
- Rich query patterns (full-text, geospatial, JSONB).
- Mixed structured + semi-structured data.

## When NOT to use

- **Extreme write throughput** → Cassandra or Redis.
- **Multi-region, globally distributed writes** → CockroachDB (ACID) or Cassandra/DynamoDB (eventual).
- **Simple key-value access** → Redis or DynamoDB (PostgreSQL is overkill).
