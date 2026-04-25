---
title: Connection Pooling
type: concept
tags: [software, systems, databases]
---

## Definition

A technique where a proxy layer maintains a pool of open database connections that are reused across many application clients — avoiding the overhead of establishing a new connection per request.

## Discussion

PostgreSQL has a hard connection limit (5,000 in Azure PostgreSQL). Each new connection carries setup cost and memory overhead. Without pooling, applications under high concurrency exhaust connections or accumulate idle ones, causing "connection storms."

**PgBouncer** is the standard connection pooler for PostgreSQL. It runs in three modes:
- **Session mode**: one backend connection per client session — minimal benefit for high-concurrency workloads.
- **Transaction mode**: one backend connection per transaction — efficient for typical OLTP.
- **Statement mode**: one backend connection per statement — most aggressive reuse, but incompatible with multi-statement transactions.

At OpenAI's scale ([[scaling-postgresql-openai]]), running PgBouncer in transaction/statement mode cut average connection setup time from **50ms to 5ms**. Each read replica runs its own Kubernetes deployment of PgBouncer pods behind a Kubernetes Service for load balancing. Proxies are co-located with replicas and clients in the same region to minimize network overhead.

Configuration details matter: `idle_timeout` must be tuned to prevent connection exhaustion from lingering idle connections. PgBouncer misconfiguration has caused production incidents.

## Key sources

| Article | What it contributes |
|---------|---------------------|
| [[scaling-postgresql-openai]] | Production deployment pattern at 50-replica scale; concrete latency numbers |

## Related concepts

[[PostgreSQL Scaling]], [[Thundering Herd]]
