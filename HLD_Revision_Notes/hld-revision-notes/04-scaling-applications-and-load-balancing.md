# 04 — Scaling Applications and Load Balancing

> Goal: scale traffic without hiding state, overload, or failure problems behind a load-balancer box.

## 1. Scale up vs scale out

- **Vertical scaling:** larger machine. Simple, but has a ceiling, larger failure impact, and may require downtime.
- **Horizontal scaling:** more machines. Adds distribution, coordination, routing, and operational complexity but supports incremental capacity and redundancy.

Start simple. Scale out when throughput, availability, growth, or failure isolation requires it.

## 2. Stateless application tier

A stateless request handler does not require a specific instance to remember user/session state between requests. Store durable/shared state in an appropriate database, cache, or session store.

Benefits:

- any healthy instance can handle a request;
- easy replacement and autoscaling;
- safer rolling deployment;
- simpler load balancing.

“Stateless” does not mean the overall product has no state; it means the service instance is not the only owner of required state.

## 3. Reverse proxy vs load balancer

- **Reverse proxy:** client-facing intermediary for routing, TLS termination, caching, compression, filtering, and hiding origins.
- **Load balancer:** distributes traffic among backends and removes unhealthy targets.

Products often do both. Explain the role, not only the label.

## 4. L4 vs L7 balancing

| Layer | Routes using | Strength |
|---|---|---|
| L4 | IP, port, connection | Low overhead, protocol-agnostic |
| L7 | HTTP host/path/header/cookie | Content-aware routing, auth/rules, richer observability |

L7 can route `/images` and `/api` differently but must understand/terminate the application protocol.

## 5. Selection algorithms

- **Round robin:** simple when requests/servers are similar.
- **Weighted round robin:** accounts for different capacities.
- **Least connections/outstanding requests:** adapts to unequal request duration but measurements can lag.
- **Random/two choices:** simple and effective at scale.
- **Hash-based:** affinity by client/key; naive modulo remaps heavily when membership changes.
- **Consistent hashing:** limits remapping; useful for caches/stateful routing, though balance needs virtual nodes and monitoring.

No algorithm is universally best. Consider request cost, connection duration, locality, cache warmth, and health.

## 6. Health, readiness, and draining

- **Liveness:** should the process be restarted?
- **Readiness:** can it receive new traffic now?
- **Dependency-aware health:** avoid marking every instance unhealthy merely because one shared dependency fails; that can worsen an outage.
- **Connection draining:** stop new work and allow in-flight requests/connections to finish during removal/deployment.

Health checks must test meaningful behavior without becoming expensive or creating a synchronized load spike.

## 7. Sticky sessions

Affinity sends a user/session to the same backend. It may help local session/cache state, but causes uneven load, difficult failover, and scaling friction.

Prefer external/shared session state when feasible. If affinity is necessary, design for backend loss and rebalance.

## 8. Global traffic routing

Options include DNS, anycast, global proxies, or region-aware routing. Policies may prioritize latency, capacity, legal boundary, cost, or failover.

DNS has caching/TTL delay; clients may keep old answers. Global failover must account for data readiness and capacity in the destination, not just route switching.

## 9. Autoscaling

Scale on signals that predict saturation:

- CPU/memory for compute-bound work;
- requests or active connections per instance;
- queue depth **and** oldest-message age;
- custom concurrency/latency metrics.

Account for startup time, cooldown, minimum warm capacity, downstream limits, and scale-in safety. Autoscaling cannot rescue a dependency already overloaded, and it reacts after measurement delay.

## 10. Overload protection

- admission control and rate limiting;
- bounded queues;
- load shedding by priority;
- backpressure;
- concurrency limits;
- degraded/partial responses;
- retry budgets and jitter.

Unbounded queues convert overload into huge latency and eventual collapse.

## 11. Stateful services

Databases, brokers, and WebSocket gateways cannot always be treated as interchangeable stateless workers. Routing may depend on shard, leader, partition, or connection ownership.

Use consistent membership/routing metadata and handle:

- instance loss;
- ownership transfer;
- in-flight work;
- rebalancing;
- stale routing information.

## 12. Common traps

- A load balancer does not remove its own failure risk; deploy it redundantly/managed.
- Horizontal application scaling does not scale a shared database automatically.
- Removing an unhealthy server without draining can terminate valid work.
- DNS failover is not instantaneous because answers are cached.
- Sticky sessions are not durable state.
- CPU is a poor autoscaling signal for connection-bound or I/O-bound workloads.
- Consistent hashing reduces remapping; it does not eliminate hotspots.
- More instances can amplify retries and overwhelm downstreams.

## Interview checks

1. When is L4 preferable to L7?
2. Why can least-connections still make poor choices?
3. How would you scale a WebSocket gateway?
4. What happens when a destination region lacks database state or spare capacity?
5. Design a safe scale-in process for long-running requests.

## 60-second recall

- Vertical is simple; horizontal needs distribution but adds redundancy/scale.
- Stateless instances enable any-instance routing and replacement.
- L4 routes connections; L7 understands application requests.
- Health, readiness, and draining solve different lifecycle problems.
- Autoscale from relevant saturation signals with headroom.
- Protect downstreams using bounded work, admission control, and backpressure.

## Sources for deeper revision

- [Google SRE: Load Balancing at the Frontend](https://sre.google/sre-book/load-balancing-frontend/)
- [Google SRE: Handling Overload](https://sre.google/sre-book/handling-overload/)
- [AWS Well-Architected: Scaling Horizontally](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/rel_compute_horizontally_to_increase_aggregate_workload_availability.html)

