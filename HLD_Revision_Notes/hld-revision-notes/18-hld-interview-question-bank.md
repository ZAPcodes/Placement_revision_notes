# 18 — HLD Interview Question Bank

> Active recall: answer aloud before reading. Aim for 30–60 seconds for concepts and 3–5 minutes for design prompts. **Trap** marks common elimination points.

## A. Interview approach

### 1. What should you clarify first?

Core user actions, scope exclusions, scale, read/write pattern, latency, availability, durability, consistency, geography, retention, and security. Ask only questions that can change the design.

### 2. Functional vs non-functional requirement?

Functional states what the system does; non-functional states qualities/constraints such as latency, throughput, availability, durability, consistency, cost, security, and compliance.

### 3. Why define a source of truth?

It identifies authoritative ownership. Caches/indexes/read models can then be treated as rebuildable derived state with explicit freshness and reconciliation.

### 4. When should you estimate capacity?

When scale affects storage, partitioning, bandwidth, persistent connections, fan-out, or component choice. Skip detailed arithmetic if it does not change the architecture.

### 5. What makes a strong trade-off explanation?

Connect requirement → decision → benefit → cost/failure mode → mitigation. Avoid technology adjectives such as “fast” without workload evidence.

### 6. What should a high-level diagram show?

System boundary, core components/stores, ownership, sync/async links, and primary read/write flow. It should not contain unexplained decorative boxes.

### 7. How do you recover after a wrong assumption?

State what changed, identify affected requirements/components, revise only the necessary paths, and explain new trade-offs. Adaptation is part of evaluation.

### 8. How do you close the interview?

Summarize requirements, key decisions, consistency/failure behavior, bottleneck/hot key, trade-offs, and next evolution. Mention what was intentionally out of scope.

## B. Estimation, APIs, and communication

### 9. QPS vs concurrency?

QPS is arrival/completion rate; concurrency is simultaneous in-flight work. By Little’s-law intuition, concurrency grows with arrival rate × time. Persistent sockets may have low QPS but huge concurrency.

### 10. Why design for peak rather than average?

Time-zone, launches, fan-out, retries, and events create bursts. Average-sized capacity can saturate, increase latency, trigger retries, and collapse.

### 11. Availability vs durability?

Availability is ability to serve now; durability is survival of acknowledged data. A temporarily unavailable store can retain all data; an available cache can lose entries.

### 12. Why use p99 latency?

Averages hide tail behavior. A small fraction of slow dependencies can dominate multi-call request experience and affect many users at scale.

### 13. Offset vs cursor pagination?

Offset is simple but large offsets cost work and concurrent mutations cause skips/duplicates. Cursor/keyset uses stable ordering and scales better but complicates random navigation.

### 14. When choose WebSocket?

For bidirectional low-latency ongoing interaction such as chat/collaboration. It adds connection routing, heartbeat, reconnection, backpressure, and gateway capacity.

### 15. SSE vs WebSocket?

SSE is server-to-client over HTTP and simpler for feeds/notifications; WebSocket is bidirectional. Choose from communication direction and operational support.

### 16. Sync vs async call?

Sync gives immediate result but couples latency/availability. Async buffers and decouples but adds delayed state, duplicates, ordering, backlog, and observability.

### 17. What does a timeout mean?

The caller did not receive a result in time; the server may have failed, still be running, or already succeeded. Retrying requires idempotency or status lookup.

### 18. How do idempotency keys work?

Client sends stable scoped key; server atomically records key, payload identity, state, and result. Concurrent/repeated matching requests return the same effect/result; mismatched reuse is rejected.

## C. Scaling, load balancing, and caching

### 19. Vertical vs horizontal scaling?

Vertical uses a larger machine and is simple but bounded. Horizontal adds nodes and redundancy but requires distribution, routing, state placement, and coordination.

### 20. What makes an application server stateless?

No required session/durable state exists only in that instance; any healthy instance can serve the next request using shared/authoritative stores.

### 21. L4 vs L7 load balancer?

L4 routes connections using network/transport metadata; L7 understands HTTP/application content and can route by host/path/header at more processing/coupling cost.

