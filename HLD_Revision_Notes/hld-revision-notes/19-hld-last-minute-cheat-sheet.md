# 19 — HLD Last-Minute Cheat Sheet

> Read in 20–25 minutes before an interview. Use the numbered notes when any line feels vague.

## 1. The 45-minute framework

```text
1. Clarify core features and exclusions
2. Establish scale and non-functional requirements
3. Define APIs and primary data/access patterns
4. Draw the simplest correct end-to-end design
5. Walk one write and one read
6. Deep-dive bottleneck, correctness, and failures
7. Summarize decisions, trade-offs, and next evolution
```

For every choice say:

```text
Requirement → Decision → Benefit → Cost/failure → Mitigation
```

Always name:

- source of truth;
- acknowledgement/durability boundary;
- partition key and hot key;
- consistency requirement per operation;
- duplicate/timeout/retry behavior;
- degraded mode and observability.

## 2. Requirement questions

### Functional

- Who are the actors?
- What are the top 2–4 flows?
- What is out of scope?
- What must happen synchronously?

### Non-functional

- Users, QPS, peak factor, read/write ratio?
- Payload/object size, growth, retention?
- Latency and availability target?
- Durability and consistency per operation?
- Geography, privacy, abuse, compliance, cost?

Do not ask questions whose answers will not affect the design.

## 3. Estimation shortcuts

```text
average QPS ≈ daily operations / 86,400
peak QPS ≈ average × peak factor
storage ≈ object count × bytes × retention/copies
bandwidth ≈ QPS × bytes
concurrency ≈ arrival rate × operation duration
```

- Round aggressively; seek order of magnitude.
- Include indexes, replicas, logs, versions, derived media.
- Estimate internal fan-out, not only public requests.
- Persistent connections require concurrency capacity.
- A number must justify or reject a design decision.

## 4. API quick decisions

| Need | Choice |
|---|---|
| Public resource API | REST/HTTP |
| Typed low-latency internal call/stream | RPC/gRPC |
| Client-selected graph-shaped reads | GraphQL, with query-cost controls |
| Server → browser updates | SSE |
| Bidirectional real-time | WebSocket |
| Server-to-server callback | Signed webhook |
| Deferred work | `202` + job/status or async notification |

- Cursor/keyset beats large offset for changing large datasets.
- Stable order needs tie-breaker such as `(created_at, id)`.
- Timeout = unknown outcome. Retry requires idempotency/status query.
- Idempotency record needs scoped key, payload identity, state/result, expiry.
- WebSocket requires gateway routing, heartbeat, reconnect, dedup, backpressure.

## 5. Scaling and routing

| Concept | Recall |
|---|---|
| Vertical | Bigger node; simple but bounded |
| Horizontal | More nodes; routing/state/coordination complexity |
| Stateless tier | No required state only on one instance |
| L4 LB | Routes connection using IP/port |
| L7 LB | Routes application request using host/path/header |
| Readiness | Accept new traffic? |
| Liveness | Restart process? |
| Draining | Stop new work; finish in-flight work |

- Round robin assumes similar work/nodes.
- Least-connections can use stale/incomplete load signal.
- Sticky sessions hurt balance/failover; shared state is preferable.
- DNS failover has TTL/client-cache delay.
- Autoscale using actual saturation signal and startup/headroom.
- More app servers can overload a fixed database.

## 6. Cache checklist

```text
Source → Key → Value → Fill → Freshness → Invalidate → Evict → Failure
```

| Pattern | Meaning |
|---|---|
| Cache-aside | App loads on miss |
| Read-through | Cache loads on miss |
| Write-through | Source/cache updated synchronously |
| Write-around | Source write + cache bypass/invalidate |
| Write-back | Cache acknowledges; source later—durability risk |

- Stampede: single-flight, jitter, refresh-ahead, stale-while-revalidate.
- Penetration: negative cache, validation, filter/rate limit.
- Hot key: local/edge replicas, coalescing, split where possible.
- Cache outage can become source outage; protect and warm gradually.
- TTL bounds freshness but does not eliminate races.
- Include tenant/locale/permission/version dimensions in safe cache keys.

## 7. Storage selection

