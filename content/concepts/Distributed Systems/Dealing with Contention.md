---
title: Dealing with Contention
type: concept
created: 2026-06-07
updated: 2026-06-07
sources:
  - raw/articles/dealing-with-contention.md
tags: [distributed-systems, concurrency, databases, system-design, locking]
---

# Dealing with Contention

When multiple clients race to modify the same data, correctness requires closing the gap between reading a value and acting on it. Five tools exist, each suited to a different shape of write.

## The core problem: the lost update

A **race condition** in the read-modify-write pattern: two requests read the same value, both decide to act on it, and both write back — so one silently overwrites the other.

```mermaid
sequenceDiagram
    participant A as Alice
    participant DB as Database
    participant B as Bob

    A->>DB: SELECT seats → 1
    B->>DB: SELECT seats → 1
    A->>DB: UPDATE seats = 0 ✓
    B->>DB: UPDATE seats = -1 ✗
```

Both Alice and Bob believe they won the seat. Only one is right, but the database doesn't know that.

## Decision tree

```mermaid
flowchart TD
    Q1{Check is a predicate\non the row being written?}
    Q1 -->|Yes| CW[Conditional Write]
    Q1 -->|No| Q2{Collisions frequent?}
    Q2 -->|Yes| PL[Pessimistic Locking]
    Q2 -->|No| OCC[Optimistic Concurrency Control]
    PL --> Q3{Invariant spans\nnon-overlapping rows?}
    OCC --> Q3
    Q3 -->|Yes| SER[SERIALIZABLE Isolation]
    Q3 -->|No| Q4{Hold must outlive\na single transaction?}
    Q4 -->|Yes| DL[Distributed Lock]
    Q4 -->|No| DONE[Done]
```

## The five tools

| Tool | Use when | Avoid when | Complexity |
|------|----------|------------|------------|
| [[Distributed Systems/Conditional Writes\|Conditional Writes]] | Check is a WHERE predicate | Decision needs app logic | Low |
| [[Distributed Systems/Pessimistic Locking\|Pessimistic Locking]] | Read-decide-write; high contention | Low contention | Low |
| [[Distributed Systems/Optimistic Concurrency Control\|OCC]] | Read-decide-write; rare collisions | High contention → retry storm | Medium |
| [[Distributed Systems/Database Isolation Levels\|SERIALIZABLE]] | Write skew across non-overlapping rows | Hot paths (abort/retry cost) | Medium |
| [[Distributed Systems/Distributed Locks\|Distributed Locks]] | Hold must span wait, external call, or multi-step | Row guard inside one TX covers it | Medium |

## Hot partition (celebrity problem)

When everyone contends on the **same row**, standard scaling stops helping: sharding routes to the same row, load balancing queues on the same database lock, read replicas don't help because the fight is over writes.

Options (pick in order of preference):
1. **Restructure the problem** — 10 identical auction items instead of 1; eventual consistency for social follow counts
2. **Queue-based serialization** — single worker thread per hot resource; turns concurrent contention into a serial queue; caps throughput but eliminates DB lock storms

## Common interview scenarios

| Scenario | Primary tool |
|----------|--------------|
| Last concert ticket | Conditional Write on ticket row (`status = 'available'`) |
| Group seat selection | Pessimistic Locking (`FOR UPDATE` on open seats) |
| Online auction bids | OCC with current high bid as version |
| On-call scheduling | SERIALIZABLE (write skew across engineer rows) |
| Checkout seat hold | Distributed Lock with TTL |
| Bank transfer (single DB) | Pessimistic Locking or OCC |
| Bank transfer (cross-service) | [[Distributed Systems/Distributed Primitives\|Saga Pattern]] |
| Ride dispatch | Distributed Lock on driver status |

See also: [[Distributed Systems/index|Distributed Systems]], [[Distributed Systems/PostgreSQL]]
