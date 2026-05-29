---
title: DynamoDB
type: concept
created: 2026-05-12
updated: 2026-05-28
sources:
  - raw/notes/20250128201341-dynamodb.org
  - raw/notes/20250128201410-dynamodb_indexing.org
  - raw/notes/20250128202011-dynamodb_indexing.org
  - raw/notes/20250203181215-dynamodb_single_table_design.org
  - raw/notes/20250216221225-dax_dynamodb_accelerator.org
  - raw/notes/20250216221225-dynamodb_streams.org
tags: [distributed-systems, database, dynamodb, nosql, aws]
---

# DynamoDB

AWS-managed NoSQL key-value and document database. Designed for single-digit millisecond latency at any scale. AP system (availability over consistency by default; strong consistency available for reads).

## Core concepts

- **Tables**: collections of related data.
- **Items**: individual records, each with a unique primary key.
- **Attributes**: data within an item (name, age, etc.). Schema-less — attributes can vary per item.

## Data model

DynamoDB is schema-less:
- **Pros**: flexible, no migrations, only store what you need.
- **Cons**: application must enforce data consistency and handle missing attributes.

## Keys and indexing

### Primary key

Combination of **Partition Key** (required) + **Sort Key** (optional).

- **Partition Key**: hashed via [[Distributed Systems/Distributed Primitives#Consistent Hashing|consistent hashing]] to determine physical node placement.
- **Sort Key**: enables ordering and range queries within a partition. Backed by a B-tree index.

| System | Partition Key | Sort Key |
|--------|--------------|----------|
| Chat app | Chat ID | Message ID (monotonic) |
| E-commerce | User ID | Order ID (monotonic) |
| Social media | User ID | Post ID (monotonic) |

### Secondary indexes

- **GSI (Global Secondary Index)**: different partition key + sort key. Enables queries across all partitions. Example: find all messages by a user across all chat rooms.
- **LSI (Local Secondary Index)**: same partition key, different sort key. Example: sort messages within a chat by attachment count instead of message ID.

Overusing GSIs/LSIs is a sign you should consider a relational DB instead.

## Single table design

All entities share one table; partition and sort keys are carefully designed to support all access patterns.

**Use when**:
- Access patterns are consistent and known upfront.
- No complex joins.
- Transactions involve ≤10 items.

## Architecture

- Uses [[Distributed Systems/Distributed Primitives#Consistent Hashing|consistent hashing]] for partition key placement → scales horizontally.
- Writes go to a single leader node; leader replicates asynchronously to replicas in different AZs.
- Strong consistency reads come from the leader (no replication lag).
- Consistency model is **per-request**, not table-level. Pass `ConsistentRead=true` on individual `GetItem`, `Scan`, or `Query` calls to opt into strong consistency.

## When to use

- Works well for most use cases with simple access patterns.
- Multi-AZ for high availability.
- Use strong consistency reads when stale data is unacceptable.

## When NOT to use

1. **Complex query patterns** — joins, subqueries → use PostgreSQL.
2. **Multi-table transactions** — DynamoDB has low item limits per transaction.
3. **Too many GSIs/LSIs** — signals a relational model is needed.

## DAX (DynamoDB Accelerator)

In-memory cache providing **microsecond** read latency (vs single-digit millisecond for DynamoDB directly). Automatic read/write-through caching.

**Use case**: Yelp-like service where complex aggregations (average rating for a business) are cached in DAX to avoid re-computing from DynamoDB on every request.

**Important caveat**: DAX only auto-invalidates cache items for writes that go through DAX itself. If a record is updated in DynamoDB directly (bypassing DAX), that record in DAX becomes stale. Always use the DAX client instead of the DynamoDB client when DAX is in use.

## DynamoDB Streams

Change Data Capture (CDC): any add/update/delete to a streams-enabled table emits an event.

**Common pattern**: Lambda consumes stream events → pushes to a queue → downstream service processes.

**Use case**: Keeping Elasticsearch in sync with DynamoDB for Ticketmaster's complex geospatial/full-text search queries.