### 22. Sticky-session drawbacks?

Uneven load, fragile failover, scaling/rebalance difficulty, and hidden instance-local state. Shared session state is usually more replaceable.

### 23. Health vs readiness?

Liveness asks whether process should restart; readiness asks whether it should receive new traffic. Shared dependency failure should not necessarily restart every instance.

### 24. Why can autoscaling fail during overload?

Metrics/startup lag, downstream bottleneck, capacity quota, retry amplification, cold caches, or new instances adding load to an already failing dependency.

### 25. Cache-aside flow?

Read cache; on miss read source and populate with TTL. On write, commonly update source then invalidate. Simple, but races/staleness/stampede require handling.

### 26. Write-through vs write-back cache?

Write-through synchronously updates source/cache for fresher reads but higher latency. Write-back acknowledges cache then persists later, improving latency but adding durability/order/recovery risk.

### 27. What is cache stampede?

Many requests miss/expire the same hot key and overload origin. Use request coalescing, TTL jitter, refresh-ahead, stale-while-revalidate, and origin concurrency limits.

### 28. What is a hot key?

One key receives disproportionate traffic and overloads one shard. Replicate/cache it, coalesce requests, split/bucket where possible, or isolate/rate-limit.

### 29. **Trap:** Is cache failure harmless because cache is disposable?

No. Loss can redirect the entire miss load to the source and cause a database outage. Protect origin and warm gradually/degrade.

### 30. When should you not cache?

Low reuse, very high mutation, strict freshness, cheap source, sensitive keying uncertainty, or invalidation/operations cost exceeding benefit.

## D. Storage and data modeling

### 31. How do you choose a database?

Start with access patterns, invariants/transactions, scale, ordering/freshness, schema shape, lifecycle, and operational expertise—not “SQL vs NoSQL” branding.

### 32. Why is relational storage a strong default?

Transactions, constraints, joins, flexible queries, indexing, and mature tools. One capable relational cluster often serves far more scale than interview candidates assume.

### 33. When use a document store?

When bounded owned aggregates with evolving fields are commonly read/written together. Avoid unbounded embedding and unclear cross-document invariants.

### 34. Why use a search engine beside a database?

Inverted indexing, tokenization, relevance, fuzzy search, and faceting. Treat it as derived: define indexing lag, replay/rebuild, and deletion reconciliation.

### 35. Why use object storage for files?

It provides scalable durable blob storage, multipart transfer, lifecycle tiers, and CDN integration. Keep metadata/ownership/state in a database and upload via signed URLs.

### 36. What is polyglot persistence’s cost?

Each store adds expertise, deployment, monitoring, backup, security, data synchronization, migration, and incident surface. Add only for material capability.

### 37. How should a derived store be maintained?

Use outbox/change log, idempotent versioned updates, lag monitoring, deletion propagation, reconciliation, and full rebuild path.

### 38. Soft delete vs hard delete?

Soft delete preserves recoverability/audit but still requires access filtering and eventual retention cleanup. Hard delete removes data, but backups/derived copies and legal requirements need policy.

## E. Replication, partitioning, and consistency

### 39. Replication vs sharding?

Replication copies data for redundancy/read locality; sharding divides data for storage/write scale. A sharded system usually replicates each shard.

### 40. Leader–follower benefits and risks?

Simple write authority and follower reads/failover; risks include lag, stale reads, lost async writes on failover, and leader bottleneck.

### 41. When consider multi-leader?

Multi-region/offline writes with an explicit conflict strategy. It adds ordering and reconciliation complexity, so not a default.

### 42. What does `R + W > N` imply?

Read/write quorums overlap under ideal assumptions. It does not alone guarantee linearizability due to stale/conflicting versions, sloppy quorum, failures, and repair semantics.

### 43. Read-your-writes?

A client sees its own completed write. Achieve by leader read, sticky/session routing, version token, or waiting until a replica catches up.

### 44. What makes a good shard key?

Even load/storage, present in common queries, keeps atomic/local data together, stable, bounded, and routable. These goals must be balanced.

### 45. Hash vs range sharding?

