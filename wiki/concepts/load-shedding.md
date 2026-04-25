---
title: Load Shedding
type: concept
tags: [software, systems, databases]
---

## Definition

The deliberate rejection of incoming requests to protect a system from overload — accepting reduced throughput now to maintain stability and preserve capacity for the most important work.

## Discussion

Load shedding is not rate limiting. Rate limiting enforces per-caller quotas proactively; load shedding is reactive, triggered by actual system stress signals. The key design decisions are: what signal triggers shedding, which requests to drop, and where in the stack to shed.

**What signal to use**

QPS is too coarse — it ignores request cost variation (a full table scan vs. a single-row lookup look the same). **Concurrency** is the better signal: via Little's Law (Concurrency = Throughput × Latency), it directly reflects how loaded a system is. A spike in concurrency means latency is rising, which means the system is under real pressure. Uber's load manager ([[uber-intelligent-load-management]]) is built entirely around concurrency as its primary overload detector.

**Which requests to drop**

Priority-agnostic shedding (drop whoever is at the front/back of the queue) is insufficient in multitenant systems with mixed workloads. Priority-aware shedding — dropping background jobs, async pipelines, and low-tier requests first — protects user-facing traffic. Uber's Cinnamon shedder uses tiers t0 (critical infra) → t5 (background), always shedding lowest-priority first.

**Where to shed**

Shedding must live close to the storage layer, not at the routing or API layer. Routing layers lack full context on storage health; their decisions are coarse and often wrong. Both Uber ([[uber-intelligent-load-management]]) and implicitly OpenAI ([[scaling-postgresql-openai]]) arrive at this principle independently.

**Static vs. dynamic limits**

Static inflight limits and queue timeouts require manual tuning and cause instability — they under-shed when load is moderate and over-shed when spiky, triggering [[thundering-herd|Thundering Herd]] from simultaneous rejections. PID-based dynamic control (used by Uber's Cinnamon) continuously adjusts limits based on real-time latency and error signals, producing smooth "dimmer switch" behavior instead of abrupt "hammer" rejection.

**Algorithms in practice**

- **CoDel (Controlled Delay):** Borrowed from networking (bufferbloat mitigation). Tracks queue wait time rather than queue length. Switches from FIFO to LIFO under pressure — fails stale requests fast and favors fresh ones still likely to succeed. Good foundation but priority-agnostic.
- **Cinnamon:** Uber's priority-aware shedder. Uses PID control for dynamic threshold adjustment and a tiered priority model. Supports pluggable external signals (e.g., Raft follower commit lag) via a "BYOS" (Bring Your Own Signal) framework.
- **Token bucket:** Simple and predictable per-caller quota enforcement, but reactive and spiky under pressure — creates abrupt rejection waves that trigger thundering herd.

**Fairness alongside shedding**

System-wide shedding doesn't solve noisy neighbors who hog resources below the global overload threshold. A separate per-tenant concurrency limit layer (Uber's Scorecard) runs in parallel to enforce fairness even during healthy operation.

**Results from Uber (Cinnamon vs. token bucket):**
- 80% more throughput under overload
- ~70% lower P99 latency
- ~93% fewer goroutines at peak
- ~60% lower heap usage

## Key sources

| Article | What it contributes |
|---------|---------------------|
| [[uber-intelligent-load-management]] | Full evolution from static → dynamic → unified shedding; concrete metrics; lessons learned |
| [[scaling-postgresql-openai]] | Complementary: multi-layer rate limiting and cache locking as shedding alternatives |

## Related concepts

[[thundering-herd|Thundering Herd]], [[postgresql-scaling|PostgreSQL Scaling]], [[database-replication|Database Replication]]
