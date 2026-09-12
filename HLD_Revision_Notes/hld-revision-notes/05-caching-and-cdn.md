# 05 — Caching and CDN

> Goal: use caches for a measured read/latency problem while explaining freshness, invalidation, stampede, and failure behavior.

## 1. Cache mental model

A cache stores a reusable copy or computed result closer to the consumer. It can reduce latency, origin load, and cost—but introduces another version of data.

Ask:

1. What is the source of truth?
2. What is the cache key/value?
3. How stale may it be?
4. How is it filled, updated, invalidated, and evicted?
5. What happens on miss or cache failure?

## 2. Cache locations

- browser/device cache;
- DNS/proxy cache;
- CDN/edge cache;
- service-local in-memory cache;
- distributed cache;
- database buffer/result cache.

Local caches are very fast but duplicated and hard to invalidate globally. Distributed caches provide shared capacity/state but add a network dependency.

## 3. Read patterns

### Cache-aside (lazy loading)

1. Read cache.
2. On miss, read source.
3. Store result with TTL.
4. Return.

Simple and common. First request misses; application owns consistency logic.

### Read-through

Application asks the cache, which loads from source on miss. Centralizes loading but couples cache implementation to data access.

## 4. Write patterns

| Pattern | Behavior | Trade-off |
|---|---|---|
| Write-through | Update cache and source synchronously | Fresher cache; higher write latency/two-system failure handling |
| Write-around | Write source, invalidate/bypass cache | Avoids caching cold writes; next read misses |
| Write-back/behind | Acknowledge cache, persist later | Fast writes; durability, ordering, recovery complexity |

For cache-aside writes, commonly update source first then invalidate cache. Race windows still exist; TTL, versioning, event-driven invalidation, or serialization may be needed for stronger guarantees.

## 5. Expiration and eviction

- **TTL:** bounds age but creates stale window and synchronized expiry risk.
- **LRU/LFU approximations:** evict based on recency/frequency.
- **Size policies:** protect cache from unbounded values.

TTL is not correctness if stale data can cause harm. Authorization, account balance, inventory, and pricing may require validation against an authoritative path.

## 6. Cache stampede

Many clients miss the same popular key and simultaneously recompute/fetch it.

Mitigations:

- request coalescing/single flight;
- distributed lease with careful timeout;
- TTL jitter;
- refresh-ahead;
- stale-while-revalidate;
- bounded origin concurrency;
- pre-warming selected hot keys.

Locks must expire, and a slow holder can still create stale writers; version/fencing techniques may be needed.

## 7. Penetration and negative caching

Requests for nonexistent keys bypass cache repeatedly. Cache a “not found” result briefly, validate malicious inputs, use rate limits, or probabilistic membership filters when justified.

Negative TTL should be short enough that newly created data becomes visible.

## 8. Hot keys and distribution

A single popular key can overload one cache shard even when total cluster capacity is adequate.

Options:

- local near-cache or CDN;
- replicate the hot value;
- split/partition the logical key when possible;
- request coalescing;
- cache a slightly stale result;
- admission control.

Consistent hashing limits remapping as nodes change. Virtual nodes improve distribution; skew still needs detection.

## 9. Cache consistency risks

- stale reads after source update;
- invalidation before a failed source write;
- delayed/out-of-order invalidation events;
- repopulation race with old value;
- deleted data remaining cached;
- cache key missing tenant, locale, permissions, or version;
- different TTLs creating internally inconsistent views.

For privacy/authorization, include security context in the key only when safe, or avoid caching personalized decisions.

## 10. Cache failure

A cache outage can send all traffic to the source: a **cache-miss storm**. Plan:

- source capacity protection;
- gradual warm-up;
- request limiting/coalescing;
- circuit breaker/degraded results;
- local fallback where safe.

Treat cache as disposable only if the source can survive its loss or the service can shed/degrade.

## 11. CDN essentials

A CDN stores content at geographically distributed edge locations.

Good for static assets, images/video segments, downloads, and cacheable API responses. It improves latency and reduces origin egress/load.

Key concepts:

- origin and edge/point of presence;
- cache-control and TTL;
- cache key (host, path, query, selected headers);
- purge/invalidation;
- signed URLs/cookies;
- range requests;
- origin shield;
- stale serving during origin failure.

Do not put user-specific/private content in a shared cache without correct keying and access control.

## 12. When not to cache

- low reuse or uniformly random access;
- values change faster than useful freshness;
- correctness cannot tolerate stale data;
- source query is already cheap;
- invalidation/operations cost exceeds benefit;
- sensitive data isolation is uncertain.

## Common traps

- Cache is not the source of truth unless deliberately designed as durable storage.
- “Use Redis” does not specify a caching policy.
- TTL does not prevent all stale-read races.
- Deleting from cache before committing the database can expose older values again.
- 90% hit rate may still overload origin at peak.
- CDN does not automatically cache everything or make dynamic APIs safe.
- A Bloom filter can have false positives, not false negatives under standard no-delete use.

## Interview checks

1. Design cache-aside for product details and explain update races.
2. How does TTL jitter reduce stampede?
3. What makes a good cache key for a multi-tenant API?
4. What happens to the database when the cache cluster restarts?
5. When would stale-while-revalidate be unacceptable?

## 60-second recall

- Cache needs source, key, freshness, fill, invalidation, and failure policy.
- Cache-aside is simple; write-behind adds durability risk.
- Stampede: coalesce, jitter, refresh, stale serve, protect origin.
- Hot key is a skew problem, not total-capacity problem.
- Cache outage can become database outage.
- CDN is a geographically distributed cache with origin/security concerns.

## Sources for deeper revision

- [Amazon Builders’ Library: Caching Challenges and Strategies](https://aws.amazon.com/builders-library/caching-challenges-and-strategies/)
- [HTTP Caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)
- [Redis: Client-side caching](https://redis.io/docs/latest/develop/clients/client-side-caching/)

