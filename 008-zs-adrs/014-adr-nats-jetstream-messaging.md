# 014-adr-nats-jetstream-messaging.md
## ADR-014: NATS JetStream as Async Messaging Backbone
### Single 20MB Binary, Persistent Streams, FHIR R5 Topic-Based Subscriptions

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

Use NATS JetStream 2.14.4 as the asynchronous messaging backbone for the ZarishSphere Platform. All service-to-service event communication, FHIR R5 subscription notifications, audit event streaming, and domain module inter-process messaging flow through NATS subjects with JetStream persistence. The NATS binary (~20 MB) runs alongside services on all deployment planes, including Plane 1 (Raspberry Pi).

---

## 2. Context

The ZarishSphere Platform services need asynchronous communication for:

- FHIR R5 resource lifecycle events: when a Patient is created, updated, or deleted, downstream services must react (search index update, AuditEvent recording, subscription notification to external systems)
- Cross-module events: a module in the health domain may need to notify the logistics module about a supply consumption event without a synchronous HTTP dependency
- Platform events: deployment notifications, health check alerts, configuration change broadcasts
- Audit event streaming: all clinical and administrative actions must produce audit events that are durably stored and replayable — Constitution Law 10 (Every decision is auditable forever)
- At-least-once delivery for clinical events: no clinical event may be silently dropped, even if the consuming service is temporarily unavailable

The platform must meet these requirements within the constraints of Constitution Law 5 (zero cost) and ADR-006: the messaging system must be open-source, self-hosted, and runnable on a Raspberry Pi 5 for Plane 1 deployments. The system must also support Plane 0 (air-gapped) where the messaging backbone must work entirely offline.

FHIR R5 introduces a native topic-based subscription model where clients subscribe to resource type and event type patterns. The messaging backbone must map naturally to this model.

---

## 3. Alternatives Considered

- **NATS JetStream 2.14.4:** Single 20 MB binary with no runtime dependencies. JetStream provides disk-persistent streams with configurable retention policies. FHIR R5 topic subscriptions map directly to NATS subject hierarchies (e.g., `zs.fhir.Patient.created`). Supports at-least-once delivery with configurable acknowledgements. Dead letter queue support via `zs.dlq.*` subjects. Apache 2.0 licensed. Runs on Raspberry Pi with <50 MB RAM. Go-native client (nats.go) integrates naturally with the Go backend mandated by ADR-001.

