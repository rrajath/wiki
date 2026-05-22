---
title: CAP Theorem
type: concept
created: 2026-05-12
updated: 2026-05-12
sources:
  - raw/notes/20250118132202-cap_theorem.org
  - raw/notes/20250118132314-consistency_in_distributed_systems.org
  - raw/notes/20250118132420-availability_in_distributed_systems.org
  - raw/notes/20250118132534-partition_tolerance_in_distributed_systems.org
  - raw/notes/20250128193745-cap_theorem_in_system_design_interviews.org
tags: [distributed-systems, cap-theorem, consistency, availability, system-design]
---

# CAP Theorem

In a distributed system, it is impossible to simultaneously guarantee all three of: **Consistency**, **Availability**, and **Partition Tolerance**. You can have at most two.

## The three properties

### Consistency (C)

All clients see the same data at the same time, regardless of which node they connect to. A read always returns the most recent write.

**Example**: Two servers (US and EU). User A updates their profile; the update hits the US server and replicates to EU. If strongly consistent, User B reading from EU sees the updated profile — never stale data.

**When consistency is critical**: booking systems (last seat on a plane), financial transactions, e-commerce inventory. If two users can both "buy" the last item, consistency was violated.

**When eventual consistency is acceptable**: social media profiles, Yelp reviews, restaurant menus. A few seconds of staleness is harmless.

**Types of consistency**:
1. *Strong consistency* — all reads reflect the most recent write.
2. *Causal consistency* — causally related events appear in order.
3. *Read-your-writes* — a user always sees their own updates.
4. *Eventual consistency* — updates propagate eventually; temporary staleness is allowed.

**Design implications of strong consistency**:
- Distributed transactions (writes to DB + cache must coordinate).
- Higher latency (show a spinner; wait for all writes to confirm).
- Possibly limiting to a single-primary write path.

**Nuance**: different parts of the same system can have different consistency requirements. Ticketmaster's CRUD operations can be eventually consistent; booking must be strongly consistent.

### Availability (A)

The system always responds to requests — even if one or more nodes are down. The response may be stale; the key guarantee is that a response is returned.

**Design implications**:
- Multiple read replicas.
- Multi-AZ deployment (e.g., DynamoDB across availability zones).
- Change Data Capture (CDC) pipelines can be strongly consistent while read paths remain eventually consistent.

### Partition Tolerance (P)

The system continues to operate despite a network partition — a communication break between nodes.

In any real distributed system, network partitions happen. Partition tolerance is therefore **not optional** — it must be assumed. The real trade-off is between C and A when a partition occurs.

## The real trade-off: C vs A

Since P is mandatory, the choice is:

| System type | Trade-off |
|-------------|-----------|
| **CP** | During a partition, refuse to respond rather than risk inconsistency |
| **AP** | During a partition, respond with potentially stale data to stay available |
| **CA** | Not viable in a distributed system — requires no partitions |

## In system design interviews

When gathering non-functional requirements, start with CAP:
1. Establish that partition tolerance is required (it always is).
2. Ask: if a partition occurs, should the system stay available (AP) or stay consistent (CP)?
3. Use the answer to justify database and architecture choices.

See: [[Distributed Systems/DynamoDB]], [[Distributed Systems/Cassandra]], [[Distributed Systems/PostgreSQL]]
