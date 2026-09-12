# 03 — API Design and Service Communication

> Goal: choose a communication style and define contracts that remain correct under retries, partial failures, and evolution.

## 1. Design APIs from use cases

For every core operation define:

- resource/action and caller;
- request, response, and error contract;
- authentication/authorization;
- idempotency and retry rules;
- pagination/order/filtering;
- expected payload, frequency, and latency;
- sync/async acknowledgement boundary.

```http
POST /v1/orders
Idempotency-Key: 6b8...

GET /v1/conversations/{id}/messages?before=cursor&limit=50
```

## 2. REST essentials

- Model durable nouns as resources.
- Use HTTP methods according to semantics: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`.
- `GET` should be safe; `PUT` is intended to replace/upsert a known resource and is idempotent by semantics; `POST` is not automatically idempotent.
- Return useful status codes and machine-readable errors.
- Do not expose internal database tables as the public API by default.

### Idempotency

An idempotent operation can be repeated with the same intended effect as once. Network retries can arrive after the first attempt succeeded.

For create/payment operations, accept a scoped idempotency key, atomically store it with request identity/result, reject mismatched payload reuse, and define retention.

## 3. Pagination

| Method | Strength | Weakness |
|---|---|---|
| Offset/limit | Simple; random page jump | Large offsets cost work; concurrent writes cause skips/duplicates |
| Cursor/keyset | Stable and scalable over ordered data | Cursor design and reverse/random navigation are harder |

Use a deterministic order such as `(created_at, id)`. The cursor should be opaque to clients and tied to filter/order semantics.

## 4. REST vs RPC/gRPC vs GraphQL

| Style | Good fit | Trade-off |
|---|---|---|
| REST/HTTP | Public resource APIs, broad tooling | Over/under-fetching and multi-call workflows |
| RPC/gRPC | Internal typed, low-latency service calls and streaming | Tighter coupling; browser/public compatibility considerations |
| GraphQL | Client-selected graph-shaped reads | Query-cost control, caching, authorization, N+1 complexity |

Protocol choice does not solve poor service boundaries or chatty call graphs.

## 5. Synchronous vs asynchronous

### Synchronous

Caller waits for a response. Simpler immediate semantics but latency and availability depend on the downstream chain.

### Asynchronous

Caller receives acceptance and work continues via a queue/event. Buffers bursts and decouples availability, but introduces delayed completion, duplicates, ordering, status tracking, and eventual consistency.

Return `202 Accepted` only with a way to observe status or receive completion when users need it.

## 6. Real-time delivery options

| Mechanism | Direction | Best fit |
|---|---|---|
| Polling | Client repeatedly requests | Simple, low-frequency updates |
| Long polling | Request waits until update/timeout | Moderate compatibility, fewer empty responses |
| SSE | Server → client stream | Notifications/live feeds over HTTP |
| WebSocket | Bidirectional persistent connection | Chat, collaboration, interactive state |
| Webhook | Server → another server callback | Cross-service event notification |

Persistent connections require connection gateways, heartbeats, reconnection, backpressure, per-user routing, and capacity based on concurrent sockets—not only QPS.

## 7. Webhook correctness

- Sign payloads and verify timestamp/replay window.
- Include stable event ID and type/version.
- Retry with bounded exponential backoff.
- Expect duplicates and out-of-order delivery.
- Provide delivery logs and manual replay where appropriate.
- Let receivers acknowledge quickly and process asynchronously.

## 8. API versioning and compatibility

Prefer additive evolution:

- clients ignore unknown response fields;
- new request fields are optional/defaulted;
- enum evolution is handled safely;
- producers do not remove fields until consumers migrate;
- events carry schema version and stable meaning.

URL/header versioning can manage breaking public API changes, but versioning is not a substitute for compatibility discipline.

## 9. Timeouts, retries, and deadlines

- Every remote call needs a bounded timeout.
- Propagate an end-to-end deadline so downstreams do not work after the caller has given up.
- Retry only transient failures and only when safe.
- Use exponential backoff and jitter.
- Limit attempts and retry budgets.
- Beware layered retries multiplying load.

A timeout means the client lacks an answer; it does not prove the server did nothing.

## 10. API gateway and service discovery

An edge/API gateway may handle TLS termination, authentication, routing, quotas, request validation, observability, and protocol translation. Avoid placing all business logic in it.

Service discovery maps logical service names to healthy instances. It may be client-side, proxy/service-mesh based, or load-balancer based.

## 11. Contract and data security

- Authenticate caller, authorize resource/action.
- Validate size, type, range, and content.
- Avoid sensitive data in URLs/logs.
- Use TLS and protect service-to-service identity.
- Apply rate/usage limits by correct subject.
- Define retention and deletion behavior.

## Common traps

- `POST` is not idempotent merely because the handler checks existence non-atomically.
- Retrying a timed-out payment without a stable key can double-charge.
- WebSocket is transport, not durable message storage.
- Async does not automatically make work reliable.
- Offset pagination can shift under concurrent insert/delete.
- A gateway is not a replacement for authorization inside services.
- “Exactly once API” must define effect, scope, storage, and retry window.

## Interview checks

1. Design an idempotent create-order endpoint.
2. Choose polling, SSE, WebSocket, or webhook for four different use cases.
3. Why can three layers of two retries produce overload?
4. How would you paginate a rapidly changing message history?
5. What does `202 Accepted` promise—and what does it not promise?

## 60-second recall

- API contract includes failure and retry semantics.
- REST, RPC, GraphQL: choose from callers and interaction shape.
- Sync couples latency/availability; async adds state and delivery complexity.
- Cursor pagination needs stable deterministic order.
- Persistent connections require routing, heartbeat, and reconnection.
- Timeout ≠ failure; idempotency makes uncertain retries safe.

## Sources for deeper revision

- [HTTP Semantics: Safe and Idempotent Methods](https://www.rfc-editor.org/rfc/rfc9110.html)
- [Amazon Builders’ Library: Making Retries Safe with Idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)
- [gRPC documentation](https://grpc.io/docs/what-is-grpc/introduction/)

