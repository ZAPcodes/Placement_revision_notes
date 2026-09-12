# 13 — Data Correctness and Distributed Workflows

> Goal: maintain business invariants when one ACID transaction cannot cover every service and external effect.

## 1. State the invariant first

Examples:

- one idempotency key creates at most one order;
- confirmed seats never exceed capacity;
- ledger entries balance;
- one coupon is redeemed at most once per user;
- an accepted job is eventually processed or visibly failed.

“Strong consistency” is vague. Name the invariant, scope, and failure behavior.

## 2. Prefer local transactions

Keep data requiring atomicity in one transactional boundary when possible. A database unique/check constraint or conditional update is stronger and simpler than application-side “check then write.”

```sql
UPDATE inventory
SET available = available - 1
WHERE sku = ? AND available > 0;
```

Affected-row count decides success atomically.

## 3. Optimistic concurrency

Read a version, then update only if it is unchanged:

```sql
UPDATE orders
SET status = ?, version = version + 1
WHERE id = ? AND version = ?;
```

Good when conflicts are uncommon. On zero rows, reload/retry/reject. It prevents silent overwrite but does not automatically protect multi-row predicates.

## 4. Pessimistic locking

Lock before making a decision when conflicts are frequent or retry cost is high. Keep transactions short and access resources in consistent order. Row locks may not protect predicate-wide invariants unless the DB/isolation mechanism covers the range/logical condition.

## 5. Idempotency and deduplication

Store a stable request/event identity atomically with the effect. Define:

- scope: user/merchant/API;
- payload fingerprint and mismatch behavior;
- processing/completed/failed state;
- stored response;
- expiry and replay window;
- concurrency handling;
- recovery after “in progress” owner dies.

Idempotency does not mean ignoring every duplicate; it means returning/reproducing the intended single effect safely.

## 6. Dual-write problem

Updating database A then system B can fail between steps. Reversing order merely reverses the inconsistency.

Options:

- colocate state in one transaction;
- transactional outbox/change-data capture;
- distributed transaction where supported and justified;
- saga/compensation;
- idempotent repair/reconciliation.

## 7. Two-phase commit (2PC) intuition

Coordinator asks participants to prepare, then commits or aborts.

Benefits: atomic decision across participating transactional resources.

Costs:

- blocking/availability during coordinator or network failure;
- locks held longer;
- operational recovery complexity;
- all resources must support protocol;
- does not include arbitrary real-world effects.

Do not confuse 2PC atomic commit with consensus; they solve related but different agreement/failure problems.

## 8. Saga

A saga decomposes a business workflow into local transactions. On failure, later steps stop and compensating actions semantically undo completed work.

- **Orchestration:** central workflow decides next step; clearer visibility/control, central dependency.
- **Choreography:** services react to events; loose coupling, but flow can become implicit and hard to debug.

Compensation is not database rollback: refund is a new event, an email cannot be unsent, and inventory may have changed. Design states such as `PENDING`, `CONFIRMED`, `CANCEL_PENDING`, `REFUNDING`, `FAILED`.

## 9. Transactional outbox and inbox

Outbox atomically stores state change and event-to-publish. Relay publishes at least once. Consumer inbox/processed-event table deduplicates atomically with its local effect.

Need monitoring, retention, retries, ordering/version handling, and replay/reconciliation.

## 10. Reservation pattern

For scarce resources:

1. atomically reserve with expiry;
2. complete payment/confirmation;
3. convert reservation to confirmed;
4. release/expire on failure;
5. reconcile stuck states.

Reservation reduces overselling but temporarily lowers availability and requires clock/expiry/late-confirmation rules.

## 11. Money and ledgers

Prefer immutable double-entry ledger records as authoritative financial history; derive balances. Every transfer creates balanced debits/credits under a transaction and uses idempotent business identity.

Do not use floating point for money. Separate authorization, capture, settlement, refund, and reconciliation states. External provider timeout creates unknown outcome; query/reconcile using stable provider key.

## 12. Exactly-once effect

For a defined effect, combine:

- unique operation/event ID;
- durable processed state;
- atomic state transition;
- idempotent downstream call or status query;
- retry/replay;
- reconciliation.

If physical duplicate execution is possible but the committed business state is one, describe it as exactly-once **effect** within that boundary.

## 13. Reconciliation

Online correctness mechanisms can fail due to bugs or unknown external outcomes. Periodic reconciliation compares independent records/source-of-truth, detects missing/duplicate/mismatched state, and repairs or escalates.

It is essential for payments, inventory, event pipelines, and derived indexes—not an admission that the design failed.

## Common traps

- Check-then-insert is racy without atomic uniqueness/locking.
- Retry without idempotency can duplicate business effects.
- Saga compensation is not perfect rollback.
- Outbox does not make consumers exactly-once automatically.
- 2PC cannot atomically unsend an email.
- A distributed lock cannot preserve correctness if the storage ignores stale owners.
- “Eventually consistent” does not excuse violating money/inventory invariants.
- Ignoring unknown outcomes after timeouts is dangerous.

## Interview checks

1. Prevent duplicate orders from simultaneous retries.
2. Reserve the last concert seat while payment may take minutes.
3. Compare 2PC and saga for checkout.
4. Why does an outbox relay publish duplicates?
5. How would you reconcile payment-provider state with internal orders?

## 60-second recall

- Name invariant and atomic boundary first.
- Prefer local transaction/constraint/conditional update.
- Idempotency = stable identity + atomic recorded effect.
- Dual writes need transaction, outbox, saga, or repair.
- Saga is local transactions + semantic compensation.
- Financial/external workflows need explicit states and reconciliation.

## Sources for deeper revision

- [AWS Prescriptive Guidance: Saga Pattern](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/saga.html)
- [AWS Prescriptive Guidance: Transactional Outbox](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)
- [Amazon Builders’ Library: Idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)

