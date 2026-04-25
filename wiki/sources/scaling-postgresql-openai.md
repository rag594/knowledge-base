---
title: "Scaling PostgreSQL to power 800 million ChatGPT users"
author: Bohan Zhang
publication: OpenAI Blog
url: https://openai.com/index/scaling-postgresql/
date_published: 2026-04-22
date_read: 2026-04-25
tags: [software, systems, databases, engineering]
---

## Summary

A first-person engineering account by [[Bohan Zhang]] of how OpenAI scaled PostgreSQL to millions of queries per second for 800 million ChatGPT and API users — without sharding PostgreSQL itself. The piece is remarkable for showing how far a single-primary architecture can be pushed through disciplined optimization: ~50 read replicas, aggressive caching, workload isolation, connection pooling, and multi-layer rate limiting. 10x load growth in one year is the backdrop; the article reads as both a war story and a playbook.

## Key points

- **Single primary, ~50 read replicas** across multiple Azure regions — no PostgreSQL sharding. Write-heavy workloads are migrated out to sharded systems (Azure CosmosDB) instead.
- **No new tables in PostgreSQL** — all new workloads default to sharded systems. A hard architectural boundary enforced by policy.
- **MVCC write amplification** is the core PostgreSQL weakness at scale: every update copies the full row, creating dead tuples, bloat, and autovacuum complexity. Write-heavy paths must be routed elsewhere.
- **PgBouncer** connection pooling (transaction/statement mode) dropped average connection setup time from 50ms to 5ms. Each replica gets its own Kubernetes deployment of PgBouncer pods behind a load-balancing Service.
- **Cache locking**: during a cache-miss storm, only one reader acquires a lock per key and fetches from PostgreSQL — all others wait. Prevents [[Thundering Herd]] from cascading into DB overload.
- **Workload isolation**: low-priority and high-priority traffic routed to separate PostgreSQL instances. Noisy neighbor problems contained by partition rather than throttle.
- **Rate limiting** at 4 layers: application, connection pooler, proxy, query. ORM layer enhanced to block specific query digests when needed.
- **Schema discipline**: 5-second timeout on all schema changes; only lightweight operations on existing tables; no full table rewrites; backfills rate-limited even if they take a week.
- **12-table join** was a real SEV root cause — multi-way joins are an OLTP anti-pattern; complex join logic should move to the application layer.
- **Cascading replication** (in testing with Azure): intermediate replicas relay WAL to downstream replicas, allowing 100+ replicas without overloading the primary's WAL shipping.
- **Results**: millions of QPS, p99 latency in low double-digit milliseconds, five-nines availability. Only 1 SEV-0 in 12 months — the ChatGPT ImageGen launch, when 100M users signed up in a week and writes spiked 10x.

## What's interesting

The "vicious cycle" pattern is described precisely: upstream issue → cache miss spike → DB overload → query timeouts → client retries → more load → full cascade. This exact failure mode recurs across large-scale systems, and the article's countermeasures (cache locking, rate limiting, workload isolation) are all aimed at breaking one link in that chain. The surprise isn't the techniques individually — it's that careful application of boring tools (PgBouncer, replicas, caching) gets you this far without sharding.

## Authors & publications

[[Bohan Zhang]], [[OpenAI Blog]]

## Concepts

[[PostgreSQL Scaling]], [[Connection Pooling]], [[Thundering Herd]], [[Database Replication]]

## Connections

First systems/databases article in the wiki. Sets up several infrastructure concepts that will recur if more engineering posts are ingested. The MVCC discussion connects to Bohan Zhang's prior writing with Prof. Andy Pavlo ("The Part of PostgreSQL We Hate the Most").