- **Apache Kafka 4.0:** Industry-standard event streaming platform with massive ecosystem, exactly-once semantics, ksqlDB for stream processing, and Schema Registry for schema evolution. However: requires 500 MB+ minimum RAM for broker, requires JVM (conflicts with ADR-004's prohibition on JVM dependency), ZooKeeper/KRaft adds operational complexity, and is over-provisioned for the ZarishSphere message volume. The single founder cannot justify Kafka's operational overhead for what is primarily service-to-service event relay and FHIR subscription distribution.

- **RabbitMQ 4.x:** Mature AMQP-compliant message broker with flexible routing, dead letter exchanges, and management UI. However: requires Erlang runtime (~150 MB), clustering is complex, and it lacks the native subject hierarchy that maps to FHIR R5 topic subscriptions. RabbitMQ's queue-based model is a worse fit for FHIR's publish-subscribe patterns than NATS's subject-based model.

- **Redis Streams (via Valkey 9.1.1):** In-memory streams with optional persistence. Lightweight — runs in-process with our caching layer. However: Redis Streams lack the persistence guarantees required for clinical audit events (messages can be lost on crash before AOF sync), no native dead letter queue, no consumer group rebalancing, and message retention is secondary to Redis's primary caching role. Suitable for ephemeral events but not for the durable, replayable streams that audit compliance requires.

- **Google PubSub / Amazon SQS/SNS:** Fully managed, zero-ops, global scale. However: vendor lock-in (violates ADR-009 and Constitution Law 9), requires internet connectivity (violates Plane 0/1 offline requirements), cost at scale (violates ADR-006 zero-cost guarantee), and requires a credit card to use. Unsuitable for the deployment contexts ZarishSphere targets.

---

## 4. Reason for Decision

1. **Lightweight footprint:** NATS runs as a single 20 MB binary with ~50 MB RAM consumption. This is critical for Plane 1 (Raspberry Pi, 8 GB RAM shared across all services) and for the 8 GB development machine. Kafka would consume more RAM than all Go services combined.

2. **FHIR R5 subject mapping:** NATS's hierarchical subject model (`zs.fhir.{ResourceType}.{event}`) maps directly to FHIR R5's topic-based subscription model. A FHIR Subscription resource with `criteria = "Patient?name=Smith"` maps to a NATS consumer on `zs.fhir.Patient.*` with a specific filter. This natural mapping reduces translation complexity compared to RabbitMQ's exchange/routing-key model.

3. **JetStream persistence for audit compliance:** Constitution Law 10 requires that every platform decision and event be auditable forever. NATS JetStream streams persist messages to disk with configurable retention (by time, count, or size). This provides the durability required for audit event storage without a separate audit database.

4. **Dead letter queue for failed messages:** Clinical events that cannot be processed (e.g., a Patient creation that fails schema validation on a downstream service) must not be lost. NATS's dead letter queue pattern routes failed messages to `zs.dlq.*` subjects for manual inspection and replay.

5. **At-least-once delivery guarantee:** NATS JetStream's acknowledgement model ensures that messages are redelivered until the consumer explicitly acknowledges receipt. This is essential for clinical workflows where silent message loss could lead to missed patient care events.

6. **Zero-cost and vendor freedom:** NATS is Apache 2.0 licensed, self-hosted, and requires no third-party service, API key, or credit card. It runs fully offline, satisfying Plane 0 and Plane 1 requirements. ADR-009 (no vendor lock-in) is satisfied because NATS can be replaced with any compatible messaging system if needed.

---

## 5. Consequences

**Positive:**

- Single 20 MB binary runs on all deployment planes including Raspberry Pi
- FHIR R5 topic subscription model maps directly to NATS subject hierarchy
- JetStream persistence provides durable audit event storage without a separate database
- Dead letter queues ensure no clinical events are silently lost
- Go-native client (nats.go) integrates naturally with the Go backend
- At-least-once delivery guarantees satisfy clinical event reliability requirements
- Fully offline capable — no dependency on internet or cloud services
- Zero licensing cost — Apache 2.0

**Negative:**

- Smaller ecosystem than Kafka — fewer stream-processing tools, no ksqlDB equivalent
- NATS lacks built-in Schema Registry — must implement FHIR resource schema validation at the application layer
- Message ordering is per-subject, not global — cross-subject event ordering requires application-level sequencing
- NATS community less mature than Kafka's for large-scale deployments
- Requires NATS-specific knowledge for debugging — less common skill than RabbitMQ or Kafka
- JetStream management UI is less polished than Kafka's ecosystem tools

---

## 6. Status

Accepted. NATS JetStream is the messaging backbone for all ZarishSphere Platform asynchronous communication. All services, modules, and domain implementations must use NATS subjects following the `zs.{domain}.{resource}.{event}` naming convention. JetStream streams must be used for all durable event storage and FHIR subscription delivery.

**Verification:** last_verified August 01, 2026 · next_review February 01, 2027 · verified_by ZarishSphere Foundation

---

## 7. Cross-references

→ **008-zs-adrs/001-adr-go-as-primary-language.md** — ADR-001: Go as the primary backend language (nats.go client integration)
→ **008-zs-adrs/004-adr-no-hapi-fhir.md** — ADR-004: no JVM dependency (rules out Kafka)
→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: zero-cost toolchain structural requirement
→ **008-zs-adrs/009-adr-no-vendor-lock-in.md** — ADR-009: no vendor lock-in (rules out managed cloud brokers)
→ **005-zs-standards/012-api-asyncapi-conventions.md** — AsyncAPI 3.0 event conventions over NATS JetStream (subject naming, DLQ routing)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
