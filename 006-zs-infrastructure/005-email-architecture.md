# 005-email-architecture.md
## Secure decentralised communication and relay infrastructure
### Email routing, security, forwarding, and notification strategy

**Document type:** Architecture
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Draft

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Design principles](#2-design-principles)
3. [Email infrastructure overview](#3-email-infrastructure-overview)
4. [Routing addresses](#4-routing-addresses)
5. [Email security (SPF, DKIM, DMARC)](#5-email-security-spf-dkim-dmarc)
6. [Notification email architecture](#6-notification-email-architecture)
7. [Contact form email integration](#7-contact-form-email-integration)
8. [Transactional email](#8-transactional-email)
9. [Anti-spam policy](#9-anti-spam-policy)
10. [Plane 0 email strategy](#10-plane-0-email-strategy)
11. [Cross-references](#11-cross-references)

---

## 1. Purpose

This document defines the email architecture for the ZarishSphere ecosystem. Email at ZarishSphere serves three functions:

1. **Inbound contact** — Public email addresses for inquiries, contributions, submissions, and security disclosures
2. **Notification** — System-generated email alerts (deployment status, security events, account activity)
3. **Transactional** — User-facing emails (password reset, verification, confirmation)

All email infrastructure runs on the Cloudflare free tier. No mail server is operated. No email infrastructure cost is incurred.

---

## 2. Design principles

### 2.1 No mail server

ZarishSphere operates no mail server of its own. All inbound email is handled by Cloudflare Email Routing. All outbound email (notifications, transactional) is sent via Cloudflare Email Routing or a transactional email provider that supports the free tier.

> **Constraint:** No email infrastructure component may require a dedicated mail server, SMTP relay, or paid email service. The system must function within Cloudflare Email Routing free tier limits.

### 2.2 Zero-cost forwarding

All `@zarishsphere.com` addresses are forwarding aliases. No mailbox storage is provided — all email is routed to the Foundation's central mailbox (`zarishsphere@gmail.com`). This keeps costs at zero while providing unlimited address creation.

### 2.3 Plane 0 compatibility

On Plane 0 (air-gapped), email is not available. The system queues outgoing messages for transmission when connectivity is restored. Notifications are displayed in-app as an alternative delivery channel.

### 2.4 Security-first

Every email domain is protected by SPF, DKIM, and DMARC at enforcement level. Email spoofing is prevented. Phishing attempts are blocked at the DNS level before they reach any recipient.

---

## 3. Email infrastructure overview

### 3.1 Component diagram

```
Inbound flow:

Sender → zarishsphere.com DNS → Cloudflare Email Routing
                                       ↓
                              Forwarding rules
                                       ↓
                              zarishsphere@gmail.com
                                       ↓
                              Foundation mailbox (central)

Outbound flow:

Application (Worker / Go service) → Transactional provider
                                       ↓
                              zarishsphere.com sending
                                       ↓
                              Recipient inbox
```

### 3.2 Service inventory

| Service | Provider | Free tier limit | Purpose |
|---|---|---|---|
| Email Routing | Cloudflare | Unlimited inbound-only aliases | Inbound email forwarding |
| DKIM signing | Cloudflare | Auto-configured | Outbound email authentication |
| SPF record | Cloudflare DNS | Included | Sender policy |
| DMARC policy | Cloudflare DNS | Included | Domain alignment enforcement |
| Transactional sending | Reserved (future) | — | System-generated outbound email |

---

## 4. Routing addresses

### 4.1 Primary addresses

All addresses are configured as routing rules in Cloudflare Email Routing. The catch-all rule forwards any unmatched address to the central mailbox.

| Address | Purpose | Forward target |
|---|---|---|
| `hello@zarishsphere.com` | General contact and inquiries | `zarishsphere@gmail.com` |
| `founder@zarishsphere.com` | Founder direct contact | `zarishsphere@gmail.com` |
| `contribute@zarishsphere.com` | Contribution submissions | `zarishsphere@gmail.com` |
| `index@zarishsphere.com` | ZARISH-INDEX submissions | `zarishsphere@gmail.com` |
| `standards@zarishsphere.com` | ZARISH-STANDARDS inquiries | `zarishsphere@gmail.com` |
| `security@zarishsphere.com` | Security vulnerability disclosure | `zarishsphere@gmail.com` |
| `legal@zarishsphere.com` | Licensing and legal inquiries | `zarishsphere@gmail.com` |

### 4.2 Catch-all behaviour

| Setting | Value |
|---|---|
| Catch-all | Enabled |
| Unmatched address destination | `zarishsphere@gmail.com` |
| Spam filtering | Gmail's built-in spam filter applies to forwarded mail |
| Reason | Ensures no legitimate email is lost during early-stage operations |

> **Note:** The catch-all may generate spam noise. As the project matures, specific routing rules will replace the catch-all. Each address will be individually configured and the catch-all will be disabled.

### 4.3 Future addresses (post-launch)

| Address | Purpose |
|---|---|
| `support@zarishsphere.com` | User support |
| `privacy@zarishsphere.com` | Privacy inquiries (GDPR/CCPA) |
| `abuse@zarishsphere.com` | Abuse reporting |
| `press@zarishsphere.com` | Media and press inquiries |
| `partners@zarishsphere.com` | Partnership inquiries |

---

## 5. Email security (SPF, DKIM, DMARC)

### 5.1 SPF (Sender Policy Framework)

SPF defines which servers are authorised to send email from `zarishsphere.com`.

| Record | Value |
|---|---|
| Type | TXT |
| Name | `zarishsphere.com` |
| Value | `v=spf1 include:_spf.mx.cloudflare.net ~all` |

| Mechanism | Meaning |
|---|---|
| `include:_spf.mx.cloudflare.net` | Cloudflare Email Routing servers are authorised senders |
| `~all` | Soft fail — unauthorised senders are marked as suspicious but not rejected |

**Future hardening:** When a transactional email provider is selected, the SPF record will be updated to include that provider's SPF include. The `~all` will be changed to `-all` (hard fail) once all authorised senders are accounted for.

### 5.2 DKIM (DomainKeys Identified Mail)

DKIM provides cryptographic signing of outgoing emails. Cloudflare Email Routing auto-generates and manages DKIM keys.

| Record | Value |
|---|---|
| Type | TXT |
| Name | `*._domainkey.zarishsphere.com` |
| Value | Auto-generated by Cloudflare Email Routing |
| Key length | 2048-bit RSA |
| Selector | Cloudflare-managed |

**Key rotation:** DKIM keys are rotated by Cloudflare automatically. No manual intervention is required.

### 5.3 DMARC (Domain-based Message Authentication, Reporting, and Conformance)

DMARC tells receiving mail servers what to do when SPF or DKIM validation fails.

| Record | Value |
|---|---|
| Type | TXT |
| Name | `_dmarc.zarishsphere.com` |
| Value | `v=DMARC1; p=quarantine; rua=mailto:security@zarishsphere.com; ruf=mailto:security@zarishsphere.com; pct=100` |

| DMARC tag | Value | Purpose |
|---|---|---|
| `p` | `quarantine` | Messages failing authentication are sent to spam |
| `rua` | `security@zarishsphere.com` | Aggregate DMARC reports (daily/weekly summaries) |
| `ruf` | `security@zarishsphere.com` | Forensic failure reports (individual message details) |
| `pct` | `100` | Policy applies to 100% of messages |
| `sp` | (inherits `p`) | Subdomain policy matches domain policy |

**DMARC policy evolution:**

| Phase | Policy | Condition |
|---|---|---|
| Pre-launch (testing) | `p=none` (monitoring only) | During initial DNS setup, before all legitimate senders are verified |
| Launch / current | `p=quarantine` | After verifying all legitimate senders are authenticated |
| Post-launch (mature) | `p=reject` | After 6+ months of clean DMARC reporting |

> **Implementation note:** The recorded DMARC policy is `p=quarantine` (see the record table above). During early DNS testing the policy may be temporarily set to `p=none` (monitoring only); it is raised back to `p=quarantine` once all legitimate senders are verified.

### 5.4 BIMI (future)

Brand Indicators for Message Identification (BIMI) is reserved for post-launch adoption. BIMI requires `p=reject` DMARC before it can be implemented.

---

## 6. Notification email architecture

### 6.1 Notification types

| Notification type | Trigger | Delivery method | Priority |
|---|---|---|---|
| Deployment complete | GitHub Actions workflow success | Email to `hello@zarishsphere.com` | Low |
| Deployment failed | GitHub Actions workflow failure | Email to `hello@zarishsphere.com` | High |
| Security alert | WAF attack detected, DMARC report | Email to `security@zarishsphere.com` | High |
| Certificate renewal | SSL certificate nearing expiry | Email to `hello@zarishsphere.com` | High |
| DNS change | DNS record modified | Email to `hello@zarishsphere.com` | Medium |
| Storage limit | R2 storage approaching free tier limit | Email to `hello@zarishsphere.com` | Medium |

### 6.2 Notification sending

During pre-launch, all notifications are sent directly from the triggering service (GitHub, Cloudflare) to the Foundation mailbox. No custom notification service is needed.

Post-launch, a notification microservice (part of ZarishSphere Services) will manage notification delivery with queuing, retry, and delivery status tracking.

---

## 7. Contact form email integration

### 7.1 Contact forms

Static sites (home page, docs) include contact forms that submit to Cloudflare Workers:

```
User submits form → Cloudflare Worker → Email Routing → hello@zarishsphere.com
```

### 7.2 Form security

| Security measure | Implementation |
|---|---|
| CAPTCHA | Cloudflare Turnstile (free, privacy-first) |
| Rate limiting | Workers rate limit: 5 form submissions per IP per hour |
| Content validation | Worker validates all fields server-side |
| Honeypot field | Hidden field to block bots |
| CORS | Restricted to originating domain only |

---

## 8. Transactional email

### 8.1 Transactional email strategy

Transactional email (password reset, verification codes, confirmation receipts) is not required during the pre-launch phase because:

1. The Console is not yet deployed to production
2. User accounts are not yet self-service
3. All contact is handled through manual email correspondence

### 8.2 Future transactional email provider

When transactional email is needed, the following constraint applies:

> **Constraint:** The transactional email provider must have a free tier that supports at least 100 emails/day. No paid email service may be used.

Potential providers evaluated when needed:

| Provider | Free tier limit | Notes |
|---|---|---|
| Cloudflare Email Routing | Outbound not supported (inbound only) | Not an option for outbound |
| SendGrid | 100 emails/day | Viable for low volume |
| Mailgun | 100 emails/day | Viable for low volume |
| Resend | 100 emails/day | Modern API, good DX |
| SMTP2GO | 1,000 emails/month | Viable for low volume |
| GitHub Notifications | Via GitHub-only | For developer notifications only |

### 8.3 Transactional email templates

Each transaction type uses a distinct template:

| Template | Trigger | Content |
|---|---|---|
| Password reset | User requests password reset | Reset link, expiry time |
| Email verification | New user registration | Verification link |
| Deployment notification | CI/CD pipeline event | Status, commit, links |
| Security alert | Suspicious activity detected | Event details, action required |

---

## 9. Anti-spam policy

### 9.1 Inbound spam protection

| Layer | Protection |
|---|---|
| DNS | SPF, DKIM, DMARC reject spoofed email at the domain level |
| Cloudflare | DDoS protection blocks volumetric spam attacks |
| Gmail | Inbound spam filter (applied after forwarding) |
| Manual | Security team reviews DMARC reports for abuse patterns |

### 9.2 Outbound spam prevention

| Measure | Implementation |
|---|---|
| Rate limiting | Transactional sending capped at 100 emails/day |
| Authentication | SPF + DKIM on all outbound email |
| List hygiene | No purchased lists; all recipients opt in |
| Unsubscribe | Every notification email includes an unsubscribe link |
| Monitoring | Bounce rate and complaint rate tracked |

### 9.3 DMARC reporting

DMARC aggregate reports (`rua`) are sent to `security@zarishsphere.com`. These reports are reviewed:

- Weekly during pre-launch and launch phases
- Monthly after 6 months of stable DMARC reporting
- Immediately if a spoofing incident is suspected

---

## 10. Plane 0 email strategy

On Plane 0 (air-gapped), there is no internet connectivity and therefore no email:

### 10.1 Queuing strategy

1. Outgoing messages are serialised to a local message queue (SQLite-backed)
2. When connectivity is restored (device moves to Plane 1+), the queue is drained
3. Messages are sent through the transactional email provider
4. Failed messages are retried with exponential backoff (max 3 retries)

### 10.2 Offline notification alternatives

| Notification type | Plane 0 delivery |
|---|---|
| System alerts | In-app notification drawer |
| Deployment updates | Local log file |
| Security events | Local audit log |
| User-facing messages | In-app message centre |

### 10.3 Email at higher planes

| Plane | Email capability |
|---|---|
| 0 | Queued only (no delivery) |
| 1 | Occasional — queue and send when connected |
| 2 | Periodic — scheduled sync sends queued mail |
| 3 | Persistent — near-real-time delivery |
| 4 | Always-on — real-time delivery |

---

## 11. Cross-references

→ **006-zs-infrastructure/003-cloudflare-architecture.md** — Cloudflare Email Routing configuration and routing addresses
→ **006-zs-infrastructure/004-domain-architecture.md** — DNS security records (SPF, DKIM, DMARC, CAA)
→ **006-zs-infrastructure/006-ci-cd-architecture.md** — Deployment notification triggers
→ **006-zs-infrastructure/008-security-policies.md** — Vulnerability disclosure and incident response
→ **003-zs-platform/003-deployment-planes.md** — Plane definitions and email capability per plane

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
