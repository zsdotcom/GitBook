# 011-distributions-spec.md
## ZarishSphere Distributions specification
### Pre-packaged deployment bundles

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Available distributions](#2-available-distributions)
3. [Distribution structure](#3-distribution-structure)
4. [One-click deployment](#4-one-click-deployment)
5. [Distribution lifecycle](#5-distribution-lifecycle)
6. [Plane-specific optimizations](#6-plane-specific-optimizations)
7. [Version pinning and signing](#7-version-pinning-and-signing)
8. [Update channel](#8-update-channel)
9. [Requirements and documentation](#9-requirements-and-documentation)
10. [Cross-references](#10-cross-references)

---

## 1. Purpose

ZarishSphere Distributions are pre-packaged, pre-configured, pre-tested deployment bundles for specific use cases. They let users deploy a complete, working solution for their context without assembling components manually.

## 2. Available distributions

| Distribution | Contents | Target plane | Use case |
|---|---|---|---|
| Air-gapped clinic | Single module + form engine + offline sync | Plane 0 | Tent clinic, no internet |
| Edge health post | Health module + AI (optional) + PWA | Plane 1 | Camp clinic, RPi |
| District health office | Core modules + console + reporting | Plane 2 | Sub-national office |
| National program | All modules + marketplace + analytics | Plane 3 | Ministry of Health |
| Global SaaS | Full ecosystem, multi-tenant | Plane 4 | Foundation-hosted |

### Planned distributions

| Distribution | Contents | Target plane | Status |
|---|---|---|---|
| Primary Health Care | Patient registry, NCD tracker, immunization, nutrition | Planes 0-2 | In development |
| Maternal & Child Health | ANC/PNC tracking, delivery registry, growth monitoring | Planes 0-2 | In development |
| Nutrition & WASH | Screening, supplementation, water quality, sanitation | Planes 0-1 | In development |
| Emergency Response | Case management, supply chain, incident reporting | Planes 0-2 | In development |
| Education Monitoring | Student registry, attendance, learning assessments | Planes 1-2 | Planned |
| Environmental Compliance | Emissions inventory, compliance tracking, audit trail | Planes 2-3 | Planned |

## 3. Distribution structure

Each distribution is a packaged archive with a YAML manifest, Docker Compose files, and pre-configured content:

```
zs-distro-primary-health-care-1.2.0/
├── manifest.yaml              ← Distribution metadata (name, version, checksums)
├── docker-compose.yml         ← Service definitions for Planes 1-4
├── docker-compose.plane0.yml  ← Consolidated Plane 0 configuration
├── config/                    ← Pre-configured settings per plane
│   ├── plane0/
│   ├── plane1/
│   ├── plane2/
│   └── plane3/
├── forms/                     ← Pre-loaded form definitions (FHIR Questionnaire)
├── modules/                   ← Pre-packaged domain modules
├── workflows/                 ← Pre-loaded workflow templates
├── dashboards/                ← Pre-configured dashboard definitions
├── scripts/                   ← Setup, sync, and migration scripts
├── checksums.sha256           ← Integrity checksums for all files
├── signature.asc              ← GPG signature of checksums file
└── README.md                  ← Quick-start guide and requirements
```

### Archive format

Distributions are distributed as `tar.gz` archives:

```
zs-distro-primary-health-care-1.2.0.tar.gz
```

The archive name follows the pattern `zs-distro-{name}-{semver}.tar.gz`. All filenames use lowercase with hyphens.

### Manifest format

```yaml
# manifest.yaml
apiVersion: zarishsphere.io/v1
kind: Distribution
metadata:
  name: primary-health-care
  displayName: "Primary Health Care Distribution"
  version: "1.2.0"
  description: >
    Complete primary health care solution with patient registry,
    NCD tracking, immunization scheduling, and nutrition screening.
  maintainer: "ZarishSphere Foundation <foundation@zarishsphere.com>"
  tags:
    - health
    - primary-care
    - immunization
    - nutrition
  license: "Apache-2.0"

spec:
  targetPlanes: [0, 1, 2]
  minPlane: 0
  dependencies:
    modules:
      - health-common: "1.5.0"
      - patient-registry: "2.1.0"
      - immunization: "1.8.0"
      - nutrition-screening: "1.3.0"
      - ncd-tracker: "1.1.0"
    apps:
      - immunization-tracker: "1.2.0"
      - nutrition-screening: "1.0.0"
    services:
      - identity
      - audit
      - sync
      - export
  system:
    minCpus: 1
    recommendedCpus: 2
    planes:
      plane0:                  # Air-gapped (Plane 0) — authoritative minimums
                               # per 003-zs-platform/003-deployment-planes.md
        minMemoryMb: 4096
        minStorageGb: 64
      edge:                    # Edge / Plane 1 — smaller figures
        minMemoryMb: 512
        recommendedMemoryMb: 2048
        minStorageGb: 10
        recommendedStorageGb: 50
```

## 4. One-click deployment

Distributions deploy from the Marketplace in one click:

1. User selects a distribution in the → **[010-zs-ecosystem/002-marketplace-spec.md](010-zs-ecosystem/002-marketplace-spec.md)**
2. Console downloads the archive from the content repository
3. Archive integrity verified against `checksums.sha256` and `signature.asc`
4. Dependencies checked against available modules and services
5. Plane-specific configuration applied from `config/{plane}/`
6. Docker Compose services started (or single binary for Plane 0)
7. Forms and workflows loaded into the Engine
8. Health checks run against all services
9. User redirected to the Console dashboard for the deployed distribution

### CLI deployment

```bash
# Download and verify
zs distribution download primary-health-care --version 1.2.0 --out ./distros/
zs distribution verify ./distros/zs-distro-primary-health-care-1.2.0.tar.gz

# Deploy to a plane
zs distribution deploy ./distros/zs-distro-primary-health-care-1.2.0.tar.gz \
    --plane 1 \
    --config ./configs/edge-clinic.yaml
```

## 5. Distribution lifecycle

Every distribution follows a managed lifecycle:

```
Build → Test → Sign → Publish → Verify → Deploy → Update → Deprecate
```

| Stage | Description | Tooling |
|---|---|---|
| **Build** | Assemble archive from component repos, resolve dependency versions, generate manifests | CI pipeline (GitHub Actions) |
| **Test** | Deploy to test environment (Plane 0-2 emulators), run integration test suite, validate all forms and workflows | CI pipeline |
| **Sign** | Generate SHA-256 checksum file, sign with Foundation GPG key | Automated signing in CI |
| **Publish** | Upload archive to `zs-content-distributions` repository, publish listing in Marketplace | GitHub release + API |
| **Verify** | Consumer verifies checksums and GPG signature before deployment | CLI `zs distribution verify` |
| **Deploy** | Unpack archive, apply plane-specific configuration, start services | Console or CLI |
| **Update** | Download delta update, apply migration scripts, restart affected services | Console notification |
| **Deprecate** | Mark version as deprecated in Marketplace, recommend upgrade path | Foundation action |

## 6. Plane-specific optimizations

Each distribution includes plane-aware configurations that adjust the deployment for the target environment.

### Plane 0 (air-gapped)

- **Database:** SQLite instead of PostgreSQL
- **Services:** Single binary running all services in-process
- **Storage:** 64 GB minimum local storage, USB sync bundles
- **Authentication:** Local accounts with bcrypt, no OAuth2
- **Networking:** No outbound network access
- **UI:** PWA cache, all assets bundled in archive
- **Binary size:** Optimised for < 100 MB download
- **RAM target:** 4 GB minimum — authoritative per → **[003-zs-platform/003-deployment-planes.md](003-zs-platform/003-deployment-planes.md)**. Earlier drafts listed a 256 MB figure for air-gapped deployments; that figure applies only to the smaller Edge/Plane 1 hardware profile and is corrected here to the plane-doc minimums (4 GB RAM, 64 GB storage).

### Plane 1 (edge / RPi)

- **Database:** SQLite or PostgreSQL (if available)
- **Services:** Separate binaries for core services, optional microservices
- **Storage:** Local filesystem with periodic sync
- **Authentication:** Local accounts, optional OAuth2 proxy
- **Networking:** Intermittent, sync queue with retry
- **UI:** PWA with service worker cache
- **Hardware:** ARM64, 1 GB RAM minimum

### Plane 2 (district server)

- **Database:** PostgreSQL
- **Services:** Full microservice deployment via Docker Compose
- **Storage:** NFS or local RAID
- **Authentication:** Keycloak with LDAP federation
- **Networking:** Continuous, NATS JetStream for internal messaging
- **UI:** Full Console with all features
- **Hardware:** x86_64, 4 GB RAM minimum

### Plane 3 (national cloud)

- **Database:** PostgreSQL cluster with streaming replication
- **Services:** Docker Compose with horizontal scaling
- **Storage:** SAN/NAS with automated backup
- **Authentication:** Keycloak cluster with SAML/OIDC federation
- **Networking:** Load-balanced, TLS everywhere
- **UI:** Full Console with multi-org support
- **Hardware:** 8+ GB RAM, multiple nodes

### Plane 4 (global SaaS)

- **Database:** PostgreSQL cluster with read replicas
- **Services:** Kubernetes-native deployment (Helm charts)
- **Storage:** Cloud object storage (S3-compatible)
- **Authentication:** Keycloak federation with social login
- **Networking:** Global load balancing, CDN
- **UI:** Full Console with multi-tenant isolation
- **Hardware:** Elastic, auto-scaling

Detailed plane specifications are in → **[003-zs-platform/003-deployment-planes.md](003-zs-platform/003-deployment-planes.md)**.

## 7. Version pinning and signing

All distributions use strict version pinning for every component:

```yaml
# manifest.yaml — dependency versions pinned exactly
dependencies:
  modules:
    - health-common: "1.5.0"      # Exact version, no ranges
    - patient-registry: "2.1.0"
    - immunization: "1.8.0"
  apps:
    - immunization-tracker: "1.2.0"
```

### Pinning rules

- Every dependency is pinned to an exact semver version — no version ranges, no unversioned references, no `^` or `~` prefixes.
- All transitive dependencies are resolved at build time and recorded in a `lock.yaml` manifest.
- Version updates are intentional, tested, and released as new distribution versions.
- The `latest` tag is forbidden per ZUSS §4.3 in all distribution metadata.

### Signing process

1. Build process generates `checksums.sha256` for every file in the archive
2. Foundation GPG key signs the checksums file: `gpg --detach-sign --armor checksums.sha256`
3. Signature file `checksums.sha256.asc` is included in the archive
4. Public GPG key is published on the Foundation website and in the GitHub org

### Verification

```bash
# Verify archive integrity
zs distribution verify ./zs-distro-primary-health-care-1.2.0.tar.gz

# Manual verification
tar xzf zs-distro-primary-health-care-1.2.0.tar.gz
cd zs-distro-primary-health-care-1.2.0
gpg --verify checksums.sha256.asc checksums.sha256
sha256sum -c checksums.sha256
```

## 8. Update channel

Distributions receive updates through a structured channel:

### Update mechanism

- **Pull-based** — the Console periodically checks for updates by querying the Marketplace API for newer distribution versions.
- **Diff bundles** — updates are delivered as differential archives containing only changed files, reducing download size by 60-90% for typical updates.
- **Migration scripts** — each update includes SQL migration scripts and configuration transforms in a `migrations/` directory.

### Update workflow

```
Console detects update available
        │
        ▼
User reviews changelog (linked in Marketplace)
        │
        ▼
User approves update
        │
        ▼
System downloads diff bundle
        │
        ▼
System creates backup snapshot
        │
        ▼
Run pre-update health checks
        │
        ▼
Apply migration scripts
        │
        ▼
Restart affected services
        │
        ▼
Run post-update verification
        │
        ▼
Update complete — notification sent
```

### Rollback

If post-update verification fails, the system automatically rolls back:

1. Restore backup snapshot
2. Revert migration scripts
3. Restart services
4. Notify operator with rollback report

Manual rollback is also supported:

```bash
zs distribution rollback primary-health-care --version 1.1.0
```

### Update channels

| Channel | Cadence | Stability | Audience |
|---|---|---|---|
| `stable` | Monthly | Production-ready | All users |
| `rc` | Weekly | Release candidates | Testers |
| `nightly` | Daily | Bleeding-edge | Developers |

Users opt into channels via Console settings. The default channel is `stable`.

## 9. Requirements and documentation

Every distribution includes embedded documentation and system requirements.

### Requirements section in manifest

```yaml
spec:
  requirements:
    hardware:
      minCpus: 1
      recommendedCpus: 2
      planes:
        plane0:               # Air-gapped (Plane 0) — authoritative minimums
                              # per 003-zs-platform/003-deployment-planes.md
          minMemoryMb: 4096
          minStorageGb: 64
        edge:                 # Edge / Plane 1 — smaller figures
          minMemoryMb: 512
          recommendedMemoryMb: 2048
          minStorageGb: 10
          recommendedStorageGb: 50
    software:
      os: "Linux kernel 5.10+"
      runtime: "Docker Engine 24+ or containerd 1.7+"
      database: "PostgreSQL 16+ (Planes 1-4) or SQLite 3.44+ (Plane 0)"
    network:
      plane0: "No network required"
      plane1: "Intermittent (2G+ recommended)"
      plane2: "Broadband (1 Mbps+)"
      plane3: "Dedicated (10 Mbps+)"
```

### Embedded README

Each distribution includes a `README.md` file with:

- Purpose and use case description
- System requirements (hardware, software, network)
- Quick-start deployment instructions
- Plane-specific guidance
- Configuration reference
- Maintenance procedures (backup, update, rollback)
- Troubleshooting guide
- License and attribution

### Additional documentation

Distributions may also include:

- `docs/` — extended documentation directory with architecture diagrams, API references
- `CHANGELOG.md` — version history with migration notes
- `LICENSE` — Apache 2.0 license text
- `SECURITY.md` — security contact and disclosure policy

## 10. Cross-references

→ **003-zs-platform/003-deployment-planes.md** — Deployment planes: the authoritative hardware minimums — Plane 0 air-gapped is 4 GB RAM and 64 GB storage
→ **010-zs-ecosystem/016-content-templates-spec.md** — zs-content-templates specification: template and distribution packages; its air-gap manifest agrees with this document at 4 GB RAM and 64 GB storage
→ **010-zs-ecosystem/002-marketplace-spec.md** — Marketplace specification: the one-click deployment entry point for distributions
→ **010-zs-ecosystem/010-modules-spec.md** — Modules specification: the domain modules bundled into distributions
→ **010-zs-ecosystem/004-apps-spec.md** — Apps specification: the pre-built apps bundled into distributions
→ **010-zs-ecosystem/012-engine-spec.md** — Engine specification: loads forms and workflows during distribution deployment
→ **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS §4.3: the Docker image `latest` tag prohibition applied to distribution metadata

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
