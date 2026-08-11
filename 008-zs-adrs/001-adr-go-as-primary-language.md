# 001-adr-go-as-primary-language.md
## ADR-001: Selection of Go as the Primary Engine Language
### Go 1.26.5 for all backend services, FHIR server, and G2A Engine

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

Use Go 1.26.5 as the sole language for all backend services, including the FHIR server, G2A Engine, API gateway, module runtime, and all platform-level infrastructure. No other language will be used for server-side components. Frontend remains JavaScript/TypeScript (React 19.2.8, Next.js 16.2.12).

---

## 2. Context

ZarishSphere Platform requires a backend language capable of:

- Running within an 8 GB RAM budget (Constitution Law 11 — platform outlives its creators)
- Producing single-binary deployments for Plane 0 (air-gapped) and Plane 1 (Raspberry Pi) without JVM or runtime dependencies
- Supporting FHIR R5 resource modeling and validation
- Serving as the runtime language for the G2A Engine (Guideline-to-Action transforms)
- Being maintainable by a single founder with no dedicated backend engineering team
- Compiling quickly on a Lenovo i3 / 8 GB RAM development machine

Five candidate languages were evaluated: Go, Rust, Node.js (JavaScript/TypeScript), Python, and Java (JVM).

---

## 3. Alternatives Considered

| Alternative | Pros | Cons |
|---|---|---|
| **Go 1.26.5** | Single binary deploy (~15 MB), low RAM footprint (~50-150 MB per service), fast compilation (seconds), excellent standard library, good FHIR libraries (fhir-go, go-fhir), built-in concurrency via goroutines, static typing, fast-growing ecosystem | Smaller third-party ecosystem than Java or Node.js; fewer enterprise FHIR tools |
| **Rust** | Zero-cost abstractions, memory safety without GC, smallest binary sizes (~5-10 MB), ideal for embedded/Plane 0 | Steep learning curve (ownership model); single founder cannot afford ramp-up time; slower compile times on i3 hardware; fewer FHIR/health libraries |
| **Node.js (TypeScript)** | Familiarity if frontend is also JS/TS, vast npm ecosystem, good for I/O-bound workloads | Single-threaded event loop unsuitable for compute-heavy G2A transforms; runtime requires Node.js installed (not single binary); memory overhead per process (~50-100 MB base); dependency hell with node_modules |
| **Python** | Largest ML/data ecosystem, easiest to learn, excellent for prototyping | Slow execution for production workloads; no static typing (without mypy overhead); GIL limits concurrency; requires Python runtime + virtualenv for deployment; not suitable for FHIR server performance requirements |
| **Java / JVM (with HAPI FHIR)** | Dominant FHIR ecosystem (HAPI FHIR is the reference implementation), vast enterprise adoption, mature JIT compilation | JVM baseline RAM: 2-4 GB before any application logic; constitutionally incompatible with 8 GB RAM constraint (Law 11); JAR deployment model (~200-500 MB); slow startup times; single founder cannot maintain Java AND Go codebases; blocked by Law 5 (zero cost) in practice for some JVM tooling |

---

## 4. Reason for Decision

1. **RAM efficiency:** Go services run at 50-150 MB resident memory. The same functionality in Java requires 2-4 GB for the JVM alone. With 8 GB total on the development machine and a target of running multiple services (FHIR server, G2A Engine, API gateway, identity service) simultaneously, Java is structurally infeasible.

2. **Single binary deployment:** Go compiles to a statically linked binary with no runtime dependencies. This is critical for Plane 0 (air-gapped, no package manager) and Plane 1 (Raspberry Pi, ARM architecture). A single `scp` of a Go binary deploys a service. No JRE, no Python interpreter, no Node runtime required.

3. **Compilation speed:** Full builds of the Go codebase complete in under 30 seconds on the Lenovo i3 development machine. Rust compilation on the same hardware would take 5-15 minutes for comparably sized projects, significantly slowing the development cycle for a solo developer.

4. **FHIR ecosystem viability:** While Go lacks a HAPI-equivalent monolithic FHIR server, the combination of fhir-go, go-fhir, and custom resource definitions provides sufficient foundation. Given ADR-005 (FHIR R5 only), R5's simplified resource model reduces the complexity of building a custom server — making Go a viable choice where it would not have been for R4 with its extensive resource surface.

5. **Founder fit:** The founder (Mohammad Ariful Islam) has reading-level familiarity with Go (per founder profile §4.1). Go's simplicity and minimal syntax reduce cognitive load compared to Rust's ownership model or Java's annotation-heavy ecosystem. Go documentation is excellent, and the community is responsive.

6. **Constitution alignment:** Law 11 (platform outlives its creators) requires that the platform remain deployable without any individual's participation. Go binaries compiled for linux/amd64 and linux/arm64 can be distributed as static artifacts — no language runtime evolution can break the platform years later.

---

## 5. Consequences

**Positive:**

- Every backend service consumes <200 MB RAM, allowing multiple services on 8 GB machine
- Deployment is `scp binary && ./binary` — no containers required for Plane 0/1
- Cross-compilation for ARM (Raspberry Pi) is a single `GOARCH=arm64` flag
- Fast iteration cycle improves solo developer productivity
- Go's built-in profiling (pprof, trace) provides production observability without third-party tools

**Negative:**

- No plug-and-play FHIR server exists for Go — must build custom FHIR R5 server from scratch (see ADR-004)
- Go's type system is less expressive than Rust or Haskell for domain modeling
- Smaller health-IT ecosystem means more foundational work (terminology server, validation engine)
- Hiring Go developers in humanitarian/global health sector is harder than Java or Python

---

## 6. Status

Accepted. This decision is foundational — all subsequent ADRs (especially ADR-004, ADR-005, ADR-006) build upon it.

---

## 7. Cross-references

→ **008-zs-adrs/004-adr-no-hapi-fhir.md** — ADR-004: custom Go-native FHIR server instead of HAPI FHIR
→ **008-zs-adrs/005-adr-fhir-r5-over-r4.md** — ADR-005: FHIR R5 as the canonical FHIR version
→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: zero-cost toolchain structural requirement
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 5 (zero cost), Law 9 (vendor freedom), Law 11 (platform outlives its creators)
→ **001-zs-meta/003-zarishsphere-founder-profile.md** — Founder hardware and skill profile (§2.4, §4.1)
→ **007-zs-tech-stack/001-tech-stack-master.md** — Tech stack master record (Go as primary language)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
