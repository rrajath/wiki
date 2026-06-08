---
title: Cache Invalidation
type: concept
created: 2026-06-08
updated: 2026-06-08
sources:
  - raw/notes/20260608105252-scaling_reads.org
tags: [distributed-systems, caching, system-design, redis, consistency]
---

# Cache Invalidation

Determining when and how to remove or update stale data from a cache. Famously the hardest part of caching: invalidate too aggressively and you lose the performance benefit; invalidate too lazily and you serve wrong data.

The challenge compounds in multi-layer systems — application cache, CDN edges, and browser caches may all hold copies of the same data.

## Strategies

### TTL (Time-to-Live)

Entries expire automatically after a fixed duration.

**When to use**: data that can tolerate brief staleness (product catalog, recommendation scores).

**Problem**: a write at T=0 is cached at T=1 and stays stale until T=TTL. Users may see outdated data for the full TTL window. Choosing the right TTL is a trade-off between freshness and cache hit rate.

### Write-through invalidation

Delete or update the cache entry immediately after writing to the database, in the same request path.

```
1. Write to DB
2. DELETE cache_key (or SET cache_key = new_value)
3. Return success to client
```

**When to use**: data that must be immediately consistent (event venues, pricing).

**Problem**: adds latency to every write. If the cache delete fails after the DB write succeeds, the cache stays stale until TTL. Requires careful error handling.

### Write-behind (async) invalidation

Queue a cache invalidation event and process it asynchronously after the write returns.

**When to use**: write latency is critical; brief post-write staleness is acceptable.

**Problem**: a small window exists between write and invalidation where stale data may be served. If the queue is slow or backed up, the window grows.

### Cache key versioning

Include a version number in the cache key. Increment the version in the database on every write. Readers construct the cache key by first fetching the current version.

```
Read flow:
  version = GET version_key   (or SELECT version FROM db)
  data = GET "event:123:v{version}"
  if miss: fetch from DB, SET "event:123:v{version}" = data

Write flow:
  BEGIN TRANSACTION
    UPDATE events SET ... WHERE id = 123
    UPDATE events SET version = version + 1 WHERE id = 123
  COMMIT
  SET "event:123:v{version+1}" = new_data
```

Old entries (`event:123:v42`) become unreachable the moment the version increments to v43 — no explicit deletion needed. Stale entries expire via their natural TTL.

**When to use**: entity-level caches (user profiles, product details) where invalidation races are a concern.

**Advantages**:
- No race conditions — a "late writer" can't overwrite new data; the DB enforces a new version number atomically
- No partial invalidation — you never delete a cache; you route around it
- Works naturally across CDN and browser caches (version is part of the URL or key)

**Disadvantages**:
- Two cache lookups per request (version key + data key)
- Stale versions accumulate; need TTLs for garbage collection
- Doesn't help with computed data (search results, feeds) where no single version number applies

### Tagged invalidation

Associate cache entries with semantic tags (e.g., `user:123:posts`). When related data changes, delete all entries matching the tag.

**When to use**: when multiple cache keys depend on the same underlying data (e.g., all cached feeds that contain a specific post).

**Problem**: requires tag tracking infrastructure; invalidation fan-out can be large.

### Deleted items cache

Instead of invalidating all feeds that contain a deleted item, maintain a small, fast-to-query cache of recently deleted/hidden item IDs. Readers filter results against this set before serving.

```
serve_feed(user):
  feed = GET cached_feed[user]
  deleted = GET deleted_items_cache   # small, fast
  return feed.filter(item => item.id not in deleted)
```

**When to use**: content moderation, privacy deletions, where full invalidation is expensive but correctness is required.

**Advantage**: defers the expensive full invalidation to a background job while immediately serving correct results.

## Choosing a strategy

| Data type | Acceptable staleness | Recommended |
|-----------|---------------------|-------------|
| Static content (images, CSS) | Days | Long TTL |
| Product catalog | Minutes | TTL or write-through |
| User profile | Seconds to minutes | Cache key versioning |
| Event venue / address | Zero | Write-through + CDN purge |
| Feeds / search results | Seconds | Deleted items cache + background invalidation |

Different data in the same system may use different strategies. Design caching strategy per-entity based on consistency requirements, not uniformly.

## CDN invalidation

CDNs cache responses across potentially hundreds of edge locations. Invalidation via CDN API works but propagation takes time — critical updates may need `Cache-Control: no-store` headers to bypass edge caching entirely, trading latency for consistency.

See also: [[Cache Stampede]], [[Scaling Reads]], [[Redis]], [[concepts/Distributed Systems/index|Distributed Systems]]
