---
title: "How Uber Conquered Database Overload: The Journey from Static Rate-Limiting to Intelligent Load Management"
author: Dhyanam Vaidya, Prathamesh Deshpande, Mike Ma, Chaitanya Yalamanchili
publication: Uber Engineering Blog
url: https://www.uber.com/in/en/blog/from-static-rate-limiting-to-intelligent-load-management/
date_published:
date_read: 2026-04-25
tags: [software, systems, databases, engineering]
---

## Summary

A multi-author engineering post from Uber's Storage Platform team tracing the full evolution of overload protection for Docstore and Schemaless — Uber's in-house distributed databases built on MySQL, serving tens of millions of QPS for 170M monthly active users. The article is unusually complete: it walks through three distinct design generations, names what failed in each, and ends with concrete performance numbers. The core thesis is that static, priority-agnostic, locally-scoped shedding is insufficient — effective overload protection must be dynamic, priority-aware, and signal-unified.

## Key points

**The system:** Docstore (full CRUD, transactions) and Schemaless (append-only), both built on MySQL with Raft-based consensus and NVMe SSDs. Three-layer architecture: stateless query engine → stateful storage engine → control plane. Load management lives in the storage layer — the key design principle learned early.

**Phase 1 — Quota-based rate limiting (failed):**
- Quotas stored in Redis from the stateless query engine layer.
- Failed for three reasons: (1) Redis adds a failure point and network hop per request; (2) stateless layer can't accurately track health of thousands of storage partitions; (3) cost model was imprecise — a full table scan returning one row cost the same as a single-row lookup.
- Core insight: *overload management must live as close to the storage nodes as possible.*

**Phase 2 — CoDel + Scorecard + Regulators:**
- CoDel (Controlled Delay, borrowed from networking): separate queues for reads, writes, and slow ops. Switches from FIFO to LIFO under pressure — favors fresh requests, fails stale ones fast.
- Scorecard: per-tenant concurrency limits, deterministic, prevents noisy neighbors from hogging resources without triggering global overload.
- Regulators: plug-in node-local detectors for write bytes, hot partition keys, memory, goroutines.
- Limitation: priority-agnostic (critical ride requests dropped alongside background jobs), static timeouts caused [[Thundering Herd]] when rejected requests all retried simultaneously.

**Phase 3 — Cinnamon replaces CoDel:**
- Priority tiers t0 (critical infra) → t5 (least critical). Sheds lowest priority first, preserving user-facing flows.
- Auto-tuner dynamically adjusts inflight concurrency limits using P90 latency (no manual tuning).
- PID-based control: smooth, continuous adjustment rather than abrupt rejection. "Dimmer switch, not a hammer."
- Limitation: only local signals — no awareness of follower node lag (commit index lag), causing split-brain shedding in distributed partitions.

**Phase 4 — Unified Load Shedding Engine:**
- Cinnamon extended with pluggable external signals (e.g., follower commit lag from Raft replicas).
- "BYOS" (Bring Your Own Signal) ethos: modular framework — any new overload signal plugs in and routes to the right shedding path.
- Unified local + remote overload logic into a single control loop.
- Results vs. token bucket baseline: **80% more throughput** (5,400 vs 3,000 QPS), **~70% lower P99 latency** (1.0s vs 3.1s upsert), **~93% fewer goroutines** at peak (10k vs 150k), **~60% lower heap** (1 GB vs 5–6 GB).

**Lessons learned (direct from the article):**
1. Prioritization is paramount — protect user-facing traffic first.
2. Fail fast, don't block — early rejection beats holding requests in memory.
3. PID regulation for stable shedding — reactive systems overcorrect; PID incorporates history and direction.
4. Place control close to the source of truth (the storage layer).
5. Embrace dynamism — avoid static configuration.
6. Invest in observability — know what's being shed, why, and by which component.
7. Simplicity over complexity.

## What's interesting

The framing of "concurrency, not QPS" as the right overload signal (via Little's Law: Concurrency = Throughput × Latency) is the conceptual heart of the article. QPS-based limits are too coarse — a low-QPS caller sending massive write payloads can overload the system just as easily. Concurrency directly reflects actual resource consumption in stateful systems. This is a durable principle, not an Uber-specific trick.

## Authors & publications

[[Uber Engineering Blog]]

## Concepts

[[Load Shedding]], [[Thundering Herd]], [[PostgreSQL Scaling]]

## Connections

Directly extends [[Thundering Herd]] — CoDel's static timeouts caused exactly the thundering herd pattern described in [[scaling-postgresql-openai]], and Cinnamon's PID control is the fix. Both articles deal with the same cascade failure dynamic from different angles (caching layer vs. queuing layer). The "place control close to the source of truth" lesson mirrors OpenAI's finding that overload management must live at the storage layer, not the routing layer.
