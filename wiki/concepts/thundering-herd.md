---
title: Thundering Herd
type: concept
tags: [software, systems, databases]
---

## Definition

A failure pattern where many clients simultaneously request the same resource — typically after a cache miss or service recovery — causing a sudden spike that overwhelms the backend. Also called a "cache stampede" or "dog-pile effect."

## Discussion

The classic sequence (from [[scaling-postgresql-openai]]): a caching layer fails → many requests simultaneously miss on the same keys → all hit the database at once → CPU saturates → query latency rises → client retries → load amplifies → cascade failure. The vicious cycle can bring down an entire service even if the original trigger was minor.

**Cache locking (cache leasing)** is the primary mitigation: when multiple requests miss on the same key, only one acquires a lock and fetches from the backend. All others wait for the cache to be repopulated. This converts N parallel DB hits into 1, breaking the cascade at its source.

Other mitigations used in practice:
- **Rate limiting** at multiple layers (application, proxy, query) to shed load before it reaches the DB.
- **Retry backoff** — overly short retry intervals amplify thundering herd; exponential backoff with jitter is standard.
- **Workload isolation** — separate instances per tier so a stampede on one workload doesn't cascade into another.

The pattern is not exclusive to caching: a retry storm after a timeout, or a surge of expensive queries after a new feature launch, follow the same amplification dynamic.

## Key sources

| Article | What it contributes |
|---------|---------------------|
| [[scaling-postgresql-openai]] | Cache locking as the fix; full vicious cycle pattern at ChatGPT scale |
| [[uber-intelligent-load-management]] | Static CoDel timeouts caused thundering herd from simultaneous retries; Cinnamon's PID control as the fix — smooth rejection vs. abrupt waves |

## Related concepts

[[postgresql-scaling|PostgreSQL Scaling]], [[connection-pooling|Connection Pooling]], [[load-shedding|Load Shedding]]
