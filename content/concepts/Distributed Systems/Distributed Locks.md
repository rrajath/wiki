---
title: Distributed Locks
type: concept
created: 2026-06-07
updated: 2026-06-07
sources:
  - raw/articles/dealing-with-contention.md
tags: [distributed-systems, concurrency, system-design, redis, zookeeper, locking]
---

# Distributed Locks

A lock that lives **outside the database**, shared across all stateless servers, used when exclusive access must span multiple steps, a wait, or a call to another service — things a single database transaction can't contain.

Database row locks last only as long as the transaction holding them. A checkout flow that holds a seat for ten minutes while the user enters payment details can't hold a database row lock for that duration — it would pin a connection and block every other writer on that row.

## The three options

### Redis with TTL

```
SET seat:A15 user123 NX EX 600
```

`NX` = only set if the key doesn't exist (atomic compare-and-set). `EX 600` = 10-minute TTL. Redis clears the key automatically when it expires, so no cleanup job is needed. Any server can check or set the key.

**Advantage**: fast (sub-millisecond), simple, automatic expiry.

**Weakness**: if the lock holder stalls past the TTL (long GC pause, slow network), Redis hands the lock to the next caller. For a brief window, two clients hold the lock simultaneously. Safe for soft reservations where a duplicate grant is a recoverable hiccup; **not safe** for operations where a double-grant corrupts data. Redis is also a single point of failure.

See also: [[Distributed Systems/Redis]]

### Database columns

Add `reserved_by` and `reserved_until` to the row. Acquiring the lock is a [[Distributed Systems/Conditional Writes|conditional write]]:

```sql
UPDATE seats
SET reserved_by = 'user123', reserved_until = NOW() + INTERVAL '10 minutes'
WHERE seat_id = 'A15'
  AND (reserved_until IS NULL OR reserved_until < NOW());
```

One row updated → lock acquired. Zero rows → someone else holds it. Because the expiry is in the `WHERE` clause, a lapsed hold counts as free — no cleanup job needed.

**Advantage**: no new infrastructure; same consistency guarantees as the rest of the data.

**Weakness**: database writes are slower than cache; the lock row becomes a hot contention point under high load.

### ZooKeeper / etcd

Purpose-built coordination services. ZooKeeper creates **ephemeral nodes** that disappear when the client session ends — natural cleanup for crashed processes. etcd uses Raft consensus to maintain leases across node failures.

Both provide strong consistency guarantees during network partitions and leader failures — what Redis's TTL approach can't guarantee.

**Advantage**: robust; designed for complex distributed failure scenarios.

**Weakness**: operational complexity; requires running and maintaining a separate coordination cluster.

See also: [[Distributed Systems/Zookeeper]]

## Comparison

| Option | Speed | Consistency | Complexity | Failure behavior |
|--------|-------|-------------|------------|------------------|
| Redis TTL | Fastest | Weak (TTL leak) | Low | Dual-grant possible |
| DB column | Medium | Strong | Low | No new infra |
| ZooKeeper/etcd | Medium | Strongest | High | Automatic cleanup |

## When to use

- **Seat hold during checkout** — user selects a seat; app holds it for 10 minutes while payment is filled in; prevents users from completing payment only to find the seat taken
- **Ride dispatch** — driver status set to `pending_request` while request is being evaluated; prevents duplicate dispatch to the same driver
- **Multi-step operations** — any flow where the "take ownership" window spans more than a single database transaction

## When NOT to use

If exclusive access fits inside a single database transaction, reach for [[Distributed Systems/Conditional Writes]], [[Distributed Systems/Pessimistic Locking]], or [[Distributed Systems/Optimistic Concurrency Control|OCC]] instead. Distributed locks add a new component and new failure modes — use them only when the scope genuinely requires it.

See also: [[Distributed Systems/Dealing with Contention]], [[Distributed Systems/index|Distributed Systems]]
