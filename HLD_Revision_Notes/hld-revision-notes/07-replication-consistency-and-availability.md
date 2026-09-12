# 07 — Replication, Consistency, and Availability

> Goal: explain where copies live, how updates propagate, what clients can observe, and what happens during failure.

## 1. Why replicate

- tolerate machine/zone failure;
- improve read capacity;
- reduce geographic read latency;
- support disaster recovery;
- isolate analytics/reporting.

Replication is not automatically backup: bad deletes and corruption can replicate too.

## 2. Leader–follower replication

Writes go to a leader; followers apply its ordered changes.

- Simple write authority and conflict model.
- Followers can serve reads and fail over.
- Asynchronous followers may lag or lose the newest acknowledged writes if promoted.
- Synchronous acknowledgement improves durability but adds latency and may reduce write availability.

Failover requires failure detection, choosing/promoting a leader, preventing two leaders, redirecting clients, and reconciling the old leader.

## 3. Multi-leader replication

Multiple leaders accept writes, useful across regions or disconnected clients. It improves local write availability/latency but creates conflicts, ordering ambiguity, and complex convergence.

Avoid it unless the workload has a clear conflict strategy: partition ownership, commutative operations, deterministic merge, user reconciliation, or domain-specific resolution.

## 4. Leaderless replication

Clients/coordinators write to and read from multiple replicas. With `N` replicas, write acknowledgement `W`, and read responses `R`, `R + W > N` creates overlap under ideal assumptions.

However, overlap alone does not guarantee linearizable behavior: sloppy quorums, stale versions, clock issues, concurrent writes, failures, and repair semantics matter.

Mechanisms can include versioning, conflict resolution, hinted handoff, read repair, and anti-entropy repair.

## 5. Synchronous vs asynchronous

| Mode | Benefit | Cost |
|---|---|---|
| Synchronous | Stronger durability/freshness before acknowledgement | Higher latency; dependency on replica availability |
| Asynchronous | Low write latency and better local availability | Lag and possible data loss on failover |
| Semi-sync | Wait for selected replicas | Middle-ground; exact guarantee must be stated |

The question is not “sync or async globally?” Different paths may make different choices.

## 6. Consistency models

- **Linearizability:** operations appear atomic in real-time order.
- **Sequential consistency:** one global operation order preserving each client’s program order, not necessarily real time.
- **Eventual consistency:** replicas converge if updates stop and repair succeeds; says little about interim observations.
- **Causal consistency:** causally related operations appear in order.
- **Read-your-writes:** a client sees its completed writes.
- **Monotonic reads:** once a version is observed, later reads do not go backwards.
- **Consistent prefix:** reads observe an order prefix, not later effects without earlier ones.

Use session routing, version tokens, leader reads, or waiting for replica position when product behavior requires session guarantees.

## 7. Staleness is a product decision

Different data can tolerate different lag:

- social like count: seconds may be fine;
- chat message after send: user expects read-your-writes;
- authorization revocation: stale permission may be unsafe;
- account balance/inventory reservation: stronger correctness often needed.

State consistency **per operation**, not for the whole system in one word.

## 8. Replication lag effects

- user writes then cannot see result;
- deleted data reappears;
- monotonicity breaks when requests move between replicas;
- derived jobs run on stale state;
- cache/search shows inconsistent versions;
- failover loses acknowledged async writes.

Monitor lag by time and log position; time alone can hide stalled or bursty replicas.

## 9. Conflict resolution

Last-write-wins is simple but may silently lose data and depends on trustworthy ordering/time. Alternatives:

- version vectors/version checks;
- merge commutative data types;
- preserve siblings for application/user merge;
- assign single-writer ownership;
- make updates operations (`increment`) rather than replacement;
- serialize high-value invariants.

## 10. Availability and failure domains

Replicas on the same machine/rack/zone do not protect against broader failure. Place copies across independent failure domains, but account for network latency and correlated dependencies.

An architecture is only as available as its required synchronous dependencies. A long chain of individually reliable services can have lower end-to-end availability.

## 11. Read scaling caveats

Followers can scale stale-tolerant reads, but:

- replication apply consumes resources;
- long/analytical reads can slow replay;
- application must choose consistency routing;
- writes still bottleneck on leader;
- more replicas increase cost and operational surface.

## Common traps

- Replication does not split the dataset; sharding does.
- More replicas do not automatically scale writes.
- Async replication can acknowledge data not present on a promoted follower.
- Eventual consistency is incomplete without convergence/conflict mechanism.
- `R + W > N` is not a universal linearizability proof.
- Read-your-writes is not guaranteed by arbitrary replica reads.
- Last-write-wins can lose concurrent updates.
- Multi-leader is not the default answer for multi-region.

## Interview checks

1. How would you provide read-your-writes while serving most reads from replicas?
2. What can be lost during asynchronous leader failover?
3. Why may three replicas in one zone be insufficient?
4. Give two alternatives to last-write-wins.
5. When would synchronous cross-region replication be unacceptable?

## 60-second recall

- Leader/follower: simple writes; lag and failover matter.
- Multi-leader: local writes but conflict complexity.
- Leaderless: quorum/version/repair; `R+W>N` is not magic.
- Consistency models describe observable behavior.
- Set guarantees per operation and user expectation.
- Replication improves redundancy/read locality; it is not sharding or backup.

## Sources for deeper revision

- [Google Cloud: Deployment Archetypes](https://cloud.google.com/architecture/deployment-archetypes)
- [Apache Cassandra: Dynamo](https://cassandra.apache.org/doc/latest/cassandra/architecture/dynamo.html)
- [PostgreSQL: High Availability and Replication](https://www.postgresql.org/docs/current/high-availability.html)

