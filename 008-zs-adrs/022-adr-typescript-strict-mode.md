# 022-adr-typescript-strict-mode.md
## ADR-022: TypeScript 7.0.2 Strict Mode for All Frontend Code
### Strict Null Checks, No Implicit Any, Strict Function Types, Zero Tolerance for Untyped Clinical UI Code

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

All ZarishSphere Platform frontend code must use TypeScript 7.0.2 with strict mode enabled. The `tsconfig.json` for every frontend package — microfrontends, shared libraries, ecosystem applications (Console, Marketplace, Builder), and domain apps — must include `"strict": true`. This enables strict null checks, no implicit any, strict function types, and exact optional property types. TypeScript is the canonical language for all frontend code. No plain JavaScript files are permitted in production code. Runtime validation of external data (API responses, user input, FHIR resource payloads) uses Zod 4.4.3 for runtime type validation, ensuring that compile-time type guarantees extend to runtime boundaries.

---

## 2. Context

The ZarishSphere Platform frontend spans 40+ domain microfrontends, shared utility packages, and ecosystem applications — all handling clinical and administrative data where type errors can have real-world consequences:

- A `null` check missed in React component rendering can crash a patient registration form — a clinician loses their work
- An `any` type propagating through a data pipeline can silently transform a patient's medication dosage into an unexpected format
- An untyped function parameter in a FHIR resource builder can accept invalid resources, leading to data corruption in the PostgreSQL database
- An implicit `any` in a sync conflict resolver can incorrectly merge patient records, creating duplicate medical histories

Key requirements:

- **Clinical software safety:** Frontend type safety is not a developer convenience — it is a clinical safety requirement. TypeScript strict mode catches entire categories of runtime errors at compile time, before they reach a clinician or patient.
- **Multi-developer readiness:** Though currently a single-founder project, the architecture must support future contributions from developers with varying levels of TypeScript expertise. Strict mode enforces a consistent baseline that prevents untyped code from entering the codebase.
- **FHIR R5 type definitions:** FHIR R5 resources have complex, deeply nested type structures with optional fields, choice types, and constrained value sets. Strict TypeScript ensures that FHIR resource builders and consumers correctly handle this complexity.
- **Microfrontend boundary safety:** In the microfrontend architecture (ADR-020), modules expose typed interfaces (props, shared state, events). Strict type checking at these boundaries prevents cross-module errors that would otherwise manifest as runtime failures.
- **Compiler upgrade path:** TypeScript 7.0.2 is the current stable release with full support for strict mode, decorators, and improved type inference. Version 7.0.2 provides the best balance of type checking power and compilation performance for the existing codebase.

**Amendment (August 01, 2026):** TypeScript 7.0.2 is a Go-native implementation. The original rationale that no native, high-performance TypeScript tooling existed is obsolete, but the strict-mode decision stands: the Go-native compiler improves compilation performance without changing the language, and strict mode remains mandatory for all frontend code.

---

## 3. Alternatives Considered

