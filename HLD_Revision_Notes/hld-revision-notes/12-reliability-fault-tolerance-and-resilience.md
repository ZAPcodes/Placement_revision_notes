# 12 — Reliability, Fault Tolerance, and Resilience

> Goal: prevent one slow/failing dependency from becoming a system-wide outage.

## 1. Reliability mindset

Assume machines, processes, networks, disks, dependencies, deployments, and humans will fail. Design so failure is contained, visible, recoverable, and tested.

- **Reliability:** performs intended function over time.
- **Availability:** can serve valid requests now.
- **Fault tolerance:** continues acceptable behavior despite specified faults.
- **Resilience:** absorbs, adapts to, and recovers from disruption.

## 2. Remove single points of failure

For every critical component ask:

- Is there redundancy across independent failure domains?
- Is failover automatic and tested?
- Is state replicated/durable enough?
- Does the replacement have capacity?
- Can routing discover the change?
- Could a shared dependency or configuration fail all copies?

Redundant instances in one zone do not protect against zone failure. Identical replicas can share the same software/configuration bug.

## 3. Timeouts and deadlines

Every remote call must stop waiting. Choose timeout from dependency latency distribution, end-to-end budget, retry allowance, and connection/setup cost.

Propagate a deadline through the call chain. Cancel work when the caller no longer needs it where safe. A timeout too low creates false failures/retries; too high ties up resources.

## 4. Retries

Retry only transient failures, only when operation is safe, and within a bounded budget.

- exponential backoff;
- random jitter;
- maximum attempts/elapsed time;
- idempotency key;
- respect server retry hints;
- avoid retries at every layer;
- stop when deadline expires.

If three layers each make three attempts, one request can cause up to 27 downstream calls.

## 5. Circuit breaker

States:

- **Closed:** requests flow; failures counted.
- **Open:** fail fast for a period.
- **Half-open:** allow limited probes to test recovery.

It prevents repeated work against a known-unhealthy dependency and gives recovery room. It does not fix the dependency and can reject traffic incorrectly if tuned poorly.

## 6. Bulkheads and isolation

Separate resources so one workload cannot consume everything:

- connection/thread pools per dependency;
- tenant quotas;
- separate queues for priorities;
- cells/shards/failure domains;
- dedicated critical-path capacity.

Bulkheads trade utilization efficiency for blast-radius control.

## 7. Load shedding and admission control

Reject work before saturation rather than accept everything and time out slowly.

- limit concurrency/queue size;
- reject low-priority work;
- sample telemetry;
- return cached/stale/partial response;
- enforce per-tenant fairness;
- use `429`/`503` plus retry guidance where appropriate.

Good overload behavior preserves a useful core service.

## 8. Backpressure

Slow consumers signal producers to reduce/stop input. Without it, buffers grow until latency/memory/storage collapse.

Mechanisms include bounded queues, flow-control windows, pull-based consumption, quotas, and adaptive concurrency. Backpressure must propagate far enough to the admission point.

## 9. Cascading failures

Common chain:

1. one dependency slows;
2. callers accumulate waiting work;
3. pools/threads/connections exhaust;
4. timeouts trigger retries;
5. retries amplify load;
6. adjacent services fail.

Break it with deadlines, concurrency limits, retry budgets, circuit breakers, load shedding, isolation, and graceful degradation.

## 10. Graceful degradation

Examples:

- serve cached product page without recommendations;
- accept an upload and delay processing;
- make feed chronological if ranking is unavailable;
- disable nonessential analytics;
- read-only mode when safe writes cannot be guaranteed.

Define degraded behavior before incidents and communicate freshness/completeness when user decisions depend on it.

## 11. Disaster recovery

- **RPO:** maximum tolerable data loss.
- **RTO:** maximum tolerable recovery time.

Strategies progress from backup/restore to pilot-light, warm standby, and active-active, with rising cost/complexity. Restore tests, dependency inventory, DNS/routing, secrets/config, and destination capacity are essential.

Replication is not backup; a backup is unproven until restoration is tested.

## 12. Resilience testing

- unit/integration failure injection;
- load and stress tests;
- dependency latency/error simulation;
- zone/instance failure drills;
- backup restoration;
- game days/chaos experiments with safeguards;
- rollback and incident exercises.

Test steady-state metrics and stop conditions. Chaos without hypotheses/guardrails is merely disruption.

## 13. Common traps

- Retry is not harmless; it consumes capacity when the system is weakest.
- Autoscaling may react too slowly and can overload dependencies.
- Huge queues hide overload as latency.
- Failover without spare capacity fails twice.
- Multi-zone deployment can share database/DNS/config dependencies.
- Circuit breakers and health checks need hysteresis to avoid flapping.
- Active-active without conflict semantics is not automatically resilient.
- Graceful degradation must preserve correctness/security.

## Interview checks

1. Trace how a slow database causes a cascading failure.
2. Where should retries occur in a three-service call chain?
3. What is the difference between bulkhead and circuit breaker?
4. How would you protect premium traffic from one noisy tenant?
5. Why can an automatic regional failover make the outage worse?

## 60-second recall

- Redundancy must cross the relevant failure domain.
- Deadline bounds end-to-end work; timeout bounds one call.
- Retry transient + safe + bounded + backoff/jitter.
- Circuit breaker fails fast; bulkhead isolates; shedding protects capacity.
- Bounded queues and backpressure stop hidden overload.
- RPO/RTO guide DR; test restores and destination capacity.

## Sources for deeper revision

- [Google SRE: Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/)
- [AWS Builders’ Library: Timeouts, Retries, and Backoff with Jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)
- [AWS Well-Architected: Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)

