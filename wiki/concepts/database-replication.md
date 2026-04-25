---
title: Database Replication
type: concept
tags: [software, systems, databases]
---

## Definition

The process of copying data from a primary database instance to one or more replicas, enabling read scaling, fault tolerance, and geographic distribution.

## Discussion

**WAL (Write Ahead Log) streaming** is PostgreSQL's native replication mechanism: the primary writes all changes to the WAL and streams it to replicas, which apply the changes to stay current. Replication lag — the delay between a write on the primary and its visibility on replicas — is the key health metric.

**Scaling challenges as replica count grows**: the primary must ship WAL to every replica, increasing network bandwidth and CPU pressure. At ~50 replicas (OpenAI's scale per [[scaling-postgresql-openai]]), this still works with large instance types and high-bandwidth links, but it has a ceiling.

**Cascading replication** addresses this: intermediate replicas receive WAL from the primary and relay it downstream to further replicas. This decouples the fan-out from the primary, theoretically allowing 100+ replicas without primary overload. As of 2026, OpenAI is testing this with the Azure PostgreSQL team.

**High-availability standby**: a hot standby is a fully synchronized replica that can be promoted to primary quickly if the primary fails. This reduces the blast radius of a primary failure — reads keep serving from replicas even if writes go down, degrading from SEV-0 to a partial outage.

**Read/write routing**: application logic must route reads to replicas and writes to the primary. Some reads must stay on the primary (those inside write transactions), so query efficiency on the primary is doubly important.

## Key sources

| Article | What it contributes |
|---------|---------------------|
| [[scaling-postgresql-openai]] | Real-world account of managing ~50 replicas, WAL fan-out limits, cascading replication testing |

## Related concepts

[[PostgreSQL Scaling]], [[Connection Pooling]]
