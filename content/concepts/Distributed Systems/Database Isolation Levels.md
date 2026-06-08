---
title: Database Isolation Levels
type: concept
created: 2026-06-07
updated: 2026-06-07
sources:
  - raw/articles/dealing-with-contention.md
tags: [distributed-systems, concurrency, databases, system-design, acid]
---

# Database Isolation Levels

Controls **how much one transaction can see of another's in-flight work**. Weaker levels allow anomalies for better performance; stronger levels prevent more anomalies at higher cost.

## The four standard levels

| Level | Dirty Reads | Non-repeatable Reads | Phantom Reads | Write Skew |
|-------|-------------|----------------------|---------------|------------|
| READ UNCOMMITTED | possible | possible | possible | possible |
| READ COMMITTED *(PostgreSQL default)* | prevented | possible | possible | possible |
| REPEATABLE READ *(MySQL default)* | prevented | prevented | prevented | possible |
| SERIALIZABLE | prevented | prevented | prevented | prevented |

These are **options**, not a ladder — choose based on what anomalies you need to prevent.

## Write skew: the anomaly weaker levels miss

Write skew occurs when two transactions **read overlapping rows** and each makes a locally valid decision, but together they violate a cross-row invariant. No single row is written by both transactions, so a row lock or version check won't help.

**Example**: on-call scheduling rule — at least one engineer must always be on call. Alice and Bob are both on call. Both get sick and try to step down simultaneously.

- Alice reads the schedule, sees Bob is on call → decides it's safe to remove herself → writes `alice.is_active = false`
- Bob reads the same schedule, sees Alice is on call → same decision → writes `bob.is_active = false`
- Both commits succeed. Now nobody is on call.

Neither transaction did anything wrong individually. The database committed both because it never compared what each read against what the other wrote. Only SERIALIZABLE catches this.

## SERIALIZABLE: how it works

SERIALIZABLE guarantees the outcome is equivalent to running transactions **one at a time in some serial order**. The double-resignation has no valid serial ordering (either Alice resigns first → Bob would see himself as last and abort, or vice versa), so the database aborts one with a serialization error.

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT count(*) FROM on_call
WHERE team_id = 'payments'
  AND is_active = true;

-- App sees 2, decides it's safe, writes:
UPDATE on_call SET is_active = false WHERE engineer_id = 'alice';

COMMIT;
-- If Bob's transaction ran the same logic concurrently, one of the two aborts.
```

PostgreSQL implements SERIALIZABLE via Serializable Snapshot Isolation (SSI), which tracks read-write dependencies without taking locks on reads — retries are needed on conflict but reads aren't blocked.

## Cost

SERIALIZABLE makes the database track all read-write dependencies. Every abort throws away work that must be redone. It's the most expensive isolation level and should be reserved for conflicts that genuinely span rows — conditional writes, pessimistic locks, and OCC reach the same result more cheaply when both transactions touch the same row.

**Materialize the conflict where possible**: if the invariant can be expressed as a single row (a team's `on_call_count` column, or a `FOR UPDATE` lock on the team row before any engineer steps down), you turn a cross-row conflict back into same-row contention — cheaper and more predictable.

## Interaction with other tools

| Conflict type | Better tool | Why |
|---------------|-------------|-----|
| Same row, predicate check | [[Distributed Systems/Conditional Writes\|Conditional Writes]] | One atomic statement |
| Same row, app decision | [[Distributed Systems/Pessimistic Locking\|Pessimistic Locking]] or [[Distributed Systems/Optimistic Concurrency Control\|OCC]] | Cheaper than SERIALIZABLE |
| Cross-row invariant | **SERIALIZABLE** | The others miss it |

See also: [[Distributed Systems/Dealing with Contention]], [[Distributed Systems/PostgreSQL]], [[Distributed Systems/index|Distributed Systems]]
