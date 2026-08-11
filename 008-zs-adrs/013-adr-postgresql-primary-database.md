# 013-adr-postgresql-primary-database.md
## ADR-013: PostgreSQL 18.4 as Primary Relational Database
### JSONB for FHIR Resources, GIN Indexes, TimescaleDB 2.29.0 for Time-Series, RLS for Multi-Tenancy

**Document type:** ADR
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Accepted

---

## Table of Contents

1. [Decision](#1-decision)
2. [Context](#2-context)
3. [Alternatives Considered](#3-alternatives-considered)
4. [Reason for Decision](#4-reason-for-decision)
5. [Consequences](#5-consequences)
6. [Status](#6-status)
7. [Cross-references](#7-cross-references)

---

## 1. Decision

Use PostgreSQL 18.4 as the primary relational database for the ZarishSphere Platform across all deployment planes (Plane 2+). FHIR R5 resources are stored as JSONB documents with GIN indexes for efficient JSON search. TimescaleDB 2.29.0 extension (minimum version 2.23.0 for PostgreSQL 18 compatibility) provides time-series capabilities for vitals and trend data. Row-Level Security (RLS) enforces multi-tenant data isolation at the database layer. On Plane 0 (air-gapped) and Plane 1 (Raspberry Pi), SQLite serves as the local fallback database with PostgreSQL-compatible query patterns via a feature-flag abstraction layer.

---

## 2. Context

The ZarishSphere Platform requires a database system capable of:

- Storing FHIR R5 resources as flexible JSON documents with fast querying across arbitrary JSON paths
- Supporting multi-tenancy at the database layer so that tenant data is structurally isolated without application-layer filtering logic
- Providing ACID transactions for clinical data integrity — patient records, medication orders, lab results must never be lost or partially written
- Handling time-series workloads for vitals monitoring (heart rate trends, blood pressure over time, lab value trajectories) without a separate time-series database
- Running on Raspberry Pi-class hardware, 4 GB minimum (Plane 1 Edge) in offline edge deployments via SQLite compatibility
- Being completely free with no row limits, storage caps, or usage-based pricing — consistent with ADR-006 (zero-cost toolchain)

The legacy health system stored FHIR resources in MongoDB but encountered consistency issues during concurrent writes and lacked native relational capabilities for the administrative and billing modules. The ZarishSphere Platform requires a single database technology that can serve both the FHIR document store and the relational data model for modules across all 40 domains.

Constitution Law 11 (The platform outlives its creators) requires that the database be maintainable by any competent operator without specialized training. Law 5 (Zero cost is a structural guarantee) prohibits any database with usage-based pricing or paid tiers for essential functionality.

---

## 3. Alternatives Considered

- **PostgreSQL 18.4 + TimescaleDB 2.29.0:** JSONB with GIN indexing provides O(log n) JSON search across FHIR resources. Native uuidv7() for time-ordered FHIR IDs. PostgreSQL 18.4 introduces async I/O providing ~3× improvement on read-heavy FHIR workloads. Row-Level Security is built-in. TimescaleDB 2.29.0 hypertables handle time-series without a separate database. Zero cost — BSD licensed. SQLite compatibility layer for Plane 0/1 deployments. All capabilities in a single database system.

- **MongoDB 8.0:** Native JSON document storage with flexible schema, ideal for FHIR resource variability. Mature aggregation pipeline for analytics. However: no ACID transactions across collections (multi-document transactions are slow and limited), no native relational capability for administrative data, SSPL license creates uncertainty for commercial use, requires separate time-series database (InfluxDB or TimescaleDB), higher memory footprint per connection, and sharding complexity for horizontal scaling. Constitution Law 5 (zero cost) concern with SSPL licensing restricting third-party deployment rights.

- **CockroachDB 25.x:** PostgreSQL wire-protocol compatible, distributed by design, strong consistency across regions, auto-sharding. However: operational complexity for a single-founder project, higher resource consumption (2+ GB RAM baseline), some PostgreSQL features missing or behaving differently, managed service pricing excessive for resource-constrained deployments. The distributed capabilities are unnecessary for Plane 0-2 deployments and add complexity without benefit for the primary use cases.

- **MariaDB 11.x:** Widely deployed, MySQL-compatible, familiar tooling. However: JSON support is less mature than PostgreSQL's JSONB (no GIN equivalent), no native Row-Level Security, no uuidv7(), no async I/O, weaker GIS support for location-aware modules. MariaDB's JSON storage uses LONGTEXT internally with no binary JSON format, making JSON queries significantly slower.

- **SQLite-only:** Zero configuration, single-file deployment ideal for Plane 0/1. However: no concurrent write support (WAL mode helps but does not solve), limited SQL feature set compared to PostgreSQL, no Row-Level Security, no role-based access control, no native time-series extension. Not suitable for Plane 2+ deployments with multiple concurrent users.

---

## 4. Reason for Decision

1. **FHIR JSONB storage with GIN indexing:** FHIR R5 resources are inherently JSON-heavy with variable structures. PostgreSQL's JSONB data type stores JSON in a decomposed binary format that supports indexed search via GIN. This enables efficient FHIR search operations (Patient?birthdate=gt1990-01-01) without a separate search index, keeping architecture simple and reducing the number of services.

2. **Native uuidv7() in PostgreSQL 18.4:** FHIR R5 specifies UUIDv4 for resource IDs but uuidv7 (time-ordered UUID) provides B-tree-friendly sequential ordering. PostgreSQL 18.4 adds native uuidv7() generation, eliminating the need for application-side UUID generation or custom extensions. This improves B-tree index performance for FHIR ID lookups by ~20× compared to random UUIDv4 inserts.

3. **Row-Level Security for multi-tenancy:** RLS enables tenant isolation at the database layer — each query automatically filters by tenant_id via `current_setting('app.tenant_id')`. This eliminates the risk of application-layer data leaks between tenants and simplifies the application code by removing WHERE tenant_id = ? clauses from every query. This is critical for multi-tenant deployments in Plane 3 (national cloud) and Plane 4 (global SaaS).

4. **TimescaleDB 2.29.0 integration:** Vital signs, lab results, and monitoring data are inherently time-series workloads. TimescaleDB 2.29.0 hypertables provide automatic partitioning by time, continuous aggregates for pre-computed rollups, and compression for old data. This eliminates the need for InfluxDB or Prometheus, keeping the stack lean and consistent with ADR-006 (zero cost) and ADR-009 (no vendor lock-in).

5. **Async I/O in PostgreSQL 18.4:** Read-heavy FHIR workloads benefit significantly from PostgreSQL 18.4's async I/O improvements. Benchmark data shows ~3× throughput improvement on JSONB read workloads, critical for FHIR search operations on large resource repositories.

6. **Single database technology reduces operations complexity:** Using one database system across the platform (with SQLite fallback for edge) means the single founder maintains only one database skill. Constitution Law 11 (platform outlives its creators) is better served by a single, well-understood database than by multiple specialized databases.

---

## 5. Consequences

**Positive:**

- FHIR JSONB + GIN indexes provide fast search without a separate search index for most query patterns
- Multi-tenancy enforced at database layer via RLS — no application-layer filtering logic needed
- TimescaleDB eliminates separate time-series database for vitals and monitoring
- PostgreSQL 18.4 async I/O improves read performance on cloud and on-premise deployments
- uuidv7() improves B-tree index performance for FHIR ID lookups
- SQLite compatibility layer allows the same query patterns on Plane 0 and Plane 1
- Single database technology reduces learning curve for future contributors
- Complete zero-cost compliance — no licensing fees, no row limits, no storage caps

**Negative:**

- PostgreSQL 18.4 is relatively new (late 2026 release) — some third-party tooling and ORMs lack full feature support
- TimescaleDB extension must be managed alongside PostgreSQL upgrades — version compatibility requires attention (minimum 2.23.0 for PostgreSQL 18)
- SQLite fallback lacks PostgreSQL 18.4 features (RLS, uuidv7(), TimescaleDB 2.29.0) — feature flags and conditional queries required
- JSONB + GIN is performant for most FHIR queries but may require a dedicated search engine (e.g., Meilisearch or Elasticsearch) for full-text search at scale on Plane 4
- Schema migrations for JSONB-based FHIR storage require careful versioning of the FHIR resource registry

---

## 6. Status

Accepted. This ADR applies to all ZarishSphere Platform deployment planes (Plane 2+) with SQLite as the designated fallback for Plane 0 and Plane 1. All new modules, services, and domain implementations must use PostgreSQL-compatible query patterns. PostgreSQL 18.4-specific features (JSONB, RLS, uuidv7, async I/O) may be used freely with documented SQLite compatibility alternatives.

**Verification:** last_verified August 01, 2026 · next_review February 01, 2027 · verified_by ZarishSphere Foundation

---

## 7. Cross-references

→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: zero-cost toolchain structural requirement
→ **008-zs-adrs/009-adr-no-vendor-lock-in.md** — ADR-009: no vendor lock-in
→ **007-zs-tech-stack/006-oss-tool-catalog.md** — OSS tool catalog (PostgreSQL 18.4 pinned as the primary database)
→ **003-zs-platform/003-deployment-planes.md** — Deployment plane model (Plane 2+ primary, Plane 0/1 SQLite fallback)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
