# 003-deployment-planes.md
## Deployment planes
### Plane 0 through Plane 4 specifications

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Plane model overview](#1-plane-model-overview)
2. [Plane 0: air-gapped](#2-plane-0-air-gapped)
3. [Plane 1: edge](#3-plane-1-edge)
4. [Plane 2: district](#4-plane-2-district)
5. [Plane 3: national](#5-plane-3-national)
6. [Plane 4: global SaaS](#6-plane-4-global-saas)
7. [Plane cross-cutting concerns](#7-plane-cross-cutting-concerns)
8. [Cross-references](#8-cross-references)

---

## 1. Plane model overview

The ZarishSphere Platform deploys across five named infrastructure planes. No plane is first-class. All are equal citizens of the architecture. The Plane 0 constraint drives every technology decision: if a feature cannot work on a 4 GB air-gapped device, it is not a ZarishSphere feature.

| Plane | Name | Connectivity | RAM target | Deployer manages |
|---|---|---|---|---|
| 0 | Air-gapped | None | 4 GB | Everything |
| 1 | Edge | Occasional | 8 GB | OS + apps + data |
| 2 | District | Periodic | 16 GB | Apps + data |
| 3 | National | Persistent | 64+ GB | Config + data |
| 4 | Global SaaS | Always-on | Elastic | Data only |

## 2. Plane 0: air-gapped

### 2.1 Description

A single device with zero internet connectivity. No external dependencies of any kind. The primary use case is a tent clinic in a disaster zone, a mobile health unit in a conflict area, or any environment where connectivity is impossible.

### 2.2 Hardware minimum

- 4 GB RAM
- 64 GB storage
- Any architecture (x86_64 or ARM64)
- No GPU required

### 2.3 What runs

- Go-native API server (standalone binary)
- Embedded SQLite or minimal PostgreSQL
- Offline PWA served from localhost
- USB/QR code sync for data export
- All forms work fully offline
- All data collection functions work without modification

### 2.4 What does not run

- AI models (optional — disabled at Plane 0)
- Cloudflare Tunnel (no internet)
- NATS cluster (single-node NATS for local queue)
- Keycloak (basic local auth instead)

### 2.5 Data sync mechanism

Data leaves Plane 0 via encrypted USB bundle or QR code sequence. The deployer exports a sync package, carries it to a Plane 1+ node, and imports.

## 3. Plane 1: edge

### 3.1 Description

A device with occasional internet connectivity. Primary target is a Raspberry Pi 5 (8 GB) in a camp clinic, community health post, or field office. This is the primary deployment context for Cox's Bazar.

### 3.2 Reference hardware

| Component | Specification |
|---|---|
| Board | Raspberry Pi 5 |
| RAM | 8 GB |
| Storage | 256 GB NVMe (via hat) or 128 GB SD |
| Power | 5V 5A USB-C (solar or battery optional) |
| Total cost | ~USD 209 |

### 3.3 What runs

- All 13 ecosystem components in Docker Compose
- Go FHIR R5 server (~80 MB RAM)
- G2A Engine (Python + Go, ~200 MB)
- Ollama (Llama 3.1 8B Q4_K_M, ~4.7 GB)
- NLLB-200 translation (on-demand, ~3 GB)
- PostgreSQL 18.4 (~200 MB)
- NATS JetStream (~20 MB)
- Cloudflare Tunnel (~20 MB) for secure remote access
- All forms and apps

### 3.4 Sync

Automatic NATS JetStream sync when connectivity is available. Cloudflare Tunnel provides secure remote management.

## 4. Plane 2: district

### 4.1 Description

A server at a district health office, humanitarian hub, or sub-national administrative centre. Has intermittent connectivity. Supports multiple Plane 1 nodes syncing to it.

### 4.2 Hardware minimum

- 16 GB RAM
- 500 GB storage
- x86_64 or ARM64 server

### 4.3 What runs

- Docker Compose or K3s
- Full platform stack including Console
- Keycloak for user management
- Portainer GUI for management
- NATS JetStream intermediate sync hub
- Grafana dashboards

### 4.4 Sync hub

Plane 2 acts as a sync hub for multiple Plane 1 nodes. Data flows:
- Plane 1 → Plane 2 (encrypted sync via NATS)
- Plane 2 → Plane 3 (aggregated data)
- Plane 2 → Plane 1 (configuration updates, new forms)

## 5. Plane 3: national

### 5.1 Description

A self-hosted cloud for a national health system, large humanitarian programme, or government ministry. Persistent connectivity. Multi-node cluster.

### 5.2 Hardware minimum

- 64+ GB RAM across cluster
- 2+ TB storage
- K3s or full K8s cluster

### 5.3 What runs

- Full platform with all modules
- DHIS2 sync (aggregate only for FDMN-protected populations)
- Donor portal
- National user management
- All monitoring and alerting

### 5.4 DHIS2 integration

Aggregate indicators sync to national DHIS2. Individual records remain at Plane 2 or below. FDMN individual records never sync to Plane 3 or government-accessible DHIS2.

## 6. Plane 4: global SaaS

### 6.1 Description

A fully managed, multi-region, high-availability instance of the entire platform. Run by the Foundation on free-tier infrastructure. Deployers use it as a service — they bring only their configuration and data.

### 6.2 Infrastructure

| Service | Provider | Tier |
|---|---|---|
| Compute | Railway | Free tier credits |
| CDN | Cloudflare | Free tier (unlimited) |
| Database | Railway PostgreSQL or Supabase free | Free tier |
| Cache | Railway Valkey | Free tier |
| Storage | Cloudflare R2 | Free tier (10 GB) |
| CI/CD | GitHub Actions | Free tier (2,000 min/month) |

### 6.3 What deployers own

Deployers own their configuration and data. The Foundation operates the infrastructure. No deployer data is used for training, analytics, or any purpose beyond serving the deployer.

## 7. Plane cross-cutting concerns

| Concern | How it works across planes |
|---|---|
| Identity | Plane 0: local auth. Plane 1-2: Keycloak. Plane 3-4: Keycloak + SSO. All planes support offline auth. |
| Audit | Hash-chain audit log on every plane. Syncs upward when connectivity permits. |
| Forms | Same form definitions run on all planes. Offline forms queue data for sync. |
| Modules | Same modules deploy on all planes. Plane 0 modules exclude AI-dependent features. |
| Updates | Plane 0: USB bundle. Plane 1-2: NATS push. Plane 3-4: Argo CD sync. |

## 8. Cross-references

→ **003-zs-platform/001-platform-overview.md** — Five-plane deployment model summary and platform technology stack
→ **003-zs-platform/010-free-tier-resource-map.md** — Every zero-cost service the planes are built from, with capacity limits
→ **003-zs-platform/007-data-model.md** — Data routing rules and FDMN protection boundaries across planes
→ **007-zs-tech-stack/001-tech-stack-master.md** — Version-pinned technology inventory for all planes
→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: the zero-cost toolchain obligation behind Plane 4 infrastructure
→ **009-zs-operations/005-sop-deployment-checklist.md** — Pre-deployment verification steps for each plane

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
