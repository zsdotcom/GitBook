# 004-adr-no-hapi-fhir.md
## ADR-004: Rejection of Java HAPI FHIR and Decision to Build a Go-native FHIR Server
### No JVM-based FHIR implementation due to 8 GB RAM constraint

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

Reject HAPI FHIR and all JVM-based FHIR implementations. Build a custom Go-native FHIR R5 server instead. No Java Virtual Machine will be installed on any ZarishSphere development, build, or deployment machine. The FHIR server will be written entirely in Go 1.26.5, producing a single ~15 MB static binary with no runtime dependencies.

---

## 2. Context

HAPI FHIR is the dominant open-source FHIR implementation. It is the reference server used by the HL7 FHIR community, supports both R4 and R5, and provides extensive validation, terminology, and persistence capabilities out of the box. However, HAPI FHIR runs on the Java Virtual Machine (JVM), which imposes structural requirements incompatible with the ZarishSphere constitution:

- **Constitution Law 11** (platform outlives its creators): The platform must remain functional on the founder's hardware — a Lenovo i3 with 8 GB RAM (founder profile §4.1). The JVM baseline consumes 2-4 GB before any FHIR server logic loads.
- **Constitution Law 5** (zero-cost structural guarantee): Some JVM performance monitoring and profiling tools require paid licensing.
- **Constitution Law 8** (deployment plane sovereignty): Plane 0 (air-gapped, single device) and Plane 1 (Raspberry Pi) cannot run a JVM-based server with acceptable performance.
- **Constitution Law 9** (vendor freedom): HAPI FHIR has a large dependency tree (~200+ JAR files), creating transitive dependency risks.

Additionally, the founder has no Java development experience and would need to maintain both Go (backend platform) and Java (FHIR server) codebases — an unsustainable burden for a solo builder.

---

## 3. Alternatives Considered

| Alternative | Pros | Cons |
|---|---|---|
| **Build custom Go FHIR server** | Single ~15 MB binary; 50-150 MB RAM at runtime; no JVM overhead; Go is the primary language (ADR-001); single codebase for entire platform; cross-compiles to ARM64 for RPi; full control over FHIR R5 implementation | Must implement from scratch: validation engine, search parameters, terminology server, persistence layer; no FHIR bulk data API out of the box; no community plugin ecosystem; significant development effort estimated at 4-6 months |
| **HAPI FHIR (Java/JVM)** | Gold standard FHIR implementation; comprehensive validation (instance-level, profile-level); built-in terminology server (UMLS, Snomed, Loinc); R4 and R5 support; active community and commercial support | Requires 2-4 GB JVM heap minimum; 500 MB+ deployment footprint; Java development overhead; founder does not write Java; violates Constitution Law 11 (8 GB RAM constraint); 30-60 second startup times incompatible with Plane 0/1 |
| **Firely .NET FHIR Server** | Mature .NET implementation; good R5 support; built-in terminology services | Requires .NET runtime (~200 MB); founder does not write C#; Windows-oriented tooling; Linux runtime via Mono is unreliable for production; adds second language to stack |
| **Azure API for FHIR (managed)** | Fully managed; no server maintenance; supports R4 | Vendor lock-in to Azure (violates Law 9); $500+/month minimum cost; requires internet connection (incompatible with Plane 0/1); not available on free tier; R5 support uncertain |
| **Use FHIR as data model only (no FHIR server)** | Simplest approach — store data in SQLite and serve via REST API without FHIR validation | Loses all FHIR interoperability value; cannot exchange data with other FHIR systems; no standard query syntax (search parameters); violates Constitution Law 3 (every standard is executable — FHIR R5 is the standard data model) |

---

## 4. Reason for Decision

1. **RAM constraint is absolute:** The development machine has 8 GB RAM total. Running Ubuntu 26.04 LTS with VS Code, a browser with multiple tabs, Docker (if used), and a JVM-based FHIR server is structurally impossible. The JVM alone would consume 2-4 GB for heap, plus another 500 MB-1 GB for class loading, GC overhead, and monitoring. A Go FHIR server binary runs at 50-150 MB resident memory — a 20-40x reduction.

2. **Single-language ecosystem:** ADR-001 establishes Go as the primary backend language. Adding Java for FHIR would require the founder to write, debug, and maintain code in two languages. The founder profile (§2.4) lists Go at reading-level and Java as not proficient. A single-language Go codebase is the only realistic path for a solo builder.

3. **Deployment plane compatibility:**
   - Plane 0 (air-gapped, 1 GB RAM device): Cannot run JVM. Can run Go binary.
   - Plane 1 Edge (Raspberry Pi-class hardware, 4 GB minimum RAM): Could barely run JVM but leaves no room for other services. Go binary leaves 3.8 GB for other processes.
   - Plane 2+ (server class): JVM would work but adds unnecessary operational overhead.

4. **FHIR R5 simplification:** ADR-005 (FHIR R5 over R4) is a prerequisite enabler. FHIR R5 simplified several complex resource structures (e.g., Medication, Immunization, Event resources) compared to R4. A custom server for R5 is significantly less work than a custom server for R4 would have been.

5. **Go FHIR ecosystem maturity:** While no Go FHIR server matches HAPI's completeness, the ecosystem has matured. Libraries like `fhir-go` (resource definitions), `go-fhir` (base types), and `samply/blazectl` (FHIR operations) provide sufficient foundation. The G2A Engine generates FHIR resources from standards — the FHIR server primarily needs to store, validate, and serve these resources, not handle every FHIR edge case.

---

## 5. Consequences

**Positive:**

- FHIR server consumes <150 MB RAM, compatible with all deployment planes
- Single Go binary deploys in seconds — no JVM tuning, no classpath issues
- Full control over FHIR R5 implementation — can optimize for ZarishSphere 40-domain model
- Cross-compilation for ARM64 Raspberry Pi: `GOARCH=arm64 go build`
- Startup time <1 second, enabling rapid iteration and crash recovery
- No transitive dependency risk from HAPI's 200+ JAR dependencies

**Negative:**

- Must implement FHIR validation engine from scratch (resource validation, profile conformance, terminology binding)
- No built-in terminology server — must implement or integrate with external terminology services
- No FHIR bulk data API (patient-everything, group-export) without custom implementation
- Must maintain compatibility with HL7 FHIR R5 specification changes
- Significant upfront development investment (estimated 4-6 months for basic server, 8-12 months for production readiness)
- Smaller testing community — bugs in edge-case FHIR behavior may surface in production

---

## 6. Status

Accepted. This decision is mandated by the 8 GB RAM constraint and is referenced by Constitution Law 11 implementation. ADR-001 (Go as primary language) and ADR-005 (FHIR R5) are prerequisites.

---

## 7. Cross-references

→ **008-zs-adrs/001-adr-go-as-primary-language.md** — ADR-001: Go as the primary backend language
→ **008-zs-adrs/005-adr-fhir-r5-over-r4.md** — ADR-005: FHIR R5 as the canonical version (prerequisite)
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 5 (zero cost), Law 8 (plane sovereignty), Law 9 (vendor freedom), Law 11 (8 GB RAM constraint)
→ **001-zs-meta/003-zarishsphere-founder-profile.md** — Founder skill profile (§2.4, §4.1)
→ **007-zs-tech-stack/002-go-fhir-server.md** — Go FHIR server architecture and libraries

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
