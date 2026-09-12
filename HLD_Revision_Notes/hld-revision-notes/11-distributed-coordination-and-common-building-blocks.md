# 11 — Distributed Coordination and Common Building Blocks

> Goal: use IDs, leases, locks, leader election, and discovery with correct failure assumptions.

## 1. Coordination is expensive

Coordination establishes shared ordering or exclusive authority across unreliable nodes. It adds latency and may reduce availability during partitions. Avoid global coordination when data can be independently partitioned or operations can safely commute.

Use it for correctness-sensitive metadata such as leader ownership, configuration, shard placement, and unique allocation—not every ordinary request.

## 2. Unique ID strategies

| Strategy | Strength | Trade-off |
|---|---|---|
| DB auto-increment | Simple, ordered locally | Central dependency; reveals volume; multi-writer coordination |
| UUID v4/random | Decentralized, huge space | Larger index; random locality; not time ordered |
| Time-ordered UUID/ULID-like | Decentralized and roughly sortable | Clock/privacy/specification caveats |
| Snowflake-style | Time + worker + sequence; compact sortable ID | Worker-ID allocation, clock rollback, implementation dependency |
| Range allocation | Few coordinator calls | Gaps and range-owner recovery |

Ask whether IDs need global uniqueness, ordering, unpredictability, compactness, offline generation, and index locality. “Unique” and “strictly time ordered” are different.

## 3. Distributed locks

A lock service attempts to grant one owner exclusive access for a scope. Questions:

- Where is authority stored?
- How is ownership identified?
- What if holder pauses longer than lease?
- How is stale work rejected?
- What happens under partition?
- Is mutual exclusion actually required, or can an atomic conditional update work?

Prefer database constraints/compare-and-set when the protected data is in one transactional store.

## 4. Lease and fencing token

A lease expires, allowing progress after holder failure. But old holder may resume after expiry and continue writing.

Issue a monotonically increasing **fencing token** on each acquisition. The protected resource accepts only tokens newer than any previously observed token. Expiry controls liveness; fencing protects against stale owners.

```text
Worker A gets token 41, pauses
Lease expires; Worker B gets token 42 and writes
Worker A resumes with 41 → storage rejects it
```

## 5. Leader election

A group chooses one node to coordinate/serialize work. Safe election needs:

- terms/epochs;
- quorum authority;
- stale-leader rejection;
- state/log readiness before serving;
- re-election and client redirection.

A leader improves simplicity but can bottleneck and becomes temporarily unavailable during election. Multiple leaders without conflict rules create split brain.

## 6. Consensus intuition

Consensus protocols replicate an ordered log/value while tolerating a bounded number of failures. Majority quorums overlap, so conflicting majorities cannot independently decide under the assumed model.

Remember:

- safety: never decide conflicting values;
- liveness: eventually progress under required conditions;
- leader/term identifies current authority;
- commit requires quorum acknowledgement;
- minority partition cannot safely continue authoritative writes.

Do not claim consensus remains available after losing a majority.

## 7. Heartbeats and membership

Heartbeats detect suspected failure and measure reachability. False suspicion can occur from pause, overload, packet loss, or partition.

Membership changes affect routing, replication, and consistent hashing. They need version/epoch and careful rollout; two nodes using different membership views may make conflicting decisions.

## 8. Service discovery

Maps a logical service to healthy endpoints.

- **Client-side:** client chooses an instance from registry; efficient but pushes logic/upgrades to clients.
- **Server-side/proxy:** client calls stable endpoint; proxy discovers/routes; centralizes policy but adds hop/dependency.
- DNS is simple but has cache and TTL behavior.

Discovery must remove draining/unhealthy endpoints, handle stale caches, and authenticate registrations.

## 9. Configuration management

Configuration is production state. Good systems provide:

- schema/type validation;
- versioning and audit;
- staged rollout;
- safe defaults;
- rollback;
- access control;
- cached last-known-good value;
- separation of secrets from ordinary config.

A globally pushed bad config can cause a correlated outage faster than code deployment.

## 10. Rate limiter building block

Algorithms:

- fixed window: simple, boundary burst;
- sliding log: accurate, expensive;
- sliding-window counter: approximate compromise;
- token bucket: allows bounded bursts at refill rate;
- leaky bucket: smooth output.

Decide key (user/IP/API/tenant), limit, burst, scope, atomic update, distributed consistency, failure behavior, and response headers. A global exact limiter costs coordination; regional approximate enforcement is often enough.

## 11. Scheduler building block

For delayed/periodic jobs store:

- job ID and schedule/next run;
- payload reference;
- status, attempts, priority;
- lease owner and expiry;
- idempotency/dedup key;
- last error and timestamps.

Workers atomically claim eligible jobs, process idempotently, extend bounded leases, retry with backoff, and expose DLQ/manual replay. Exactly one active scheduler is less important than exactly-once-safe effects.

## Common traps

- Redis `SET NX` plus expiry alone does not protect against a paused stale holder without fencing.
- Clock-based IDs need clock rollback handling.
- UUID uniqueness does not imply chronological ordering.
- Heartbeats provide suspicion, not proof.
- Leader election without stale-leader fencing can still split brain.
- Service discovery does not automatically provide load balancing or retries.
- Global strict rate limiting may reduce availability and add latency.
- Consensus is not needed for every data path.

## Interview checks

1. Choose an ID scheme for public short links versus internal messages.
2. Explain why lease expiry alone is insufficient.
3. What happens to consensus writes after the majority is unavailable?
4. Client-side vs proxy service discovery?
5. Design an atomic distributed rate-limit check.

## 60-second recall

- Coordinate only when shared order/authority is required.
- ID choice balances uniqueness, order, locality, secrecy, and coordination.
- Lease gives liveness; fencing rejects stale actors.
- Leader needs epoch/quorum and stale-leader protection.
- Heartbeats suspect failure; they do not prove it.
- Configuration and discovery require versioning, security, and stale-state handling.

## Sources for deeper revision

- [Google SRE: Distributed Consensus for Reliability](https://sre.google/sre-book/managing-critical-state/)
- [Redis: Distributed Locks](https://redis.io/docs/latest/develop/clients/patterns/distributed-locks/)
- [AWS Prescriptive Guidance: Distributed Locking](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/distributed-lock.html)

