# 02 — Capacity Estimation and System Requirements

> Goal: make quick estimates only when they guide architecture. This is a reasoning sheet, not a numerical exercise set.

## 1. Why estimate

Estimation tells you whether one ordinary database is enough, whether content belongs behind a CDN, whether partitioning is necessary, and which resource dominates.

Estimate orders of magnitude—not fake precision.

## 2. Inputs to ask for or assume

- daily/monthly active users;
- operations per active user;
- read/write ratio;
- average and peak request rate;
- object/payload size;
- retention period and growth;
- concurrent connections;
- geographic distribution;
- latency, availability, durability, and freshness targets.

State assumptions aloud and proceed. The interviewer can correct them.

## 3. Core relationships

```text
average QPS ≈ operations per day / 86,400
peak QPS ≈ average QPS × peak factor
storage ≈ objects × bytes per object × retention/replication overhead
bandwidth ≈ requests per second × bytes transferred
concurrency ≈ arrival rate × average time in system
```

Use rounded powers of ten. A 2× difference rarely changes HLD; a 100× difference can.

## 4. Average is not enough

Design for expected bursts:

- time-of-day and regional peaks;
- product launches or ticket sales;
- retry storms after a dependency recovers;
- fan-out from one celebrity or tenant;
- batch jobs sharing production resources.

Use a justified peak factor and say whether the system absorbs bursts using spare capacity, autoscaling, queues, admission control, or degradation.

## 5. Latency and throughput

- **Latency:** time for an operation; examine p50, p95, and p99, not only average.
- **Throughput:** completed work per time unit.
- **Bandwidth:** transferred bytes per time unit.
- **Concurrency:** simultaneously active work/connections.

Improving throughput can worsen individual latency through batching or queuing. Optimize the requirement that matters.

## 6. Availability and durability

- **Availability:** fraction of valid requests the service can handle successfully over a period.
- **Durability:** probability that acknowledged data remains intact.

They are different. A database can be temporarily unavailable while preserving every byte; an available cache can serve while losing evicted entries.

Approximate annual downtime intuition:

| Availability | Maximum downtime/year (approx.) |
|---:|---:|
| 99% | 3.65 days |
| 99.9% | 8.76 hours |
| 99.99% | 52.6 minutes |
| 99.999% | 5.26 minutes |

Do not promise “five nines” without redundant failure domains, automated recovery, safe deployments, operational maturity, and budget.

## 7. Read/write ratio

A read-heavy system may benefit from caching, replicas, denormalized read models, or CDN. A write-heavy system makes replication, index cost, partitioning, batching, and hot keys more important.

Ratios alone are insufficient: one expensive analytical read can cost more than thousands of point reads.

## 8. Storage estimation checklist

Include only relevant factors:

- primary data and metadata;
- indexes;
- replication copies;
- logs/events;
- version/history retention;
- thumbnails/transcoded variants;
- backup/archive;
- compression and encoding;
- temporary processing space.

Store blobs in object storage and metadata in a database when their access/lifecycle needs differ.

## 9. Cache estimation

Estimate the hot working set, not the entire dataset:

- fraction of keys generating most traffic;
- size of cached object;
- acceptable TTL/freshness;
- replication and allocator overhead;
- hit-rate target.

A larger cache does not guarantee a useful hit rate if access is uniformly random or data changes constantly.

## 10. Fan-out amplification

One logical operation may cause many internal operations:

- one post delivered to millions of followers;
- one upload producing several renditions;
- one event consumed by many subscribers;
- one query scattering to every shard.

Estimate amplification at the busiest internal component, not just public API QPS.

## 11. SLO framing

Define service-level indicators for user-visible behavior:

- successful request rate;
- end-to-end latency;
- data freshness;
- notification delivery delay;
- durable ingestion rate.

An SLO is a target for an SLI over a window. An SLA is an external commitment and may include consequences. Error budget is the tolerated unreliability under the SLO.

## 12. When estimates change the design

Say the consequence:

- “Peak writes fit one primary with headroom; I will not shard initially.”
- “Media egress dominates, so use object storage plus CDN.”
- “Millions of persistent sockets require connection-oriented gateway capacity.”
- “A celebrity post creates fan-out spikes, so use hybrid fan-out.”

Numbers without decisions waste interview time.

## Common traps

- Treating daily average as capacity requirement.
- Multiplying units incorrectly or forgetting seconds/day.
- Assuming every user is active simultaneously.
- Counting payload but ignoring replication/indexes/variants.
- Using request QPS where connection concurrency matters.
- Designing exactly at estimated peak with no headroom.
- Quoting extreme availability without defining which operations it covers.
- Spending too long calculating insignificant numbers.

## Interview checks

1. Which estimate determines whether a chat system needs WebSocket gateway sharding?
2. Why can a low-QPS video service still need enormous infrastructure?
3. Give an example where internal QPS greatly exceeds external QPS.
4. Availability vs durability—give a failure that affects only one.
5. How would an SLO influence degradation during overload?

## 60-second recall

- Estimate order of magnitude and peak, not false precision.
- QPS, bytes, storage, concurrency, and fan-out are distinct.
- Average hides bursts and skew.
- Availability ≠ durability; p99 ≠ average.
- Include indexes, copies, logs, versions, and derived media when relevant.
- Every estimate must justify or reject a design choice.

## Sources for deeper revision

- [Google SRE Workbook: Implementing SLOs](https://sre.google/workbook/implementing-slos/)
- [Google SRE Book: Service Level Objectives](https://sre.google/sre-book/service-level-objectives/)
- [AWS Well-Architected: Reliability](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)