| Need | Candidate |
|---|---|
| Transactions, constraints, joins | Relational DB |
| Known-key simple access | Key-value |
| Bounded owned aggregate | Document |
| Huge query-shaped partitioned writes | Wide-column |
| Full-text/ranking/facets | Search index |
| Large immutable bytes | Object storage |
| Time-window metrics | Time-series |
| Deep relationship traversal | Graph |

- Start from queries, invariants, scale, lifecycle, and operations.
- Search/cache/warehouse are normally derived stores.
- Every derived store needs propagation, lag, versioning, rebuild, deletion, repair.
- Blob bytes in object store; metadata/authorization/status in database.
- Every extra database adds synchronization and operational cost.

## 8. Replication and consistency

| Model | Strength | Risk |
|---|---|---|
| Leader–follower | Simple write order, read replicas | Lag, leader bottleneck/failover loss |
| Multi-leader | Local/multiple-region writes | Conflict and ordering complexity |
| Leaderless | Availability and quorum access | Versions, conflicts, repair complexity |

- Sync replication: fresher/durable before ack, higher latency/lower availability.
- Async: lower write latency, lag and possible failover loss.
- Read-your-writes via leader, sticky/session route, version token, or replica wait.
- Eventual consistency must include convergence/conflict strategy.
- `R + W > N` overlap is not a universal linearizability proof.
- Replication copies data; it does not split dataset or replace backup.

## 9. Sharding

Good shard key:

```text
even load + query locality + atomic locality + stable + bounded + routable
```

| Strategy | Strength | Weakness |
|---|---|---|
| Hash | Broad key distribution | Poor ranges |
| Range | Range/order locality | Sequential/skew hotspot |
| Directory | Flexible placement | Metadata dependency |
| Tenant/geo | Isolation/locality | Large-tenant/region skew |

- Hash distributes keys, not necessarily traffic.
- Salting spreads hot writes but makes reads merge buckets.
- Cross-shard joins, transactions, uniqueness, ordering, secondary indexes cost more.
- Consistent hashing limits remapping; virtual nodes improve balance; hot keys remain.
- Reshard: copy + capture changes + validate + cutover + drain + rollback window.

## 10. CAP, PACELC, and coordination

- During partition: cannot guarantee both linearizable consistency and every nonfailed node’s successful response.
- Choose behavior per operation, not one permanent system label.
- PACELC: without partition, latency still trades against consistency.
- Timeout/heartbeat gives suspicion, not proof of failure.
- Split brain needs quorum/epoch/fencing, not only health checks.
- Consensus provides ordered agreement; majority loss normally stops safe writes.
- Lease provides expiry/liveness; fencing token rejects stale holders.
- Avoid coordination when operations can partition or commute safely.

## 11. Messaging

| Model | Recall |
|---|---|
| Work queue | Competing workers; one handles job |
| Pub/sub | Each subscription gets event |
| Stream/log | Retained partition order; offset/replay |

- At-most-once may lose; at-least-once duplicates.
- Idempotent consumer records event ID atomically with effect.
- Preserve order only within required key, often one partition.
- Consumer parallelism is bounded by partition count.
- Ack after durable effect; crash before ack causes redelivery.
- Retry transient failures with backoff/jitter; poison → monitored DLQ.
- Track oldest-message age, arrival/completion/retry rates—not depth alone.
- Queue absorbs temporary burst, not permanent overload.
- Outbox fixes DB + publish gap; consumer still deduplicates.

## 12. Correctness workflows

- State the invariant and atomic boundary.
- Prefer local DB transaction, unique constraint, conditional update.
- Optimistic version check for low conflict; lock for high conflict/expensive retry.
- Dual write: colocate, outbox/CDC, 2PC, saga, or reconciliation.
- 2PC: atomic participants but blocking/availability/lock cost.
- Saga: local transactions + semantic compensation + visible states.
- Reservation: hold with expiry → external work → confirm/release → reconcile.
- Exactly-once effect = identity + atomic recorded effect + safe retry + repair.
- Payment timeout is unknown; query provider with stable reference.
- Ledger is immutable balanced history; refund is a new entry.

## 13. Resilience toolbox

| Tool | Purpose |
|---|---|
| Timeout/deadline | Bound waiting/work |
| Retry + backoff/jitter | Recover transient failure without synchronization |
| Circuit breaker | Fail fast while dependency is unhealthy |
| Bulkhead | Isolate resource/blast radius |
| Backpressure | Tell upstream to slow |
| Load shedding | Reject/drop lower-priority work |
| Graceful degradation | Preserve useful core behavior |

