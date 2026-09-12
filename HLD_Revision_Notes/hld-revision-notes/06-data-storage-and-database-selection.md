# 06 — Data Storage and Database Selection

> Goal: select storage from queries, invariants, scale, and operations—not from popularity or “SQL vs NoSQL” slogans.

## 1. Start with access patterns

For each dataset ask:

- What are the most frequent reads and writes?
- Point lookup, range scan, full-text search, graph traversal, or aggregation?
- What uniqueness/referential/business invariants exist?
- Is atomic multi-record update required?
- How large and fast-growing is it?
- What ordering and freshness are required?
- Is schema predictable or heterogeneous?
- What retention, deletion, backup, and regional rules apply?

The best database is the one whose model and guarantees fit the dominant workload with acceptable operational cost.

## 2. Storage families

| Store | Strong fit | Main trade-offs |
|---|---|---|
| Relational | Transactions, constraints, joins, flexible queries | Horizontal write scale/partitioning can be complex |
| Key-value | Known-key lookup, sessions, cache, simple state | Limited secondary queries/relationships |
| Document | Aggregate-shaped records, evolving fields | Cross-document relationships and duplicated data need care |
| Wide-column | Huge partitioned write/read patterns | Query-first schema, partition/hotspot/tombstone concerns |
| Search index | Full-text, ranking, faceting | Usually secondary, eventually updated—not authoritative state |
| Time-series | Time-window ingestion, retention, downsampling | Specialized access patterns |
| Graph | Deep relationship traversal | Operational scale and non-graph queries may be weaker |
| Object/blob | Large immutable files/media/backups | Not a general transactional/query database |

These are capabilities, not rigid product categories; modern systems overlap.

## 3. Relational database strengths

- declarative queries and optimizer;
- ACID transactions;
- uniqueness, checks, foreign keys;
- joins and evolving query needs;
- mature indexing, backup, and tooling.

Use a relational database by default when one instance/cluster meets requirements and business correctness matters. Do not shard early merely to appear scalable.

## 4. Key-value and document modelling

Key-value stores favor known access keys. Document stores favor aggregates commonly retrieved/updated together.

### Embed when

- child data is owned by one parent;
- bounded in size;
- read together;
- updated atomically as one aggregate.

### Reference when

- data is shared;
- grows without bound;
- has an independent lifecycle;
- is queried separately;
- duplication would be costly/inconsistent.

“Schema flexible” does not mean “schema absent.” Validation and versioning move into the application/database configuration.

## 5. Wide-column stores

Model tables around queries and partition keys. They can provide high distributed write throughput, but require deliberate control of:

- partition size and access locality;
- hot partitions;
- clustering/order columns;
- denormalized duplicate tables;
- consistency level;
- tombstones and compaction;
- cross-partition operations.

## 6. Search as a derived index

Use a search engine for tokenization, inverted indexes, relevance, faceting, and fuzzy/prefix search. Common architecture:

```mermaid
flowchart LR
    DB[Source database] --> OUT[Change log or outbox]
    OUT --> IDX[Search indexer]
    IDX --> SEARCH[Search cluster]
```

Define freshness lag, replay/rebuild, deletion, and what happens when the search result references an updated/deleted source record.

## 7. Blob/object storage

Large files should usually bypass application servers through signed upload/download URLs. Keep metadata, ownership, status, and object key in a database.

Consider multipart upload, checksum, virus/content scanning, lifecycle tiers, CDN, derived versions, and cleanup of abandoned uploads.

## 8. Source of truth and derived views

Name one authoritative owner for each fact. Caches, search indexes, analytics warehouses, and denormalized read models are usually derived.

For every derived store define:

- propagation mechanism;
- acceptable lag;
- idempotent update;
- ordering/version rule;
- reconciliation/rebuild process;
- deletion/privacy propagation.

## 9. Polyglot persistence

Multiple stores can fit distinct needs, but every added store brings expertise, deployment, monitoring, backup, data synchronization, and incident cost.

Add a second store when its capability produces material value that cannot reasonably be achieved in the existing system.

## 10. Indexes and write cost

Indexes accelerate matching reads/order but add storage, memory, write amplification, and maintenance. Design composite indexes from actual filters/order; avoid indexing every field speculatively.

At HLD level, state example keys:

- `messages(conversation_id, created_at, message_id)`;
- `jobs(status, available_at, priority)`;
- unique `orders(user_id, idempotency_key)`.

## 11. Data lifecycle

Design creation, update, retention, archive, and deletion:

- soft vs hard delete;
- legal/audit retention;
- user deletion across derived systems/backups;
- TTL and partition expiration;
- schema/data migrations;
- backfill rate and production load;
- backup and restore verification.

## 12. Selection answer template

> “The transaction path needs unique order IDs, inventory constraints, and atomic order/payment state transitions, so I’ll use a relational source of truth. Product search is served from a derived search index updated through an outbox. That gives richer search at the cost of bounded indexing lag and reconciliation.”

## Common traps

- “NoSQL is faster/scales better” without workload and guarantee.
- Treating a cache/search index as authoritative accidentally.
- Selecting a database before defining queries.
- Ignoring cross-partition transactions and hot keys.
- Using JSON columns to avoid all schema design.
- Dual-writing two stores without atomicity/reconciliation.
- Assuming read replicas provide read-your-writes.
- Forgetting deletion in indexes, caches, archives, and derived media.

## Interview checks

1. Choose stores for payment state, product search, and product images.
2. When does embedding comments inside a post become harmful?
3. Why is search commonly a derived store?
4. When is adding Redis unnecessary?
5. Defend relational storage for a system expected to grow.

## 60-second recall

- Queries + invariants + scale + operations choose storage.
- Relational is a strong default for transactions and flexible querying.
- Embed bounded owned aggregates; reference shared/independent data.
- Search/cache/warehouse are usually derived, with lag and rebuild plans.
- Blob bytes in object storage; metadata and ownership in DB.
- Every new database creates synchronization and operational cost.

## Sources for deeper revision

- [Google Cloud: Database Options](https://cloud.google.com/products/databases)
- [PostgreSQL documentation](https://www.postgresql.org/docs/current/)
- [Apache Cassandra: Data Modeling](https://cassandra.apache.org/doc/latest/cassandra/developing/data-modeling/index.html)