Hash spreads diverse keys but hurts ranges. Range preserves order/range scans but sequential or skewed keys can hot-spot.

### 46. Why can `user_id` be a poor key?

Celebrity/large-tenant traffic and data cause skew; operations involving many users may scatter; one user’s partition can become unbounded.

### 47. Consistent hashing benefit?

Adding/removing nodes remaps only part of keyspace rather than almost all modulo assignments. Virtual nodes improve balance; hot keys and movement remain.

### 48. Why are scatter-gather queries risky?

Cost grows with shard count; slowest shard dominates tail latency; partial failure/merging/limits are complex.

### 49. Local vs global secondary index?

Local index is updated with its shard but queries without partition key scatter. Global index routes directly but has its own consistency, sharding, and failure problems.

### 50. How do you reshard safely?

Plan ownership, copy history, capture concurrent changes, validate, switch routing, drain old owner, retain rollback, then clean up. Avoid unsafe dual writes.

## F. CAP and coordination

### 51. Explain CAP without “pick two.”

During network partition, a replicated system cannot guarantee both linearizable consistency and every nonfailed node returning a non-error response. Choose behavior per operation.

### 52. What does PACELC add?

Even without partition, systems trade lower latency against stronger consistency, especially across regions.

### 53. Why is timeout not proof of failure?

Node may be slow, paused, unreachable only from one observer, or completed without acknowledgement. This uncertainty drives idempotency, leases, and consensus.

### 54. What is split brain?

Multiple partitions believe they have write authority. Prevent with quorum/epoch/term/fencing or explicit ownership, not simple health checks.

### 55. Lease vs lock?

A lease expires to preserve liveness. Expiry can let old holder continue after pause, so protected resource should enforce a newer fencing token.

### 56. Why fencing tokens?

Monotonically increasing token lets storage reject operations from a stale lease holder after a newer owner begins.

### 57. What does consensus provide?

Agreement on ordered value/log despite bounded failures. It preserves safety through quorum/term rules but generally loses progress without communicating majority.

### 58. UUID vs Snowflake-style ID?

Random UUID is decentralized/unpredictable but large and unordered. Snowflake-style is compact/roughly time ordered but needs worker allocation and clock rollback handling.

### 59. Heartbeat limitation?

It detects lack of response, which can mean failure, pause, overload, or partition. It creates suspicion, not certainty or authority revocation.

### 60. When avoid distributed coordination?

When data can be independently partitioned, operations commute/merge, approximate results suffice, or a single transactional owner can enforce the invariant.

## G. Messaging and correctness

### 61. Queue vs pub/sub vs stream?

Queue distributes work among competing workers; pub/sub delivers to independent subscriptions; stream retains partitioned ordered log with consumer offsets/replay.

### 62. At-most-once vs at-least-once?

At-most-once may lose but does not broker-redeliver; at-least-once retries but duplicates. Exactly-once effect needs an explicit atomic/dedup boundary.

### 63. How do you make a consumer idempotent?

Atomically record stable event ID with local state change, or make operation naturally idempotent/versioned. For external effects, pass downstream idempotency key and reconcile.

### 64. What determines ordering?

Broker partition/key and producer semantics. Preserve only required scope such as conversation/order; global ordering limits parallelism and availability.

### 65. Consumer-group parallelism limit?

Normally active partition count: one partition is handled by at most one member of a group at a time. More consumers than partitions sit idle.

### 66. Why track oldest-message age?

Queue depth may stay stable while processing falls behind or message sizes vary. Oldest age reflects user-visible delay/backlog health.

### 67. DLQ purpose and limitation?

Isolates repeatedly/permanently failing messages with context. It needs alerting, diagnosis, retention, and safe replay; otherwise it is silent data loss.

### 68. Transactional outbox?

Write business state and outbox record in one DB transaction; relay publishes with retry. It removes DB/publish dual-write gap but consumers still handle duplicates.

### 69. 2PC vs saga?

2PC coordinates atomic commit across capable participants but can block and hold resources. Saga uses local transactions plus semantic compensation and visible intermediate states.

### 70. Reservation pattern?

