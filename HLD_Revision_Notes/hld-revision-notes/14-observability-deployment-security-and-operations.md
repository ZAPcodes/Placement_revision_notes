# 14 — Observability, Deployment, Security, and Operations

> Goal: show that the proposed system can be operated, changed, secured, and recovered—not merely drawn.

## 1. Observability pillars

- **Metrics:** aggregated numeric behavior; efficient alerting/trends.
- **Logs:** discrete structured events with context.
- **Traces:** request path and timing across components.

Use correlation/request ID across boundaries. Avoid sensitive data and unbounded high-cardinality labels.

## 2. Four golden signals

- **Latency:** successful and failed request latency separately.
- **Traffic:** requests/events/bytes/connections.
- **Errors:** failures and incorrect outcomes.
- **Saturation:** constrained resource—CPU, memory, queue age, pool usage, disk, partitions.

Also track business correctness: successful checkout, notification delay, reconciliation mismatch, search freshness.

## 3. SLI, SLO, SLA, and error budget

- **SLI:** measured behavior, e.g. proportion of valid requests under 300 ms.
- **SLO:** internal reliability target over a window.
- **SLA:** external commitment, possibly with penalties.
- **Error budget:** allowed unreliability under SLO.

Measure from user perspective and define exclusions carefully. Average latency can hide tail failure.

## 4. Alerting

Alert on actionable user-impact symptoms and imminent exhaustion:

- SLO burn rate;
- sustained error/latency rise;
- oldest queue age;
- capacity saturation;
- replication lag/data-pipeline stall;
- reconciliation or durability failure.

Every page should have owner, severity, context, and runbook. Avoid paging for transient internal noise with no action.

## 5. Deployment strategies

| Strategy | Idea | Trade-off |
|---|---|---|
| Rolling | Replace instances gradually | Mixed versions coexist |
| Blue-green | Switch traffic between full environments | Fast rollback; double capacity/state migration care |
| Canary | Send small traffic portion to new version | Needs representative traffic and comparison signals |
| Feature flag | Separate code deployment from exposure | Flag debt and interaction complexity |

Use readiness, connection draining, automated health/SLO gates, and rollback. “Rollback” may not undo schema/data/external side effects.

## 6. Backward-compatible schema change

Expand-and-contract:

1. add compatible schema/field;
2. deploy code that handles old and new;
3. backfill gradually and idempotently;
4. switch reads/writes;
5. validate;
6. remove old field only after all consumers migrate.

Avoid deploying code that requires a column before it exists or dropping a field while old instances/events still use it.

## 7. Authentication and authorization

- **Authentication:** who are you?
- **Authorization:** may you perform this action on this resource?

Authorization belongs near the owning service/data and should be deny-by-default. Gateways can perform coarse checks but downstream services must not blindly trust user-controlled identity/tenant headers.

## 8. Sessions, JWT, and OAuth

- **Server session:** opaque client token; state stored server-side. Easy revocation, adds lookup/state.
- **JWT/self-contained token:** locally verifiable claims. Reduces lookup, but revocation, key rotation, claim staleness, and token size matter.
- **OAuth 2.0:** delegated authorization framework; not itself user authentication. OpenID Connect adds an identity layer.

Use short-lived access tokens, secure refresh flow, key rotation, audience/issuer/expiry checks, and protected cookie settings where applicable.

## 9. Encryption and secrets

- TLS in transit, including internal sensitive paths.
- Encryption at rest with managed key rotation/access control.
- Secrets in a secret manager, not code/images/logs.
- Least-privilege service identities and short-lived credentials.
- Audit privileged actions.

Encryption does not fix excessive authorization or data collection.

## 10. Abuse and rate limiting

- limits per user, token, tenant, IP, device, or expensive operation;
- quotas versus short-term rate limits;
- bot/spam/fraud signals;
- input and upload size/type validation;
- safe URL fetch to prevent SSRF;
- moderation/scanning pipeline;
- anomaly detection and manual controls.

Fail-open vs fail-closed depends on risk: a recommendation quota may fail open; payment/security enforcement usually should not.

## 11. Multi-tenancy

- include tenant identity in authorization and storage keys;
- prevent cache/index/log leakage;
- per-tenant quota and noisy-neighbor isolation;
- encryption/data-region requirements;
- tenant-aware metrics without uncontrolled cardinality;
- deletion/export and dedicated-placement options.

One missing `tenant_id` predicate can become a severe breach.

## 12. Backup and disaster recovery

Define RPO/RTO, full/incremental backup, log/PITR retention, encryption, access separation, and restoration test. Include configuration, secrets, object storage metadata, queues, and external dependencies—not only primary database.

Cross-region failover needs routing, data readiness, spare capacity, identity/config, and a controlled failback plan.

## 13. Operational readiness checklist

- dashboards and SLOs;
- capacity headroom and load test;
- alerts/runbooks/on-call ownership;
- deployment and rollback tested;
- dependency failure behavior;
- data migration/reconciliation;
- backup restore tested;
- security/threat review;
- cost drivers and quotas;
- audit and incident trail.

## Common traps

- Logs alone are not observability.
- High-cardinality user IDs in metric labels can break the metrics system.
- Liveness check should not restart every instance during shared dependency outage.
- JWT is not “more secure” by default and complicates immediate revocation.
- OAuth is authorization; authentication needs an identity layer/protocol.
- Canary without correct success metrics only slows the outage.
- Replication is not backup; failover is not disaster recovery.
- Security cannot be delegated entirely to an API gateway.

## Interview checks

1. Define an SLI/SLO for chat message delivery.
2. What would you alert on for a queue-backed notification system?
3. Roll out a breaking database change without downtime.
4. Server sessions vs JWT for a security-sensitive product?
5. What must exist before claiming regional disaster recovery?

## 60-second recall

- Metrics alert, logs explain events, traces follow request paths.
- Golden signals: latency, traffic, errors, saturation.
- SLO guides operations; error budget measures tolerated unreliability.
- Deploy compatibly with canary/gates/draining/rollback.
- AuthN identifies; AuthZ checks action/resource at owner.
- DR needs RPO/RTO and tested restore/failover, not just replicas.

## Sources for deeper revision

- [Google SRE: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)
- [Google SRE Workbook: Implementing SLOs](https://sre.google/workbook/implementing-slos/)
- [OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/)
- [OAuth 2.0 Security Best Current Practice](https://www.rfc-editor.org/rfc/rfc9700.html)

