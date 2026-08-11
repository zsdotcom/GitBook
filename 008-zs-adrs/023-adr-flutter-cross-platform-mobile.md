# 023-adr-flutter-cross-platform-mobile.md
## ADR-023: Flutter 3.44.7 (Dart 3.12.2) for Cross-Platform Mobile Applications
### Single Codebase for Android and iOS, Offline-First with PowerSync, Riverpod State Management

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

Use Flutter 3.44.7 (Dart 3.12.2) as the cross-platform mobile application framework for all ZarishSphere Platform mobile applications. All mobile apps — Community Health Worker (CHW) app, Clinician app, Supervisor app, and Patient Portal — share a single Flutter codebase targeting both Android and iOS. PowerSync Flutter SDK provides offline-first database synchronization to the PostgreSQL backend. Riverpod 3.4.2 provides testable, null-safe state management. The mobile UI adapts the Carbon Design System (ADR-019) design language for mobile form factors while maintaining visual consistency with the web Console and web Forms engine.

---

## 2. Context

The ZarishSphere Platform requires mobile applications for several distinct user groups:

- **Community Health Workers (CHWs):** Conduct household visits in rural areas. Register patients, record vitals, assess symptoms, administer treatments, update pregnancy records, and refer cases. These users have varying levels of digital literacy and operate primarily offline. Android is the dominant mobile platform in target deployment countries (Bangladesh, Uganda, Kenya, Indonesia, Ethiopia).
- **Clinicians:** Rural health post and district hospital clinicians. Access patient records, record encounters, prescribe medications, view lab results. May use either Android or iOS depending on the organization's device procurement. Need offline access to patient data during network outages.
- **Supervisors / Program Managers:** Field monitoring, data quality review, aggregate reporting. May use tablets (Android/iOS) for field visits. Need offline access to dashboards and patient lists.
- **Patients:** Personal health record access, appointment scheduling, medication reminders, health education content. May use their own Android or iOS devices with limited storage and older OS versions.

Key constraints:

- **Offline-first:** All mobile applications must work fully offline as the primary mode of operation. Synchronization with the server occurs when connectivity is available. This is required by Constitution Law 8 (Deployment plane sovereignty) and the realities of the target deployment contexts.
- **Single codebase:** With a single founder building the entire platform, maintaining separate iOS and Android codebases is infeasible. Cross-platform development is a structural necessity, not a convenience.
- **Android dominance in target markets:** Market share in target countries is 85-95% Android. iPhone penetration is low. However, iOS support is still required for organizations that issue iPhones to their staff and for patient-facing apps on the App Store.
- **Low-end device support:** Target devices include Android 10+ devices with 2 GB RAM and limited storage (16-32 GB). The mobile framework must perform well on low-end hardware — 90fps rendering, small binary size, and efficient memory usage.
- **Clinical data integrity:** Mobile applications handle sensitive health data. Local data storage must be encrypted, and sync must ensure data integrity.
- **Zero-cost toolchain:** Flutter is free and open-source. Dart is BSD-3 licensed (permissive). No paid SDK licenses, no pro-tier features for essential functionality — consistent with ADR-006.

---

## 3. Alternatives Considered

- **Flutter 3.44.7 + Dart 3.12.2 (chosen):** Single codebase compiles to native ARM code for both Android (APK/AAB) and iOS (IPA). Dart 3.12.2 with sound null safety eliminates null reference errors at compile time. Flutter's Impeller rendering engine provides 90fps performance on low-end Android devices. The Flutter binary size (~15-20 MB for a production app) is acceptable for target markets. Riverpod 3.4.2 provides compile-time safe, testable dependency injection and state management. Flutter's widget system enables custom UI that can match the Carbon Design System (ADR-019) visual language. The Flutter SDK is free, BSD-3 licensed, and maintained by Google with quarterly stable releases. Dart FFI enables native performance for compute-heavy operations.

- **React Native 0.79:** JavaScript/TypeScript-based cross-platform framework. Reuses React knowledge from the web frontend (ADR-022: TypeScript strict mode). Larger community than Flutter, more third-party packages, and native module ecosystem. However: React Native runs JavaScript in a bridge (Hermes engine) — the JavaScript bridge adds latency for UI updates and complex state synchronization. Performance on low-end Android devices (2 GB RAM) is noticeably worse than Flutter's compiled native code. React Native's layout system (Yoga) is a subset of CSS — layouts that work on web may not work on mobile. Native module fragmentation (Android-specific and iOS-specific implementations for the same functionality) increases maintenance burden for a single founder. Over-the-air updates (CodePush) require Microsoft App Center (deprecated) or self-hosted solutions.

