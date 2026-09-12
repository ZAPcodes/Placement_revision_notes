# 08 — Partitioning, Sharding, and Consistent Hashing

> Goal: distribute data while controlling hotspots, cross-shard work, rebalancing, and routing complexity.

## 1. Partitioning vocabulary

- **Horizontal partitioning/sharding:** divide rows/items among partitions.
- **Vertical partitioning:** split columns/features or service ownership.
- **Shard/partition:** subset of data with an ownership/routing rule.
- **Replication:** copies a partition; sharding divides the dataset. They are usually combined.

Shard only when measured/estimated write throughput, storage, working set, or operational isolation exceeds a simpler system.

## 2. Partition-key requirements

A good key:

- spreads load and storage;
- appears in common queries;
- keeps related atomic/local work together;
- remains stable;
- avoids unbounded partitions;
- supports routing without broadcast.

These goals conflict. Select from actual read/write patterns.

## 3. Partitioning strategies

### Hash partitioning

`hash(key) → shard`. Usually spreads diverse keys, but loses natural range locality and makes range scans scatter.

### Range partitioning

Assign ordered ranges. Efficient range scans and archival, but sequential keys/time can create a hot newest range.

### Directory-based

A lookup service/table maps key to shard. Flexible placement but routing metadata needs high availability, caching, and consistency.

### Geographic/tenant partitioning

Localizes data and compliance or isolates tenants. Large tenants/regions may still need sub-sharding.

## 4. Hot partitions

Uniform key count does not imply uniform traffic. Causes:

- celebrity account;
- popular product/content;
- timestamp-leading key;
- one huge tenant;
- low-cardinality partition key;
- synchronized batch workload.

Mitigations:

- add a bucket/salt to hot writes;
- split large tenants/keys;
- cache/replicate hot reads;
- aggregate asynchronously;
- rate limit/isolate noisy tenants;
- choose adaptive placement;
- use hybrid fan-out.

Salting improves distribution but forces multi-bucket reads/aggregation.

## 5. Cross-shard operations

Expensive/complex cases include:

- joins and global aggregation;
- transactions across keys;
- global unique constraints;
- global ordering;
- secondary indexes;
- moving data between shards.

Possible approaches:

- colocate related data by key;
- maintain a global coordination/index service;
- use application-level saga/workflow;
- accept eventual derived results;
- scatter-gather with bounded shard count;
- redesign the requirement.

## 6. Routing

Routing can occur in clients, a proxy/router, or the database. The router needs current partition ownership and must handle stale maps, shard movement, retries, and leader changes.

A routing tier simplifies clients but may bottleneck or become a failure dependency unless replicated and cacheable.

## 7. Resharding and rebalancing

Adding capacity requires moving ownership/data without losing writes or serving inconsistent results.

Typical migration shape:

1. create new ownership plan;
2. copy historical data;
3. capture/dual-apply ongoing changes safely;
4. validate counts/checksums;
5. switch reads/routing;
6. drain old ownership;
7. retain rollback window, then clean up.

Naive dual writes can diverge. Prefer database-supported movement, logs/change streams, idempotent copy, and reconciliation.

## 8. Consistent hashing

Hash servers/partitions and keys into a ring; a key maps to the next owner. Adding/removing a node moves only a portion of keys instead of nearly all modulo assignments.

### Virtual nodes

Each physical node owns multiple positions. This improves balance, supports heterogeneous capacity, and makes movement more granular.

### Limitations

- skew/hot keys remain;
- replication placement needs rules;
- membership changes and failure detection are complex;
- range queries remain poor;
- movement still consumes network/disk capacity.

Consistent hashing is a placement technique, not a complete distributed database.

## 9. Secondary indexes

- **Local index:** each shard indexes its own data; queries without partition key scatter.
- **Global index:** direct lookup across shards; now index update consistency, sharding, and failure are separate problems.

For high-scale systems, a derived search/index pipeline is common. Define lag and repair.

## 10. Tenant isolation

Options:

- shared tables with `tenant_id`;
- separate schema/database for large or regulated tenants;
- tenant-to-shard placement map;
- dedicated capacity for noisy tenants.

Include tenant in keys, authorization, cache keys, quotas, observability, and deletion procedures.

## 11. Time-series partitions

Time windows simplify retention and range queries but the current partition receives all writes. Combine time with another dimension or bucket, then merge reads if write distribution demands it.

Avoid tiny partitions (metadata overhead) and unbounded partitions (hotspots/slow maintenance).

## Common traps

- Sharding before exhausting simpler scaling creates permanent complexity.
- `user_id` is not automatically a good key for celebrity/followers workloads.
- Hash partitioning distributes keys, not necessarily traffic.
- Range partitioning on timestamp can hot-spot the newest shard.
- Cross-shard uniqueness cannot rely on isolated local constraints alone.
- Consistent hashing reduces remapping; it does not guarantee balance.
- Dual-write migration without atomicity/reconciliation can lose changes.
- A scatter query across 100 shards can have poor tail latency.

## Interview checks

1. Choose a partition key for chat messages and explain group-chat hotspots.
2. How would you migrate one large tenant to a dedicated shard?
3. Local vs global secondary index?
4. Why can adding a salt fix writes but hurt reads?
5. What happens to tail latency in scatter-gather queries?

## 60-second recall

- Replication copies; sharding divides.
- Key must balance distribution, locality, routing, and bounded size.
- Hash spreads keys; range preserves order; directory adds flexible metadata.
- Hot keys require explicit mitigation.
- Cross-shard joins, transactions, uniqueness, and ordering are expensive.
- Reshard through copy + change capture + validation + cutover.

## Sources for deeper revision

- [Azure Architecture Center: Data Partitioning](https://learn.microsoft.com/en-us/azure/architecture/best-practices/data-partitioning)
- [Apache Cassandra: Partitioners](https://cassandra.apache.org/doc/latest/cassandra/architecture/dynamo.html#dataset-partitioning-consistent-hashing)
- [Google Cloud Spanner: Schema Design Best Practices](https://cloud.google.com/spanner/docs/schema-design)

