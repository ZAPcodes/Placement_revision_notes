# 10 — Queues, Messaging, and Event-Driven Architecture

> Goal: choose queue, pub/sub, or stream and explain delivery, ordering, retries, backpressure, and recovery.

## 1. Why asynchronous messaging

- decouple producer latency/availability from consumers;
- buffer bursts;
- run work outside request path;
- fan out events;
- replay/audit a history;
- isolate failure and scale consumers independently.

Costs: delayed completion, duplicates, ordering complexity, backlog, schema evolution, observability, and eventual consistency.

## 2. Queue vs pub/sub vs stream

| Model | Typical behavior | Good fit |
|---|---|---|
| Work queue | One worker from a group handles each message | Jobs, email sending, image processing |
| Pub/sub | Each subscription receives published event | Notifications to independent services |
| Durable log/stream | Ordered records retained by partition; consumers track offsets | Event pipelines, CDC, replay, analytics |

Products overlap. Explain consumption, retention, replay, and ordering rather than relying on labels.

## 3. Commands vs events

- **Command:** request that a specific capability perform an action: `SendEmail`.
- **Event:** fact that already occurred: `OrderCreated`.

Events should use past-tense stable meaning. Do not build hidden synchronous RPC chains through events when immediate success is required.

## 4. Delivery semantics

- **At-most-once:** may lose, avoids broker redelivery.
- **At-least-once:** retries until acknowledged; duplicates possible.
- **Exactly-once:** only meaningful within a carefully defined system/effect boundary.

At-least-once plus idempotent consumer is the common robust design.

## 5. Idempotent consumer

Use a stable message/event ID. Atomically record processing identity with the state change when possible:

```text
BEGIN
  if event_id already processed: no-op
  apply business change
  insert processed(event_id)
COMMIT
```

If the external effect is email/payment/API, use a downstream idempotency key or an outbox-style dispatcher and reconcile uncertain outcomes.

## 6. Acknowledgement and visibility

A work queue commonly hides a claimed message for a visibility/lease timeout. Worker acknowledges after durable effect. If it crashes, the message reappears.

Lease must exceed normal processing or be extended safely. Redelivery can occur even after success if acknowledgement is lost—hence idempotency.

## 7. Ordering

Global ordering reduces parallelism and availability. Prefer the smallest required scope:

- per user;
- per account/order;
- per conversation;
- per partition key.

Partition by the ordering entity. One hot entity can then become a hot partition. Sequence numbers/versions let consumers detect gaps or ignore stale events.

## 8. Consumer groups and scaling

Partitions are assigned among consumers in a group; one partition is usually processed by at most one group member at a time. Parallelism is bounded by partition count.

Rebalancing can pause processing and reassign work. Consumers must commit offsets/state carefully to avoid loss or duplicate effects.

## 9. Retries and poison messages

- bounded exponential backoff with jitter;
- distinguish transient from permanent error;
- limit attempts;
- move poison messages to dead-letter storage;
- preserve error, attempt, and original metadata;
- alert on oldest age and DLQ growth;
- support inspected, controlled replay.

Immediate retry loops can overwhelm a failing dependency.

## 10. Backpressure and overload

Queue depth alone is incomplete; track oldest message age, arrival rate, completion rate, and retry rate.

When arrival exceeds sustainable processing:

- scale consumers if downstream allows;
- throttle/admit less work;
- prioritize/degrade/drop expendable work;
- batch efficiently;
- enforce bounded retention/queue size;
- fix slow dependency.

A queue absorbs a temporary burst, not permanent overload.

## 11. Transactional outbox

Problem: updating the database then publishing can fail between the two operations.

Pattern:

1. write business change and outbox row in one local transaction;
2. relay reads/streams outbox;
3. publish with retry;
4. consumer handles duplicates idempotently;
5. monitor/reconcile stuck events.

The outbox closes the source dual-write gap; it does not create universal exactly-once delivery.

## 12. Event schema evolution

- use stable event type and ID;
- add fields compatibly;
- distinguish event time and processing time;
- avoid exposing private internal table shape;
- keep consumers tolerant of unknown fields;
- version breaking semantics;
- support replay using current or versioned handlers.

## 13. Event sourcing vs event-driven

- **Event-driven:** components communicate using events; state may live conventionally.
- **Event sourcing:** event log is authoritative history and current state is folded from it.

Event sourcing brings audit/rebuild benefits but demands versioning, replay correctness, snapshotting, privacy/deletion strategy, and operational maturity. Do not suggest it casually.

## Common traps

- Queue acknowledgement is not proof the business effect occurred exactly once.
- FIFO is often scoped to one queue/partition/group, not globally.
- More consumers cannot exceed partition-level parallelism.
- A DLQ is not a solution unless monitored and replayed safely.
- Queueing does not increase downstream capacity.
- Retrying permanent validation failures wastes capacity.
- Event sourcing and event-driven architecture are not synonyms.
- Dual writes need outbox/transaction/repair—not hope.

## Interview checks

1. Design reliable email delivery from an order transaction.
2. How would you preserve per-conversation message order while scaling?
3. What metrics reveal a hidden backlog?
4. What happens if a worker completes work but crashes before acknowledgement?
5. Why does the transactional outbox still need idempotent consumers?

## 60-second recall

- Queue: compete for work. Pub/sub: each subscriber. Stream: retained partitioned log.
- At-least-once implies duplicates; acknowledge after durable effect.
- Order only within necessary key/partition.
- Retry transient errors; DLQ poison messages; watch oldest age.
- Queue handles bursts, not infinite overload.
- Outbox fixes DB + publish dual write; consumers still dedupe.

## Sources for deeper revision

- [Google Cloud: Event-Driven Architecture with Pub/Sub](https://cloud.google.com/solutions/event-driven-architecture-pubsub)
- [Apache Kafka: Design](https://kafka.apache.org/documentation/#design)
- [AWS Prescriptive Guidance: Transactional Outbox](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)