- **Kotlin Multiplatform (KMP) + Compose Multiplatform:** Kotlin-based cross-platform framework using Jetpack Compose for UI. Kotlin is a modern JVM language with excellent tooling. Compose Multiplatform allows sharing UI code across Android, iOS, and Desktop. However: KMP is significantly less mature than Flutter — the ecosystem is smaller, fewer third-party packages exist, and the iOS implementation (Compose for iOS) is still in alpha/beta. Kotlin is JVM-based — building and debugging requires the JVM runtime (conflict with ADR-004's general JVM avoidance principle, though the JVM is only used during development here, not in production on device). The single founder would need to maintain expertise in a fourth language (TypeScript, Go, Dart, Kotlin). Smaller community means fewer answered questions and less shared knowledge.

- **Native Android (Kotlin/Jetpack Compose) + Native iOS (SwiftUI):** Full native development for each platform — best performance, best platform integration, full access to all native APIs. However: requires maintaining two completely separate codebases — 2× development effort for every feature. For a single-founder project, this is structurally infeasible. Bug fixes, feature additions, and UI changes must be implemented twice. The Constitutional mandate (Law 7 — Module sovereignty) extends to development efficiency: the platform must be buildable by a single person.

- **PWA (Progressive Web Application):** Web-based mobile application running in the browser. No app store distribution, no platform-specific SDKs, uses web technologies (React, TypeScript) that share the web frontend codebase. However: PWAs have limited access to native device features (camera for barcode scanning, Bluetooth for medical devices, file system for offline attachments). Offline support requires service workers with limited storage (quota varies by browser). Push notifications on iOS are unreliable (iOS does not support the Web Push API). PWAs cannot be distributed through app stores in a way that most users in target markets are accustomed to. For the web-based interfaces (Console, Builder, Marketplace), the existing Next.js application already serves as a responsive PWA. For field-worker mobile apps, Flutter provides a superior offline experience.

---

## 4. Reason for Decision

1. **Offline-first architecture:** Flutter's local-first architecture (all data persisted locally in SQLite via drift/PowerSync) aligns perfectly with the Platform's offline-first requirements. The Impeller rendering engine renders locally without network dependency. Flutter's widget tree initialization does not require server interaction — the app is usable from splash screen to full functionality without connectivity.

2. **Performance on low-end devices:** Flutter compiles to native ARM code via Dart's ahead-of-time (AOT) compiler. The Impeller rendering engine (default on Flutter 3.44.7) provides smooth 90fps rendering on Android devices with 2 GB RAM. This is essential for target deployment contexts where devices may be several years old with limited hardware.

3. **Single codebase, single founder:** One Flutter codebase generates both Android (APK/AAB) and iOS (IPA) builds. The single founder writes UI and business logic once, and it runs on both platforms. Platform-specific code (camera access, file storage, biometric auth) is isolated behind platform channels with well-documented APIs.

4. **Dart sound null safety:** Dart 3.12.2 with sound null safety eliminates null reference errors at compile time — the same class of bug that TypeScript strict mode (ADR-022) eliminates for web frontend code. This is critical for clinical mobile software where a null pointer crash can occur during a patient consultation.

5. **Riverpod 3.4.2 for state management:** Riverpod 3.4.2 provides compile-time safe dependency injection and state management. Dependencies are resolved at compile time, not runtime — eliminating a class of runtime errors common in other state management solutions. Riverpod's testability (each provider can be overridden in tests) supports the single-founder testing workflow without a dedicated QA team.

6. **PowerSync integration:** Flutter's drift SQLite ORM integrates natively with PowerSync (ADR-021) for offline database synchronization. The `powersync` Flutter package provides a drop-in sync client that connects the local SQLite database to the PostgreSQL backend. This integration makes offline-first mobile development a configuration concern rather than a multi-month engineering effort.

7. **Zero-cost compliance:** Flutter is free, BSD-3 licensed. Dart is free, BSD-3 licensed. No paid SDK features, no pro-tier restrictions, no usage-based pricing. Google has committed to long-term support with quarterly stable releases. Full ADR-006 compliance.

---

## 5. Consequences

**Positive:**

- Single codebase for Android and iOS — 1× development effort instead of 2×
- Native ARM performance on low-end Android devices (2 GB RAM, Android 10+)
- Dart sound null safety eliminates null reference crashes at compile time
- Riverpod provides testable, compile-time-safe state management
- PowerSync Flutter SDK provides ready offline sync integration
- Impeller rendering engine provides 90fps UI performance
- Flutter's widget system enables Carbon Design System visual language adaptation
- Flutter's hot reload enables rapid development iteration — critical for single-founder productivity
- Hot restart preserves state across code changes — faster debugging cycle

**Negative:**

- Dart is a niche language — fewer developers know Dart compared to JavaScript, TypeScript, Kotlin, or Swift
- Flutter apps are larger than native apps (~15-20 MB vs ~5-10 MB for a simple native app)
- Some platform-specific features require native module development (platform channels) — adds complexity for camera, Bluetooth, biometrics integration
- Flutter's widget system is different from React — Carbon Design System components must be adapted or rebuilt for Flutter (no direct Carbon Flutter library)
- iOS support, while functional, is less polished than Android — some iOS-specific features (Apple Health integration, widgets, Apple Watch) require additional work
- App Store review process for Flutter apps can flag some Flutter-specific patterns (e.g., use of reflection in some packages)
- Flutter's ecosystem is smaller than React Native's — fewer third-party packages, less community content, fewer answered questions
- The single founder must maintain proficiency in four languages: Go (backend), TypeScript (web frontend), Dart (mobile), and configuration/doc in YAML/Markdown

---

## 6. Status

Accepted. Flutter 3.44.7 (Dart 3.12.2) is the mobile application framework for all ZarishSphere Platform mobile apps. All mobile applications share a single Flutter codebase targeting Android (minSdk 26 / Android 10+) and iOS (min iOS 16+). Flutter is used exclusively for mobile applications — the web frontend remains React/Next.js/TypeScript. New mobile features must be implemented in Flutter and must work offline as the primary mode of operation.

**Verification:** last_verified August 01, 2026 · next_review February 01, 2027 · verified_by ZarishSphere Foundation

---

## 7. Cross-references

→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: structural requirement for open-source, zero-cost toolchain dependencies
→ **008-zs-adrs/019-adr-carbon-design-system.md** — ADR-019: IBM Carbon Design System as the UI component library
→ **008-zs-adrs/021-adr-powersync-mobile-offline.md** — ADR-021: PowerSync self-hosted offline database synchronization engine
→ **008-zs-adrs/022-adr-typescript-strict-mode.md** — ADR-022: TypeScript strict mode for web frontend code
→ **010-zs-ecosystem/013-system-spec.md** — ZarishSphere System component specification (IAM, encryption, configuration, monitoring, backup, audit infrastructure)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
