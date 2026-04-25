---
title: PostgreSQL Scaling
type: concept
tags: [software, systems, databases]
---

## Definition

The set of techniques used to scale PostgreSQL beyond a single instance — primarily by scaling reads horizontally via replicas and routing write-heavy workloads to sharded alternatives, rather than sharding PostgreSQL itself.

## Discussion

The conventional assumption is that you'll eventually need to shard a relational database at sufficient scale. OpenAI's experience (via [[scaling-postgresql-openai]]) challenges this: a single-primary + ~50 read replica architecture served 800M ChatGPT users at millions of QPS, with five-nines availability and p99 latency in low double-digit milliseconds. The key is that ChatGPT's workload is **read-heavy** — write-heavy paths are deliberately routed out to sharded systems (Azure CosmosDB).

**Techniques from practice:**

**Replica scaling**
- Read traffic offloaded to replicas. Near-zero replication lag maintained even at 50 replicas.
- High-availability standby (hot standby) on the primary for fast failover.
- Cascading replication (in testing): intermediate replicas relay WAL downstream, enabling 100+ replicas without overloading primary WAL shipping.

**Write management**
- Write-heavy, shardable workloads migrated to sharded systems.
- No new tables permitted in PostgreSQL — new features use sharded storage by default.
- Backfills rate-limited to prevent write spikes (even if the backfill takes a week).

**MVCC trade-offs**
- PostgreSQL's MVCC copies the entire row on every update — heavy write amplification under write load.
- Dead tuples accumulate, requiring autovacuum tuning. Table/index bloat is a real operational cost.
- This is why write-heavy workloads must be separated out.

**Query discipline**
- Multi-table joins (12-table example was a real SEV cause) are OLTP anti-patterns. Move join logic to the application layer.
- ORM-generated SQL must be reviewed — ORMs regularly produce expensive queries silently.
- `idle_in_transaction_session_timeout` essential to prevent long-running idle queries from blocking autovacuum.
- Schema changes capped at a 5-second timeout; only lightweight operations; no full table rewrites.

**Connection pooling**
See [[connection-pooling|Connection Pooling]]. PgBouncer in transaction/statement mode cut connection setup from 50ms to 5ms.

**Workload isolation**
Separate PostgreSQL instances for low-priority vs. high-priority traffic, and per-product isolation. Noisy neighbor containment by partition rather than throttle.

**Caching + rate limiting**
See [[thundering-herd|Thundering Herd]] for cache locking. Rate limiting at 4 layers: application, connection pooler, proxy, query-level digest blocking.

## Key sources

| Article | What it contributes |
|---------|---------------------|
| [[scaling-postgresql-openai]] | Full production account of all techniques above at ChatGPT scale |

## Related concepts

[[connection-pooling|Connection Pooling]], [[thundering-herd|Thundering Herd]], [[database-replication|Database Replication]]