- Avoid retries at every layer; use one bounded retry budget.
- Unbounded queues turn overload into extreme latency.
- Redundancy must cross actual failure domain and retain spare capacity.
- Autoscaling is delayed and does not fix downstream limits.
- RPO = tolerable data loss; RTO = tolerable restoration time.
- Failover requires state, routing, capacity, config/secrets, and failback.

## 14. Observability and deployment

- Golden signals: latency, traffic, errors, saturation.
- Metrics summarize/alert; logs explain events; traces follow request.
- Add business correctness/freshness metrics.
- SLI = measurement; SLO = target; SLA = external promise; error budget = allowed unreliability.
- Alert on user impact/burn rate, oldest queue age, saturation, lag/stalls.
- Canary requires representative traffic, comparison metrics, gates, rollback.
- Schema change: expand → compatible code → backfill → switch → validate → contract.
- Liveness is not deep dependency health; avoid restart storms.

## 15. Security and tenancy

- Authentication = identity; authorization = permission on action/resource.
- Gateway can do coarse auth; resource-owning service must enforce authorization.
- Session: easy revocation, server lookup. JWT: local verification, revocation/claim/key risks.
- OAuth = delegated authorization; OIDC adds authentication/identity.
- TLS, least privilege, secret manager, key rotation, audit.
- Rate limit by correct subject and request cost; define fail-open/closed.
- Multi-tenant: tenant in auth, DB/index/cache keys, quotas, logs, deletion.
- Signed URLs: narrow object/method/size, short expiry, post-upload verify/scan.

## 16. Case-study anchor table

| System | Dominant decision | Critical trap |
|---|---|---|
| URL shortener | ID/code and redirect cache | Enumeration/custom uniqueness |
| Rate limiter | Algorithm and enforcement scope | Global exact coordination/failure mode |
| Notification | Queue, provider adapter, retry | Accepted ≠ delivered; duplicates |
| Job scheduler | Lease + idempotent worker | Complete-before-ack redelivery |
| Feed | Push, pull, or hybrid fan-out | Celebrity and privacy deletion |
| Chat | Per-conversation order + reconnect | WebSocket is not persistence |
| Autocomplete | Precomputed prefix top-K | Hot short prefixes/privacy |
| File storage | Direct blob transfer + metadata state | Orphans/version/authorization |
| Video | Transcode pipeline + CDN | Egress/origin and takedown |
| Ride-hailing | Geo index + atomic trip assignment | Hot cell/double assignment |
| Ticketing | Expiring authoritative reservation | Cache cannot prevent oversell |
| Payment | Idempotency + state machine + ledger | Unknown provider outcome |
| Telemetry | Durable ingest + hot/cold stores | Cardinality/noisy tenant |

## 17. Ten statements that outperform memorized answers

1. “I’ll keep this in one relational database until write/storage evidence requires sharding.”
2. “The cache is derived; here is its invalidation and source-protection behavior.”
3. “The timeout leaves the outcome unknown, so retries use the same idempotency identity.”
4. “Ordering is required per conversation, not globally, so I partition by conversation.”
5. “This queue absorbs bursts; admission control handles sustained overload.”
6. “Async replication may lose acknowledged writes during failover; this path requires/does not require synchronous acknowledgement.”
7. “The hot-key case is a celebrity tenant, even if hashing distributes ordinary keys.”
8. “The outbox removes one dual-write gap, but delivery remains repeatable and consumers deduplicate.”
9. “A lease can expire while the old worker runs, so the resource checks a fencing token.”
10. “My degraded mode preserves the core invariant while dropping nonessential work.”

## Final two-minute checklist

Before finishing your design, ask yourself:

- Did I prioritize requirements?
- Did I define source of truth and data access patterns?
- Did I narrate read and write paths?
- Did I justify every cache, queue, replica, and shard?
- Did I handle duplicate, timeout, and partial failure?
- Did I identify hot key and bottleneck?
- Did I state consistency per critical operation?
- Did I cover security, observability, deployment, and recovery briefly?
- Did I clearly state one trade-off and next evolution?