- **TypeScript 7.0.2 strict mode (chosen):** Full strict mode with `noImplicitAny`, `strictNullChecks`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`, `noImplicitThis`, `alwaysStrict`, and `exactOptionalPropertyTypes` enabled. Catches null reference errors, untyped parameters, incorrect function assignments, and implicit any propagation at compile time. TypeScript 7.0.2's improved type narrowing and `using` declarations provide additional safety for resource lifecycle management. The compiler itself is mature and stable — TypeScript 7.0.2 is the Go-native implementation, now in production use, resolving the original compiler-performance concern without changing the language.

- **TypeScript without strict mode:** TypeScript with default (non-strict) settings — many of the type safety features disabled or set to warning-only. Allows faster initial development, easier adoption by JavaScript developers, and fewer compilation errors during rapid prototyping. However: the runtime failures that strict mode prevents (null reference errors in particular) are the most common source of production incidents in clinical web applications. Without strict null checks, `string | undefined` is treated as `string` — every reference to a potentially undefined value is a latent production crash. For clinical software where crashes mean disrupted patient care, this is unacceptable.

- **JavaScript (plain, no TypeScript):** No type system — fastest development iteration, no compilation step, no type annotation maintenance. Suitable for prototyping but unacceptable for production clinical software. Large-scale JavaScript applications require extensive test coverage to catch type errors that TypeScript would catch at compile time. With a single founder and limited testing bandwidth, compile-time type checking is the primary safety net. No plain JavaScript is permitted in ZarishSphere frontend production code.

- **Flow (Facebook's type checker):** Static type checker for JavaScript with a different type system philosophy than TypeScript. However: Flow has been effectively abandoned by Facebook/Meta — no significant development since 2021, declining community, fewer third-party type definitions, and limited editor support. Choosing Flow would be a dead-end decision that would require migration to TypeScript within 1-2 years. The ecosystem has overwhelmingly standardized on TypeScript.

- **ReasonML / ReScript:** Strongly typed language that compiles to JavaScript. Sound type system with full type inference — no `any` escape hatch. However: extremely small ecosystem, minimal React integration (no support for React 19.3.0 features), no FHIR type definitions, and very difficult hiring path. Learning ReScript adds cognitive overhead for the single founder who would also need to maintain Go (backend) and Dart (Flutter mobile) proficiency. Not practical for a multi-language solo development context.

---

## 4. Reason for Decision

1. **Clinical safety through compile-time checking:** TypeScript strict mode prevents null reference errors, undefined property access, and type confusion at compile time — before code reaches production. In clinical software, where a null pointer exception can crash a patient registration form during a consultation, this compile-time safety is a clinical safety requirement, not a developer convenience.

2. **FHIR R5 type complexity:** FHIR R5 resources have deeply nested optional fields, choice types (e.g., `value[x]` can be `string`, `Quantity`, `CodeableConcept`, `Reference`, etc.), and constrained value sets. TypeScript strict mode with discriminated unions, template literal types, and branded types enables precise modeling of these structures, catching invalid resource construction at compile time.

3. **Microfrontend boundary contracts:** In the microfrontend architecture (ADR-020), modules communicate through typed interfaces — component props, custom events, and shared state. Strict TypeScript ensures that these interfaces are correctly implemented on both sides of the boundary, preventing the class of bugs where a module passes an incorrectly shaped object to another module.

4. **Single-founder safety net:** With no dedicated QA team, no code review from senior engineers, and no automated end-to-end testing infrastructure, compile-time type checking is the primary safety net. Strict mode catches more bugs at compile time than any other single practice available to a solo developer.

5. **Industry standard with long-term viability:** TypeScript is the dominant typed language for frontend development, backed by Microsoft, with the largest ecosystem of type definitions (DefinitelyTyped), comprehensive editor support (VS Code), and broad community adoption. TypeScript 7.0.2's Go-native compiler improves compilation performance without changing the language. Constitution Law 11 (The platform outlives its creators) is served by choosing the most sustainable, widely-adopted technology.

6. **Zod 4.4.3 for runtime boundary validation:** While TypeScript provides compile-time type safety, runtime data (API responses, user input, FHIR resources from external systems) must be validated at runtime. Zod 4.4.3 schemas are defined alongside TypeScript types, generating both the compile-time type and the runtime validator from a single source of truth. This pattern (TypeScript for internal code, Zod for external boundaries) provides defense in depth.

---

## 5. Consequences

**Positive:**

- Null reference errors caught at compile time — the most common class of frontend production failures is eliminated
- Untyped API responses validated at runtime via Zod — no invisible data corruption
- FHIR R5 resource construction guided by precise type definitions — fewer invalid resources created
- Microfrontend boundary contracts enforced by the type system — cross-module bugs reduced
- Explicit `unknown` instead of `any` forces deliberate type narrowing
- Consistent code quality baseline across all frontend packages
- Better IDE support (autocomplete, refactoring, inline documentation)
- Self-documenting code — types serve as living documentation of data structures

**Negative:**

- Initial development is slower — strict mode requires explicit type annotations and narrows types more aggressively
- Some third-party libraries lack type definitions — require custom type declarations or wrapper modules
- Complex FHIR type definitions (deeply nested choice types, recursive structures) require sophisticated TypeScript patterns (discriminated unions, template literal types, recursive type aliases)
- Learning curve for contributors unfamiliar with TypeScript strict mode — common JavaScript patterns (implicit any, loose null handling) are rejected
- Compilation errors in strict mode can be cryptic — understanding and fixing them requires TypeScript expertise
- Migration of any existing non-strict TypeScript code requires fixing all strict mode violations — a potentially significant effort
- TypeScript compilation adds to build time, though the Go-native compiler in TypeScript 7.0.2 reduces this overhead

---

## 6. Status

Accepted. TypeScript 7.0.2 with strict mode (`"strict": true`) is required for all ZarishSphere Platform frontend code. Every frontend package must have `strict: true` in its `tsconfig.json`. No `any` types are permitted in production code — use `unknown` with type narrowing. All API boundaries must be validated with Zod 4.4.3 schemas. New packages must be created with strict mode from day one. Existing packages must be migrated to strict mode before being promoted to stable.

**Verification:** last_verified August 01, 2026 · next_review February 01, 2027 · verified_by ZarishSphere Foundation

---

## 7. Cross-references

→ **008-zs-adrs/020-adr-microfrontend-architecture.md** — ADR-020: microfrontend architecture with typed interfaces (props, shared state, events) across module boundaries

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