Atomically hold scarce resource with expiry, perform slower workflow, confirm or release, and reconcile stuck/late outcomes. Final operation validates authoritative state.

### 71. Why is saga compensation not rollback?

It is a new business action; external observers may have seen original effect, prices/state may change, and some actions such as email cannot be undone.

### 72. Define exactly-once payment effect.

One business charge for one scoped idempotency key, enforced by durable request identity, provider key/status lookup, state machine, ledger, retries, and reconciliation—not one HTTP attempt.

## H. Reliability and operations

### 73. Deadline vs timeout?

Deadline is end-to-end latest useful completion; timeout bounds one operation. Propagate remaining deadline so downstream work/retries do not outlive caller need.

### 74. Why exponential backoff plus jitter?

Backoff reduces repeated pressure; jitter prevents synchronized clients from retrying together and recreating a burst.

### 75. Retry amplification?

Retries at multiple layers multiply downstream attempts and can turn slowness into overload. Retry at one controlled layer with shared budget.

### 76. Circuit breaker?

Fails fast after sustained dependency failure, then probes recovery. Protects resources but requires careful thresholds and fallback.

### 77. Bulkhead?

Separates pools/queues/cells/tenant capacity so one failure or workload cannot exhaust all resources. Trades utilization for isolation.

### 78. Load shedding vs backpressure?

Shedding rejects/drops work to preserve service; backpressure asks upstream to slow. Systems often use both with bounded queues.

### 79. Why are unbounded queues dangerous?

They hide overload as growing latency/memory/storage and make recovery impossible while arrivals exceed service rate.

### 80. Graceful degradation example?

Serve product page without recommendations, chronological feed without ranking, delayed processing after durable upload, or read-only mode when writes are unsafe.

### 81. RPO vs RTO?

RPO is tolerated data-loss time window; RTO is tolerated restoration duration. They determine backup/replication/DR architecture and cost.

### 82. Why replication is not backup?

Replicas copy deletions/corruption and may lack historical retention. Backups/log archive provide earlier recovery points; restore must be tested.

### 83. What makes multi-region failover real?

Data readiness, routing, spare capacity, dependencies, identity/secrets/config, failure detection, runbook/automation, tested failover and failback.

### 84. Four golden signals?

Latency, traffic, errors, and saturation. Add domain correctness/freshness signals.

### 85. SLI vs SLO vs SLA?

SLI is measurement; SLO is target; SLA is external commitment. Error budget is tolerated unreliability relative to SLO.

### 86. Metrics vs logs vs traces?

Metrics summarize/alert, logs capture contextual events, traces show distributed request path/timing. Correlation IDs connect them.

### 87. Canary deployment requirement?

Representative small traffic, comparable metrics/SLO gates, safe duration, automated stop/rollback, and compatible schema/protocols.

### 88. Expand-and-contract migration?

Add compatible schema, deploy dual-compatible code, backfill, switch, validate, then remove old schema after every consumer migrates.

## I. Security and multi-tenancy

### 89. Authentication vs authorization?

Authentication verifies identity; authorization decides whether identity can perform action on resource. Resource owner service should enforce authorization.

### 90. Session vs JWT?

Server session allows immediate revocation but adds state lookup. JWT enables local verification but has revocation, claim-staleness, key-rotation, and size risks.

### 91. OAuth vs OpenID Connect?

OAuth is delegated authorization; OIDC adds identity/authentication claims over OAuth flows.

### 92. Fail-open vs fail-closed rate/security check?

Choose from risk: noncritical analytics quota may fail open; authentication/payment/fraud protection usually fails closed or degrades through a safe path.

### 93. Signed upload URL risks?

Scope method/object/size/content type, short expiry, random key, authorization before issue, post-upload verification/scanning, and prevent overwrite/path confusion.

### 94. Multi-tenant cache key?

Include tenant and every dimension affecting response/authorization; otherwise data can leak. Avoid caching personalized authorization outcomes unless rigorously keyed.

### 95. Noisy-neighbor protection?

Per-tenant quotas/concurrency, isolated pools/partitions for heavy tenants, fair scheduling, cost attribution, and tenant-aware saturation alerts.

