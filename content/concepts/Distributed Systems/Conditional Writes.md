---
title: Conditional Writes
type: concept
created: 2026-06-07
updated: 2026-06-07
sources:
  - raw/articles/dealing-with-contention.md
tags: [distributed-systems, concurrency, databases, system-design, sql]
---

# Conditional Writes

The simplest contention fix: collapse the read-check-write into **one atomic SQL statement** by embedding the guard directly in the `WHERE` clause. When the database evaluates the guard and writes, no other transaction can slip in between — the check and the decrement are indivisible.

## Pattern

```sql
UPDATE concerts
SET available_seats = available_seats - 1
WHERE concert_id = 'weeknd_tour'
  AND available_seats > 0;
```

If Alice and Bob run this simultaneously, the database serializes them at the row level. Bob's update re-checks `available_seats > 0` against the row Alice already decremented and matches zero rows — one seat, one sale.

## The zero-rows trap

A `WHERE` clause that matches nothing is **not an error**. The `UPDATE` succeeds silently. If a follow-up `INSERT` runs unconditionally, Bob gets a ticket for a seat he never got.

Fix: make the insert conditional on the update having actually matched something.

```sql
BEGIN TRANSACTION;

WITH reservation AS (
  UPDATE concerts
  SET available_seats = available_seats - 1
  WHERE concert_id = 'weeknd_tour'
    AND available_seats > 0
  RETURNING concert_id
)
INSERT INTO tickets (user_id, concert_id, seat_number, purchase_time)
SELECT 'user123', concert_id, 'A15', NOW()
FROM reservation;

COMMIT;
```

`RETURNING` only emits a row when the `UPDATE` actually changed one. The `INSERT … SELECT` draws from that result — no row → no ticket.

## Guard the right resource

A seat count tells you whether a ticket exists, not whether *this* ticket is still available. With 20 seats left and two buyers targeting A15:

```sql
-- Wrong: both buyers decrement the counter and both insert ticket A15
UPDATE concerts SET available_seats = available_seats - 1 WHERE available_seats > 0;

-- Right: guard the ticket row itself
UPDATE tickets
SET status = 'sold', user_id = 'user123'
WHERE concert_id = 'weeknd_tour'
  AND seat_number = 'A15'
  AND status = 'available';
```

The second buyer's `WHERE status = 'available'` is false after the first sale — one ticket, one winner.

**Rule**: the guard must protect the thing being contended. A counter answers "is there a ticket?" Only the ticket row answers "is *this* ticket still available?"

## Limits

Conditional writes work when the entire decision fits in a `WHERE` clause. When the decision requires application logic between the read and write (e.g., finding a contiguous block of open seats), the database can't re-evaluate it — reach for [[Pessimistic Locking]] instead.

See also: [[Dealing with Contention]], [[concepts/Distributed Systems/index|Distributed Systems]]
