---
title: Optimistic Concurrency Control
type: concept
created: 2026-06-07
updated: 2026-06-07
sources:
  - raw/articles/dealing-with-contention.md
tags: [distributed-systems, concurrency, databases, system-design, occ]
---

# Optimistic Concurrency Control (OCC)

A concurrency strategy that **assumes conflicts are rare and detects them at write time** rather than blocking to prevent them. Protects the same read-modify-write gap as [[Distributed Systems/Pessimistic Locking]] but without holding any lock — everyone proceeds and only the loser retries.

## Mechanism

Add a value to the row that is **guaranteed to change on every write** — a version counter is the canonical choice. Read the row and record the version. When writing, add a `WHERE version = <value-read>` guard. If another transaction wrote first, the version has advanced; the guard matches zero rows and the loser retries.

```sql
-- Both Alice and Bob read: 1 seat, version 42

-- Alice writes first:
BEGIN TRANSACTION;
UPDATE concerts
SET available_seats = available_seats - 1, version = version + 1
WHERE concert_id = 'weeknd_tour'
  AND version = 42;
-- Succeeds. version is now 43.

INSERT INTO tickets (user_id, concert_id, seat_number, price, purchase_time)
VALUES ('alice', 'weeknd_tour', 'A15', 750.00, NOW());
COMMIT;

-- Bob writes against stale version:
BEGIN TRANSACTION;
UPDATE concerts
SET available_seats = available_seats - 1, version = version + 1
WHERE concert_id = 'weeknd_tour'
  AND version = 42;  -- now stale; matches 0 rows
-- App checks affected row count → 0 → ROLLBACK
ROLLBACK;
```

**Zero rows is not an error** — same trap as [[Distributed Systems/Conditional Writes]]. The application must check the affected row count and roll back when it's zero; otherwise the `INSERT` runs and creates a ticket with no seat.

## Version column options

| Option | Safe? | Notes |
|--------|-------|-------|
| Dedicated integer counter (increment on every write) | ✓ Always safe | Best default |
| Last-updated timestamp | ✓ If clock resolution is fine enough | Two fast writes could land on the same millisecond |
| Business value (e.g. seat count, high bid) | Only if monotonic | Subject to ABA problem if value can return to a prior value |

## The ABA problem

If a version value can return to a previous value (A → B → A), an OCC check passes even though the row changed. Example: a restaurant's `review_count` drops from 100 to 99 (deletion) and rises back to 100 (new review) before Bob writes. Bob's `WHERE version = 100` passes, silently overwriting the state of a row that changed twice underneath him.

**Fix**: use a dedicated counter that only ever increments. The value can never return to a prior state, so the ABA cycle is impossible.

Only reuse a business value as a version when it is **guaranteed to move in one direction** — like an ever-increasing high bid in an auction.

## When to prefer OCC over pessimistic locking

| Signal | Choose |
|--------|--------|
| Collisions are common | [[Distributed Systems/Pessimistic Locking\|Pessimistic Locking]] — retries cost more than the lock |
| Collisions are rare (most e-commerce, social features) | OCC — skips locking overhead on the common case |
| Read-heavy workload (many reads, few writes) | OCC — no lock contention on reads |

## Limits

OCC protects conflicts where both transactions touch the same row. When two transactions read overlapping rows but write to different rows, the version check misses the conflict entirely — that is write skew, handled by [[Distributed Systems/Database Isolation Levels|SERIALIZABLE isolation]].

See also: [[Distributed Systems/Dealing with Contention]], [[Distributed Systems/Pessimistic Locking]], [[Distributed Systems/index|Distributed Systems]]