### 96. Why avoid sensitive metric labels/logs?

Privacy/security leakage and high-cardinality cost. Redact/tokenize, restrict access/retention, and use sampled trace/log correlation rather than user IDs in metrics.

## J. Design mini-prompts

### 97. URL shortener—first three decisions?

Code generation/unpredictability, redirect durability/latency with cache, and custom-alias uniqueness/expiry. Move analytics off redirect path.

### 98. Chat—where is ordering required?

Normally per conversation, not globally. Partition/sequence by conversation; clients dedupe and fetch gaps after reconnect.

### 99. Feed—push or pull?

Push for ordinary accounts gives fast reads; pull for celebrities avoids enormous fan-out writes; hybrid merges both at read.

### 100. Ticket booking—how prevent oversell?

Authoritative atomic conditional seat hold/reservation with expiry; payment outside DB transaction; idempotent confirmation/release and reconciliation.

### 101. Payment timeout—what next?

Treat as unknown. Query provider by stable merchant/idempotency reference, accept webhook idempotently, and reconcile—never blindly charge again.

### 102. File upload—why direct to object storage?

Avoid proxying large bytes through application fleet, improve multipart/resume and scalability. API owns signed authorization and metadata state.

### 103. Search autocomplete—serving structure?

Precomputed prefix top-K or trie/FST-like in-memory index, built from filtered aggregates, versioned and cached; guard short-prefix hotspots/privacy.

### 104. Ride-hailing—location vs trip state?

Location is ephemeral high-write geospatial data with bounded staleness; trip assignment is durable and requires atomic state transition.

### 105. Notification provider failure?

Persist state, retry transient with backoff, fail over carefully using stable ID, process callbacks idempotently, isolate critical channels, expose delayed/failed status.

### 106. Job completes then worker dies before ack?

Message/job is redelivered after lease. Idempotent handler/dedup prevents duplicate business effect; record durable outcome before ack.

### 107. Viral video?

CDN/edge serves segments; origin shield and pre-warm protect object origin; autoscale metadata path; popular content should not hit app/database per segment.

### 108. Telemetry system overloaded?

Bound buffers, batch/compress, sample/drop low-priority data, enforce tenant quotas, preserve critical signals, and keep telemetry off request critical path.

## K. Resume/project cross-questions

### 109. “Scalable architecture”—what evidence?

Workload/dataset, baseline p95/p99 or throughput, load-test method, bottleneck, exact change, resource/cost trade-off, failure test, and measured result.

### 110. Why Redis?

Name exact capability: cache with TTL, atomic limiter, ephemeral session, or queue primitive. Explain source of truth, eviction/failure behavior, hot keys, and persistence requirement.

### 111. Why MongoDB?

Defend aggregate/document access, schema evolution, and indexing. Discuss embedding bounds, transactions/relationships, shard key, and why relational alternative was less suitable.

### 112. Why Cassandra?

Defend query-driven partitioned model, write throughput, availability, and data volume. Explain partition key, consistency level, hot partitions, tombstones, and denormalized tables.

### 113. Why microservices?

Use independent ownership, scaling, deployment, failure isolation, or domain boundaries—not fashion. State costs: network failure, observability, data consistency, deployment and operational complexity.

### 114. What would fail first at 10× traffic?

Give a measured/estimated bottleneck—DB writes, hot partition, cache origin, queue consumers, external quota, socket gateways—and a specific next change plus its trade-off.

## Final trap list

1. CAP is not “pick any two” during normal operation.
2. Replication is not sharding or backup.
3. Cache failure can overload the database.
4. Timeout does not prove failure.
5. At-least-once means duplicates.
6. Outbox does not remove consumer deduplication.
7. Lease expiry needs stale-owner protection/fencing.
8. Consistent hashing does not eliminate hot keys.
9. Autoscaling does not scale a downstream bottleneck.
10. Queue absorbs bursts, not permanent overload.
11. WebSocket is not durable messaging.
12. Multi-region active-active needs conflict semantics.
13. JWT is not automatically safer than sessions.
14. A design without failure/operations/security is incomplete.

