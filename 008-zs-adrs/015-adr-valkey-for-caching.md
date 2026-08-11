# 015-adr-valkey-for-caching.md
## ADR-015: Valkey 9.1.1 as In-Memory Caching Layer
### BSD-3 Licensed, Linux Foundation Governance, Drop-In Redis Compatible

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

Use Valkey 9.1.1 as the primary in-memory caching layer for the ZarishSphere Platform. Valkey is the Linux Foundation's community-maintained fork of Redis, released under BSD-3 license with no commercial use restrictions. It serves as a drop-in replacement for Redis 7.x, providing FHIR resource hot caching, terminology lookup caching (ICD-11, SNOMED CT, LOINC), user session storage, background job queuing via Asynq v0.25.0, and rate-limiting counters.

---

## 2. Context

The ZarishSphere Platform requires a fast in-memory key-value store for several distinct workloads:

- **FHIR resource hot cache:** Frequently accessed Patient, Practitioner, and Observation resources cached to reduce PostgreSQL read load. Cache-aside pattern with configurable TTL per resource type.
- **Terminology lookup cache:** ICD-11 diagnosis codes, SNOMED CT clinical terms, LOINC lab codes, and other terminology systems cached with 24-hour TTL. Terminology lookups are a major source of database reads in clinical workflows.
- **User session tokens:** Authenticated user sessions stored in Valkey for fast O(1) lookup on every API request. Session invalidation on logout, password change, or admin revocation.
- **Background job queue:** Asynq (Go library) uses Valkey as its job queue backend for deferred tasks — report generation, data export, FHIR subscription notifications, cache warming.
- **Rate limiting counters:** Sliding window rate limit counters per API key, per tenant, per IP address for Plane 4 (global SaaS) API gateway.

This must all work within the constraints of Constitution Law 5 (zero cost) and ADR-006: the caching layer must be free, open-source, and self-hosted with no licensing restrictions. The choice between Redis and its forks was triggered by Redis Ltd.'s license change in 2024 from BSD-3 to SSPL (Server Side Public License), which restricts how the software can be offered as a managed service — a direct concern for Constitution Law 9 (vendor freedom) and ADR-009 (no vendor lock-in).

---

## 3. Alternatives Considered

- **Valkey 9.1.1:** Linux Foundation project with BSD-3 license. Fully API-compatible with Redis 7.x — existing Redis clients (valkey-go, go-redis) work without modification. Atomic slot migration for cluster reliability (introduced in Valkey 9.0). Active community with regular releases. The only fork with Linux Foundation governance, ensuring vendor independence. BSD-3 license imposes no restrictions on commercial use, managed service provision, or redistribution — fully consistent with Constitution Law 5 and Law 9.

- **Redis 7.4 (final BSD-3 version):** The terminating Redis release under the original BSD-3 license. Widely deployed, largest ecosystem, most documentation and tooling. However: stuck at 7.4 with no future feature development. Redis 8.0 and beyond are under SSPL, which restricts managed service provision and creates legal uncertainty for deployers who want to offer ZarishSphere as a managed service (Constitution Law 9 concern). Choosing Redis 7.4 means freezing on a version that will never receive security patches or performance improvements for the long-term platform lifecycle.

- **Redis 8.0 (SSPL):** The newest Redis release with new features. However: SSPL license specifically restricts offering the software as a managed service. This directly conflicts with Plane 4 (global SaaS) deployment, where the ZarishSphere Foundation may operate a hosted version. It also conflicts with Constitution Law 5 (zero cost) — SSPL's restrictions create a legal barrier for community deployments. ADR-006 prohibits any dependency with licensing restrictions.

- **DragonflyDB:** Modern in-memory store with Redis compatibility and multi-threaded architecture claiming 25× throughput improvement over single-threaded Redis. However: licensed under BSL (Business Source License) with conversion to Apache 2.0 after 4 years, which creates the same future-licensing-uncertainty pattern as Redis's SSPL change. Smaller community, fewer client libraries, less battle-tested in production. BSL license violates the precautionary principle of ADR-006: no dependency may have a license that could restrict future use.

