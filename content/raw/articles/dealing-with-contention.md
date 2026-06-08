# Dealing with Contention — Race Conditions in Distributed Systems

Source: https://www.hellointerview.com/learn/system-design/patterns/realtime-updates (linked from real-time updates article cluster)
Captured: 2026-06-07

---

## The Race Condition

Consider buying concert tickets online. There's one seat left for The Weeknd, and Alice and Bob both want it. They each hit "Buy Now" in the same instant. The obvious way to handle a purchase is the one most of us would reach for first. Read the current seat count, check that it's above zero, and if it is, decrement it and sell the ticket.

```sql
-- Read the current count
SELECT available_seats FROM concerts WHERE concert_id = 'weeknd_tour';

-- The app checks available_seats > 0, then writes the new value back
UPDATE concerts
SET available_seats = available_seats - 1
WHERE concert_id = 'weeknd_tour';
```

For a single buyer this is exactly right. The trouble starts when Alice and Bob run it at the same moment. Alice's request reads one seat available. A fraction of a millisecond later, before Alice has written anything back, Bob's request reads the same count and also sees one seat. Both check the number they just read, both conclude there's a seat to sell, and both move on to charge a card. Alice's update commits first and the count drops to zero. Bob's update commits right after, decrements again, and the count slides to negative one.

The real culprit is the gap between two steps the naive code treats as one. Reading the count and writing the new value back aren't a single action, and in between them the world can change.

## Conditional Writes

Start with the simplest case. When your rule is an if about the current data, the database can check it and make the change in a single statement — no locks or version numbers required.

```sql
UPDATE concerts
SET available_seats = available_seats - 1
WHERE concert_id = 'weeknd_tour'
  AND available_seats > 0;
```

For two writes in a transaction:

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

Guard the resource actually being contended:

```sql
UPDATE tickets
SET status = 'sold', user_id = 'user123'
WHERE concert_id = 'weeknd_tour'
  AND seat_number = 'A15'
  AND status = 'available';
```

## Pessimistic Locking

When picking a contiguous block of seats, logic runs in the application — not in a WHERE clause. Explicit row locks close this gap.

```sql
BEGIN TRANSACTION;

SELECT seat_number FROM seats
WHERE concert_id = 'weeknd_tour'
  AND section = 'floor'
  AND status = 'available'
FOR UPDATE;

UPDATE seats
SET status = 'sold', user_id = 'user123'
WHERE concert_id = 'weeknd_tour'
  AND seat_number IN ('A15', 'A16', 'A17', 'A18');

COMMIT;
```

Common failure modes: locking too much for too long; deadlocks from inconsistent ordering. Fix: always acquire locks in a consistent sorted order.

## Optimistic Concurrency Control (OCC)

Assumes conflicts are rare. Read a version value with the row; write only if the version is unchanged.

```sql
-- Alice writes first:
UPDATE concerts
SET available_seats = available_seats - 1, version = version + 1
WHERE concert_id = 'weeknd_tour'
  AND version = 42;
-- Succeeds. version = 43

-- Bob writes against stale version:
UPDATE concerts
SET available_seats = available_seats - 1, version = version + 1
WHERE concert_id = 'weeknd_tour'
  AND version = 42;  -- stale, matches 0 rows → ROLLBACK
```

ABA problem: use a dedicated monotonically-incrementing version column, not a business value that can return to a previous value.

## Database Isolation Levels

| Level | Behavior |
|-------|----------|
| READ UNCOMMITTED | Can see uncommitted changes (rarely used) |
| READ COMMITTED | Can only see committed changes (default in PostgreSQL) |
| REPEATABLE READ | Same data read multiple times stays consistent (default in MySQL) |
| SERIALIZABLE | Transactions appear to run one after another; prevents write skew |

Write skew: two transactions read overlapping rows, each makes a locally valid decision, together they violate an invariant (e.g., both on-call engineers step down simultaneously). SERIALIZABLE catches this; weaker levels don't.

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT count(*) FROM on_call WHERE team_id = 'payments' AND is_active = true;
UPDATE on_call SET is_active = false WHERE engineer_id = 'alice';
COMMIT;
```

## Distributed Locks

When exclusive access must span multiple steps, a wait, or a call to another service — use a lock that lives outside the database.

**Redis with TTL**: `SET key value NX EX 600` — atomic create-if-absent with TTL. Fast and simple; TTL leak is possible if holder stalls past expiry.

**Database column**: `reserved_by` + `reserved_until` on the row, updated with a conditional write. No new infrastructure; slower than cache.

**ZooKeeper/etcd**: ephemeral nodes (ZK) or Raft-based leases (etcd). Strongest consistency; operationally complex.

```sql
UPDATE seats
SET reserved_by = 'user123', reserved_until = NOW() + INTERVAL '10 minutes'
WHERE seat_id = 'A15'
  AND (reserved_until IS NULL OR reserved_until < NOW());
```

## Decision tree

1. **Check is a predicate on the row** → Conditional Write
2. **Read-decide-write; high contention** → Pessimistic Locking (FOR UPDATE)
3. **Read-decide-write; low contention** → Optimistic Concurrency Control
4. **Invariant spans rows** → SERIALIZABLE isolation (or materialize onto a single row)
5. **Hold must outlive a transaction** → Distributed Lock

## Hot partition (celebrity problem)

When everyone contends on one row: sharding doesn't help (same row everywhere). Options:
- Restructure the problem (10 identical auction items instead of 1)
- Make likes/follows eventually consistent
- Queue-based serialization: single worker thread per hot resource; eliminates DB contention at the cost of capped throughput and a SPOF

## Common interview scenarios

- **Online Auction**: OCC with current high bid as version (bids only go up → no ABA risk)
- **Ticketmaster**: Conditional write for seat claim; distributed lock with TTL for the checkout hold window
- **Banking (single DB)**: Pessimistic locking or OCC; transfers spanning services → Saga Pattern
- **Ride Sharing Dispatch**: Distributed lock (DB column + expiry) on driver status during request assignment
- **Flash Sale Inventory**: OCC for stock; distributed lock for cart hold
- **Yelp Reviews**: OCC with dedicated version column on business rating row
