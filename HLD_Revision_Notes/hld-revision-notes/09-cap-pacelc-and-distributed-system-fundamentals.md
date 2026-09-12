# 09 — CAP, PACELC, and Distributed-System Fundamentals

> Goal: reason about partial failure and trade-offs without repeating misleading “pick two” slogans.

## 1. What makes distribution hard

In a distributed system:

- messages can be delayed, duplicated, reordered, or lost;
- nodes can pause, crash, restart, or become unreachable;
- clocks disagree;
- one component can fail while others continue;
- a timeout cannot distinguish failure from slowness;
- operators can act on stale state.

Partial failure means no participant has perfect instantaneous knowledge of the whole system.

## 2. CAP theorem—correct framing

During a **network partition**, a replicated data system cannot simultaneously guarantee:

- **Consistency (C):** every read observes the latest completed write or an error, in the CAP/linearizability sense;
- **Availability (A):** every request to a non-failed node receives a non-error response;
- while tolerating the **Partition (P)**.

If partitions must be tolerated, the system decides per operation whether to reject/delay some requests for consistency or serve potentially divergent/stale results for availability.

## 3. Why “pick any two” is misleading

- P is not a feature you casually disable across unreliable networks.
- The trade-off matters **when communication is partitioned**.
- Systems may make different choices for different operations/data.
- Outside a partition, latency, consistency, cost, and failure handling still trade off.
- Real products often degrade rather than choose one global label.

Example: during a partition, a bank may reject balance-changing writes but continue serving cached statements.

## 4. CP and AP behavior

### CP-style choice

Preserve a single consistent decision by requiring leader/quorum/consensus. Some nodes/regions reject or wait when they cannot prove authority.

Good for uniqueness, locks, inventory reservation, configuration, and money-like invariants.

### AP-style choice

Allow reachable sides to accept/serve, then reconcile. Good when temporary divergence is acceptable and updates can merge or conflicts are tolerable.

Examples: likes, presence, some carts, telemetry.

These are behavior descriptions, not permanent product labels.

## 5. PACELC

PACELC adds the normal-operation trade-off:

- **P:** if partitioned, choose availability or consistency;
- **Else:** when not partitioned, trade latency against consistency.

Cross-region synchronous replication improves freshness/durability but adds wide-area latency. Local asynchronous writes are faster but may be stale or lost during failover.

## 6. Failure detection

Heartbeats and timeouts provide suspicion, not proof. A node considered dead may be slow or partitioned and still acting.

Avoid split brain with:

- quorum/consensus authority;
- leases with safe timing assumptions;
- fencing tokens checked by the protected resource;
- single-writer epochs/terms;
- rejecting stale leaders.

Killing/restarting a suspected process does not itself revoke actions already in flight.

## 7. Time and ordering

Physical clocks can drift or jump. Network arrival order is not a universal event order.

- **Lamport clock:** captures causal ordering implication, not exact wall time or concurrency.
- **Vector clock:** can distinguish causal order from concurrent updates, at metadata cost.
- **Logical sequence/offset:** establishes order within a leader, partition, or log.
- **Hybrid/logical clocks:** combine approximate physical time with logical ordering.

Use server-assigned IDs/sequences, versions, or consensus order when correctness depends on ordering. Timestamps alone are risky for conflict resolution.

## 8. Exactly-once—define the boundary

Networks and processes can fail between doing work and acknowledging it. Delivery is commonly at-least-once or at-most-once; exactly-once **effects** require coordination such as:

- idempotency key/deduplication;
- atomic state + processed-event record;
- transactional messaging within supported boundary;
- deterministic operation and replay;
- reconciliation.

Never promise exactly-once across arbitrary external side effects without stating the protocol and scope.

## 9. Consensus intuition

Consensus lets nodes agree on an ordered value/log despite failures, usually requiring a majority quorum and a leader/term mechanism. It supports metadata, configuration, leader election, and strongly consistent replicated state.

Know the intuition—not Paxos/Raft proofs for this revision pack:

- safety must hold even during partition;
- progress needs enough communicating nodes;
- stale leaders must be rejected;
- losing a majority usually sacrifices availability to preserve safety.

## 10. Split brain

Two partitions believe they are authoritative and accept conflicting work. Prevention requires an authority rule, not merely health checks. Majority quorum, external fencing, or partitioned single-writer ownership are common approaches.

## 11. Common traps

- CAP consistency is not ACID consistency.
- Availability in CAP is stricter than “usually up.”
- CA does not meaningfully tolerate arbitrary network partitions.
- Eventual consistency does not define conflict resolution or maximum staleness.
- Heartbeat timeout does not prove a node is dead.
- Distributed lock expiry without fencing can permit two actors.
- Wall-clock timestamp is not a safe universal total order.
- Consensus improves agreement; it has latency/availability/operations cost.

## Interview checks

1. During a partition, which cart operations could remain available and how would they merge?
2. Why is a health check insufficient to prevent split brain?
3. Explain PACELC for a cross-region database.
4. What does a fencing token prevent?
5. Define exactly-once effect for an email sender.

## 60-second recall

- Distributed systems face partial failure and uncertain time/order.
- CAP trade-off occurs during partition; decide per operation.
- PACELC adds normal-operation latency vs consistency.
- Timeout creates suspicion, not certainty.
- Consensus gives ordered authority with quorum cost.
- Exactly-once requires a defined effect boundary and durable deduplication/atomicity.

## Sources for deeper revision

- [Google SRE: Distributed Consensus for Reliability](https://sre.google/sre-book/managing-critical-state/)
- [Azure Architecture Center: Data Consistency Primer](https://learn.microsoft.com/en-us/previous-versions/msp-n-p/dn589800(v=pandp.10))
- [Google Cloud Architecture Framework: Reliability](https://cloud.google.com/architecture/framework/reliability)

