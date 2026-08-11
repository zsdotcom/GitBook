# 021-adr-powersync-mobile-offline.md
## ADR-021: PowerSync 1.23.3 for Mobile Offline Database Synchronization
### Self-Hosted, Bi-Directional SQLite-to-PostgreSQL Sync, Conflict Resolution

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

Use PowerSync 1.23.3 as the mobile offline database synchronization engine for all ZarishSphere Platform mobile and field applications. PowerSync provides bi-directional synchronization between local SQLite databases on mobile devices (Android/iOS via Flutter's drift library) and the central PostgreSQL 18.4 database. The PowerSync server is self-hosted alongside the ZarishSphere Platform services, providing row-level sync rules, CRDT-based conflict resolution, and incremental sync with no full-table scans. All mobile applications — Community Health Worker app, Clinician app, Supervisor app, Patient Portal — synchronize clinical data through PowerSync.

---

## 2. Context

ZarishSphere Platform mobile applications operate in environments with unreliable or intermittent connectivity — the defining characteristic of the target deployment contexts:

- **Community Health Workers** conduct household visits in rural areas with no cellular connectivity. A CHW may be offline for 8-12 hours at a time while still needing to register patients, record vitals, assess symptoms, and update patient records.
- **Clinicians** at rural health posts may have intermittent connectivity via satellite or low-bandwidth cellular. Patient consultations, medication administration, and lab result review must work regardless of connectivity.
- **Supervisors and program managers** perform field monitoring visits where they need access to aggregate data and individual records without an internet connection.
- **Plane 0 (air-gapped) and Plane 1 (Raspberry Pi, local network)** deployments have no internet connectivity by design. Mobile devices synchronize with the local server over Wi-Fi when in range.

Key synchronization requirements:

- **Bi-directional sync:** Data created or modified on the mobile device must propagate to the server, and data created on the server (or by other mobile devices) must propagate to the device. Both directions must work independently.
- **Conflict resolution:** When the same record is modified on two devices while both are offline, the sync engine must detect the conflict and resolve it — either automatically (last-writer-wins, CRDT merge) or manually (present both versions for resolution).
- **Row-level sync rules:** A CHW should only sync patients in their catchment area. A clinician should only sync patients under their care. A supervisor should sync aggregate data but not individual patient records. Sync rules must be configurable at the row level based on user role, tenant, and organizational hierarchy.
- **Incremental sync:** After the initial sync, only changed records should be transmitted — no full-table scans or re-downloads. This is critical for low-bandwidth environments.
- **Offline-first:** The mobile application must be fully functional without connectivity. All reads and writes go to the local SQLite database. Sync is a background process.
- **Clinical data integrity:** No data may be lost during sync failures. All operations must be idempotent — replaying a sync operation must not create duplicate records.

---

## 3. Alternatives Considered

- **PowerSync 1.23.3 (self-hosted, chosen):** Purpose-built for SQLite-to-PostgreSQL bi-directional sync. Provides row-level sync rules (SQL WHERE clauses that filter which rows sync to which device), CRDT-based conflict resolution with customizable strategies (last-writer-wins, client-wins, server-wins, manual), incremental sync via change tracking, built-in compression for low-bandwidth environments, and native Flutter SDK (drift integration). Self-hosted server is Apache 2.0 licensed — zero cost, no external dependency. The sync rules engine enables fine-grained data access control consistent with Constitution Law 4 (privacy by architecture) — a CHW can only sync data they are authorized to see.

- **Couchbase Lite:** Mature offline-first mobile database with built-in sync (Couchbase Sync Gateway). Full-text search, conflict resolution, and flexible data model. However: Couchbase Sync Gateway requires Couchbase Server as the backend — adding a second database system to the stack (incompatible with ADR-013's PostgreSQL-primary decision). The document model (not relational) requires data mapping between Couchbase documents and PostgreSQL FHIR resources. Resource footprint is larger than SQLite + PowerSync. Licensing: Couchbase Server is available under BSL — licensing concerns consistent with ADR-006's precautionary principle.

- **MongoDB Realm (now Atlas Device Sync):** Mobile SDK with Atlas-integrated sync, offline-first data model, and conflict resolution. However: Realm was acquired by MongoDB and the licensing and product direction has been in flux — MongoDB Atlas Device Sync requires an Atlas cluster (cloud dependency, vendor lock-in). The SDK has gone through multiple breaking API changes. Realm's object model is not SQL-based — the mobile app would use a different data model than the server's PostgreSQL. Conflicts with ADR-009 (no vendor lock-in) and ADR-006 (zero cost — Atlas has usage-based pricing).

- **Custom sync engine built on NATS + PostgreSQL change tracking:** Build a custom sync engine using PostgreSQL's logical replication, NATS JetStream for message delivery, and application-level conflict resolution. Full control, no third-party dependency, stack integration. However: building a production-grade bi-directional sync engine with conflict resolution, incremental sync, and row-level sync rules is a multi-month engineering effort. The single founder cannot afford to build and maintain a custom sync engine while also building the platform. The risk of subtle data loss bugs in a custom sync engine is unacceptable for clinical data.

- **Manual CSV/JSON export + import:** The simplest offline data transfer mechanism — export data to files on the server, transfer them to the mobile device via USB or SD card (or email), import them into the mobile app, make changes, export back, transfer, and import on server. Used in many humanitarian health deployments today. However: no conflict resolution, no incremental sync, no row-level filtering (all data exported), no real-time or near-real-time synchronization, extremely error-prone, and violates clinical data integrity requirements. Acceptable for Plane 0 paper-trail backups but not as the primary sync mechanism.

---

## 4. Reason for Decision

1. **Purpose-built for offline-first health deployments:** PowerSync was designed for exactly the ZarishSphere use case — mobile health workers with unreliable connectivity needing to sync clinical data to a PostgreSQL backend. Its feature set (row-level sync rules, incremental sync, CRDT conflict resolution) directly matches the requirements of the target deployment contexts.

2. **PostgreSQL-native:** PowerSync syncs to PostgreSQL directly — no intermediate database, no data model translation. The server-side database remains PostgreSQL (per ADR-013), and the mobile device uses SQLite (which is PostgreSQL-compatible at the SQL level). This avoids the data model fragmentation that alternative sync solutions would introduce.

3. **Self-hosted, zero-cost, vendor-independent:** PowerSync server is Apache 2.0 licensed and fully self-hosted. No cloud dependency, no API key, no usage-based pricing, no credit card required. Runs on the same server as the ZarishSphere Platform services. Fully satisfies ADR-006 (zero-cost toolchain) and ADR-009 (no vendor lock-in).

4. **Row-level sync rules for data privacy:** PowerSync's sync rules (SQL WHERE clauses per table per user role) implement Constitution Law 4 (privacy by architecture) at the data synchronization layer. A CHW's device only contains data for patients in their catchment area — no other patient data is present even if the device is lost or compromised.

5. **Flutter SDK integration:** PowerSync provides a ready Flutter SDK with drift (SQLite ORM) integration. This directly supports ADR-023's decision to use Flutter for cross-platform mobile applications. The sync client is a single dependency — the `powersync` Flutter package — reducing integration complexity.

---

## 5. Consequences

**Positive:**

- Mobile applications work fully offline — all reads and writes hit local SQLite
- Bi-directional sync with PostgreSQL — no database fragmentation
- Row-level sync rules enforce data privacy at the sync layer (Constitution Law 4)
- CRDT-based conflict resolution prevents data loss from concurrent offline edits
- Incremental sync minimizes bandwidth usage — only changed rows transmitted
- Self-hosted server — zero cost, no external cloud dependency
- Flutter SDK with drift integration — direct support for ADR-023
- Compression for low-bandwidth environments (satellite, 2G networks)
- Idempotent operations — safe to replay failed syncs

**Negative:**

- PowerSync server adds another service to deploy and maintain alongside the platform
- Sync rule configuration requires careful SQL — incorrect rules can leak data or block legitimate access
- CRDT merge strategies may not be suitable for all clinical data types — some conflicts require manual resolution (e.g., medication dosage changes)
- Initial sync of large datasets over low-bandwidth connections can be slow — requires progress tracking and resume capability
- PowerSync's feature set is evolving — some capabilities (e.g., selective sync of JSONB sub-fields) may require future PowerSync versions
- Dependency on a specific third-party sync engine — if PowerSync were to change licensing or discontinue the self-hosted option, migration to an alternative sync engine would be complex
- Not suitable for real-time collaborative editing — sync latency is seconds to minutes, not milliseconds

---

## 6. Status

Accepted. PowerSync 1.23.3 (self-hosted) is the mobile offline sync engine for all ZarishSphere Platform mobile and field applications. All mobile applications using Flutter (ADR-023) synchronize through PowerSync. The PowerSync server is deployed alongside the platform services and configured with row-level sync rules per domain module. Plane 0 deployments may use alternative sync mechanisms (manual export/import, Wi-Fi direct sync) with documented procedures.

**Verification:** last_verified August 01, 2026 · next_review February 01, 2027 · verified_by ZarishSphere Foundation

---

## 7. Cross-references

→ **008-zs-adrs/006-adr-flutter-mobile-apps.md** — ADR-006: zero-cost open source toolchain requirement
→ **008-zs-adrs/009-adr-no-php-web-backends.md** — ADR-009: no vendor lock-in
→ **008-zs-adrs/013-adr-postgresql-primary-database.md** — ADR-013: PostgreSQL 18.4 as the primary database (sync target)
→ **008-zs-adrs/023-adr-flutter-cross-platform-mobile.md** — ADR-023: Flutter cross-platform mobile framework (PowerSync client target)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
