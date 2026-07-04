# Platform Engineering — Capabilities in depth

M1D0 Technologies runs a single, standardized engineering platform — a *"fleet standard"* — that lets multiple production web ventures share one hardened, observable, fully self‑hosted operating model. This document describes each capability. It contains no source, no hostnames, and no operational detail.

---

## 1. Shared platform architecture — the "fleet standard"
Every venture is built to one deliberately uniform application standard rather than a bespoke stack, so a fix or upgrade in one propagates as a pattern to all. The standard: Next.js 16 (App Router, React Server Components‑first), React 19, TypeScript 6 strict, Tailwind CSS v4 with CSS‑first design tokens, a pnpm‑workspace monorepo, a shared self‑hosted headless CMS, schema‑validated environment access, and a standalone containerized production image. Coherence is enforced mechanically — a CI conformance guard asserts platform version floors and structural invariants across repos while allowing legitimate per‑brand divergence (auth, RBAC, design tokens).

## 2. Deployment & edge
Each app ships as a container behind a Caddy reverse proxy, reached exclusively through a Cloudflare Tunnel with Cloudflare's edge in front — **no exposed origin ports, no inbound firewall surface**, TLS terminated at the edge. Across the platform, dozens of self‑hosted service stacks run this way, all reached only through the edge tunnel. Base compose files plus per‑environment overrides drive topology, and unlaunched surfaces sit behind an intentional maintenance‑gating layer: reachable to operators, dark to the public.

## 3. Infrastructure as code
Cloudflare and GitHub state is codified in **OpenTofu**, with remote state and native locking. Managed as code: DNS, a fleet of zero‑trust access applications and their policies, the tunnel, and branch protection across repositories. Apply policy is risk‑tiered — routine settings apply at zero‑diff, while access/tunnel resources are observe‑only because they gate dark production. Host and platform infrastructure lives in **Ansible**: a full host baseline plus a second‑region frontend readiness tier — authored and validated, never auto‑applied, with dry‑run validation tiers.

## 4. CI/CD & release engineering
CI, deploy, and code scanning are **100% self‑hosted** on a labeled runner fleet — zero externally‑hosted jobs, billing‑independent and outage‑immune, with per‑repo isolation. The IaC repo runs plan‑on‑PR / apply‑on‑merge; app repos run containerized build / type‑check / lint / unit / E2E / accessibility jobs, then deploy with **staging auto‑deploy, gated manual production promotion of the same digest, single‑flight locking, health‑gating, and automatic rollback tagging.** Dependencies stay current via self‑hosted automated updates (grouped, SHA‑pinned, majors gated). Release tags trigger a supply‑chain pipeline: vulnerability scan, SBOM, keyless signing, digest‑pinned base images. Post‑deploy smoke tests (including data‑integrity checks) gate every release.

## 5. Observability & reliability
A complete self‑hosted stack is in production: **Prometheus** (SLO, backup, probe, and watchdog alerting), **Grafana** dashboards, **Loki + Alloy** log aggregation, **Tempo** distributed tracing, **Alertmanager**, **Uptime Kuma** external uptime, and self‑hosted application error monitoring. Reliability is defended by **dead‑man's‑switch canaries** (alert if the alerting path itself goes silent), multiple watchdogs on load‑bearing services, and a weekly health digest.

## 6. Backup & disaster recovery
Backups are layered and **verified, not assumed.** Snapshot backups run with a dedicated restore‑test routine; per‑service database backups are paired with scheduled **restore drills** that actually rehydrate and validate data. A tiered config archive runs nightly, and a critical off‑box set (encryption keys and small secrets) is pushed off‑site with round‑trip verification and freshness guards wired into alerting.

## 7. Security posture
Defense is **deny‑by‑default and layered.** External access sits behind zero‑trust; edge protection combines a WAF with a self‑hosted decision engine and an IP‑list bouncer. Applications enforce deny‑by‑default tiered RBAC through a single audited authorization choke point, with fail‑closed auth (a weak or missing signing secret refuses to start). **Secrets never touch git** — repo‑wide secret scanning, a dedicated secrets manager and vault, host secrets encrypted at rest, and an off‑site critical set. Email is hardened across all brand domains — **DMARC at `p=reject`**, DKIM, SPF, and MTA‑STS published as code. The web tier ships strict security headers and a CSP with a per‑request nonce. Container privilege is minimized (no exposed Docker socket, dropped capabilities, read‑only rootfs, non‑root, no‑new‑privileges on hardened services).

## 8. AI & automation platform
M1D0 operates a **self‑hosted, loopback‑only internal engineering agent hub** — a curated library of agent skills orchestrated from one canonical source of truth and distributed by sync tooling, used for internal engineering only. Business‑process automation runs on **n8n**: public forms flow *form → automation → CMS / database → system‑of‑record*, with retries, backoff, anti‑abuse protection, and dead‑man's‑switch monitoring on the automation path.

## 9. Quality & accessibility engineering
Quality is gated in CI, not left to review. Every app runs Playwright end‑to‑end tests and automated **accessibility checks with axe** against WCAG intent (correct document language/direction, semantic structure, accessibility baselines per venture), alongside strict type‑check and lint gates and a coverage philosophy targeting ~80%. Design‑quality guardrails are codified too.

---

<sub>Capability description only — no source, hostnames, exact counts, or operational detail. © 2026 Dahhan Enterprises LLC — M1D0 Technologies, Dahhan Industries, Miss Dantella and affiliated brands. All rights reserved.</sub>
