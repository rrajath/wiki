---
title: Database Indexing
type: concept
created: 2026-05-12
updated: 2026-05-12
sources:
  - raw/notes/20250305091929-database_indexing.org
tags: [distributed-systems, database, indexing, b-tree, performance]
---

# Database Indexing

Indexes make reads fast at the cost of disk space and write performance. Every index requires almost as much disk space as the original table and adds overhead to every write (must update table + all indexes).

## B-Tree (default)

Self-balancing tree maintaining sorted data. The default for almost all databases.

**Structure**:
- Each node can have hundreds of children (unlike binary trees with 2).
- Each node contains an array of sorted keys and pointers.
- All leaf nodes at the same depth (balanced).
- A node with k keys has exactly k+1 children.

**Why B-trees are the default**:
1. Sorted order → range queries and `ORDER BY` are efficient.
2. Self-balancing → predictable performance as data grows.
3. Structure matches how databases store data → minimal disk I/O.
4. Handles both equality (`email = 'x'`) and range (`age > 25`) queries.
5. Stable performance through random inserts/deletes.

**Where used**: PostgreSQL (default), DynamoDB (sort key), MySQL, and most RDBMS.

## Hash Indexes

Persistent hashmap: O(1) exact-match lookups. Traded flexibility for speed.

**How it works**: array of buckets → hash value determines bucket → linear probing for collisions.

**When to use**: absolute fastest exact-match, no range queries, plenty of memory.

**Rarely used** in traditional databases (B-tree handles both); more common in in-memory stores like Redis.

## Geospatial Indexes

Three main approaches:

### Geohash
- Converts 2D coordinates to 1D string (base32 encoding).
- Strings with shared prefixes = nearby locations.
- Uses B-tree for prefix/range queries on the resulting strings.
- **Limitation**: locations near each other in reality may not share a prefix if they straddle a major grid division.
- Used by: Redis.

### Quadtree
- Recursively subdivides space into 4 quadrants when a quadrant has too many points.
- Adaptive: dense areas subdivide more; sparse areas stay coarse.
- Less common in production databases.
- Used by: Google Maps (map tiles at zoom levels).

### R-Tree
- Flexible, overlapping rectangles that adapt to actual data shapes.
- Can index both points and arbitrary shapes (polygons, roads) in one structure.
- Default spatial index in PostgreSQL/PostGIS, MySQL.
- **Limitation**: overlapping rectangles may require searching multiple tree branches.

## Inverted Index

Maps terms to document IDs: `word → [doc1, doc3, ...]`.

Used for full-text search. Large storage overhead; updates require touching every term in the document. Used by Elasticsearch, and in PostgreSQL via [[Distributed Systems/PostgreSQL#Full-text search with GIN|GIN]].

## Index optimization strategies

### Composite indexes

Single index on multiple columns in a specific order. The B-tree key is a concatenation of columns.

**Order matters**: index on `(user_id, created_at)` supports queries filtering on `user_id` first; useless for queries filtering only on `created_at`. Most selective column first.

Common patterns:
- Order history: `(customer_id, order_date)`
- Event processing: `(status, priority, created_at)`
- Activity feeds: `(user_id, type, timestamp)`

### Covering indexes

Include all columns needed by the query in the index — no table fetch needed. Niche optimization for high-throughput read paths.

```sql
CREATE INDEX idx_posts_user_include
ON posts(user_id) INCLUDE (title, created_at);
```

**Trade-off**: higher storage overhead; use only when profiling shows the extra fetch is the bottleneck.