- **Memcached:** Simple, battle-tested, fast key-value store. However: no persistence, no data structures beyond strings, no pub/sub, no transactions. Cannot serve as job queue backend (no list/set operations needed by Asynq). Lacks the rich data type support that makes Valkey useful for caching FHIR resources as complex JSON structures.

- **Apache Ignite:** Distributed in-memory data grid with SQL support, ACID transactions, and compute grid capabilities. However: heavy (~500 MB+ RAM baseline), complex cluster management, JVM-based (conflicts with ADR-004's prohibition on JVM dependency). Massively over-provisioned for caching use cases.

---

## 4. Reason for Decision

1. **License certainty:** Valkey's BSD-3 license under Linux Foundation governance provides permanent license certainty. Redis's 2024 license change to SSPL demonstrated that a commercially-owned open-source project can change its license terms at any time, leaving downstream projects exposed. Valkey's community ownership under the Linux Foundation prevents this risk. This directly implements the precautionary principle of ADR-006 (zero-cost toolchain) and ADR-009 (no vendor lock-in).

2. **Drop-in Redis compatibility:** Valkey 9.1.1 is fully compatible with the Redis protocol — all existing Redis clients (valkey-go, go-redis, node-redis) work without code changes. The migration from Redis is a configuration change (different server address), not a code change. This eliminates migration risk.

3. **Atomic slot migration:** Valkey 9.1.1 includes atomic slot migration for cluster mode, improving reliability during cluster topology changes. This is important for Plane 4 (global SaaS) where the caching layer may need to scale horizontally.

4. **Linux Foundation governance:** The Linux Foundation provides neutral, community-owned governance. No single company controls Valkey's direction, licensing, or feature set. This ensures long-term viability consistent with Constitution Law 11 (The platform outlives its creators).

5. **Zero-cost compliance:** BSD-3 license imposes no restrictions on commercial use, managed service provision, or redistribution. The software is fully free with no paid tiers, no feature gating, and no usage limits. This satisfies ADR-006's zero-cost requirement unconditionally.

---

## 5. Consequences

**Positive:**

- Permanent BSD-3 license certainty — no SSPL or BSL licensing concerns
- Drop-in Redis compatible — existing Redis clients and patterns work without modification
- Linux Foundation governance ensures community ownership and vendor independence
- Atomic slot migration improves cluster reliability for Plane 4 deployments
- 9.1.1 release includes all core Redis features (strings, hashes, lists, sets, sorted sets, streams, pub/sub)
- Active community with regular security updates
- Zero licensing cost with no commercial use restrictions
- Cross-platform support including ARM64 for Raspberry Pi (Plane 1)

**Negative:**

- Smaller ecosystem than Redis — fewer third-party tools, modules, and extensions
- Fewer managed service options — most cloud providers offer Redis, not Valkey (mitigated by self-hosting)
- Community documentation less comprehensive than Redis's
- Some Redis-specific features (Redis Stack, RediSearch, RedisJSON) may lag in Valkey compatibility
- Migration from Redis requires updating deployment configurations and verifying client compatibility
- Newer project — long-term maintenance track record still being established

---

## 6. Status

Accepted. Valkey 9.1.1 is the in-memory caching layer for all ZarishSphere Platform deployments. Redis 7.x deployments must be migrated to Valkey 9.1.1. All new caching, session storage, job queue, and rate limiting workloads use Valkey. No service may depend on Redis-specific features not yet available in Valkey without documented alternatives.

**Verification:** last_verified August 01, 2026 · next_review February 01, 2027 · verified_by ZarishSphere Foundation

---

## 7. Cross-references

→ **008-zs-adrs/004-adr-no-hapi-fhir.md** — ADR-004: no JVM dependency (rules out Apache Ignite)
→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: zero-cost toolchain structural requirement
→ **008-zs-adrs/009-adr-no-vendor-lock-in.md** — ADR-009: no vendor lock-in (license-change risk)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
