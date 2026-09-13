# Product Requirements Document (PRD)
## V-Meet — Free, Open-Source, Cross-Platform Video Conferencing App

**Status:** Draft v1
**Owner:** [Your name]
**Last updated:** September 13, 2026

---

## 1. Overview

### 1.1 Problem Statement
Existing video conferencing tools (Zoom, Teams, Meet) are closed-source, often paywalled at scale, and don't offer embeddable/self-hostable options for teams who want to run their own infrastructure or integrate calling directly into another product. There's a need for a free, open-source, cross-platform alternative that is easy to self-host and easy to embed as an SDK.

### 1.2 Product Vision
An open-source video conferencing platform, available on Web, Desktop (Windows/Mac/Linux), iOS, and Android, that any team can self-host for free, embed into their own application via SDK, and moderate safely with built-in content and attention monitoring tools.

### 1.3 Goals
- Free to run at small/medium scale via self-hosting.
- Fully cross-platform from a single backend.
- Embeddable as an SDK into third-party applications.
- Safer group calls via automated moderation.
- Open-source, community-extensible codebase.

### 1.4 Non-Goals (v1)
- Not competing on enterprise features (webinars for 1000+ attendees, breakout room analytics, AI meeting summaries) in v1.
- Not building a hosted, infinitely-scaling free SaaS — the free model is "free to self-host," not "free unlimited hosting provided by us" (see Section 8, Hosting Model).
- No native UI direction decided yet — this PRD intentionally excludes UI/UX specification (see Section 10).

---

## 2. Target Users

| User type | Need |
|---|---|
| Small teams / communities | Free group video calls without per-seat pricing |
| Developers | An embeddable calling SDK to add to their own product |
| Privacy-conscious orgs | Self-hosted deployment, full data control |
| Educators / meeting leaders | Moderation tools (focus tracking, auto-moderation) to manage group behavior |

---

## 3. Core Features (v1 Scope)

### 3.1 Video Calling (Must-have)
- Create/join a room via a shareable link.
- Support group calls (target: up to 25 concurrent participants per room for v1; revisit for larger scale in v2).
- Audio/video toggle (mute/unmute, camera on/off) per participant.
- Screen sharing.
- Adaptive video quality based on network conditions (simulcast).
- Auto-reconnect on network drop.

### 3.2 Cross-Platform Clients (Must-have)
- Web (Chrome, Firefox, Safari, Edge).
- Desktop (Windows, macOS, Linux) via a shared client shell.
- Mobile (iOS, Android) via a shared codebase.
- All clients share one backend and one account system.

### 3.3 Text Chat (Should-have)
- In-call text chat, scoped per room, not persisted beyond the meeting by default.

### 3.4 Focus/Attention Tracking (Should-have)
- Per-participant focus signal, computed client-side, shown to the meeting leader only.
- Two detection tiers:
  - Tier 1 (default, always available): tab/window focus detection.
  - Tier 2 (opt-in, leader-enabled): webcam-based face/gaze presence detection.
- Must be clearly disclosed to participants when active (visible in-call indicator).
- No raw video or images are transmitted for this feature — only a derived boolean/score.

### 3.5 Automated Moderation (Should-have)
- Real-time profanity detection on text chat.
- Optional real-time profanity detection on voice (opt-in per room, given transcription cost).
- Configurable moderation policy: **warn-then-remove** by default; leader can adjust strictness.
- Leader-only moderation dashboard: view flagged incidents, sender, timestamp, and one-click re-admit for any automatic removal.
- Full audit log of moderation actions per meeting, visible to the leader.

### 3.6 Account & Room Management (Must-have)
- User registration/login (email + password to start; SSO can be a later addition).
- Meeting scheduling (create a room in advance with a link).
- Host controls: mute all, remove participant, end meeting for all.
- Waiting room: host approves entry before a participant joins media.

### 3.7 SDK / Embeddability (Must-have — key differentiator)
- Packaged as an installable SDK (npm package for web/RN, native library references for iOS/Android/desktop).
- Minimal integration path: a third-party developer should be able to embed a working call view in under ~20 lines of code.
- Documented auth flow for issuing room tokens from an external app's own backend.

---

## 4. Functional Requirements

| ID | Requirement | Priority |
|---|---|---|
| FR-1 | User can create a room and receive a shareable join link | Must |
| FR-2 | User can join a room via link without installing anything (web) | Must |
| FR-3 | System supports simultaneous audio+video for up to 25 participants per room | Must |
| FR-4 | User can mute/unmute audio and toggle video independently | Must |
| FR-5 | User can share their screen | Must |
| FR-6 | Host can remove any participant manually | Must |
| FR-7 | System detects profane text chat messages and applies moderation policy | Should |
| FR-8 | System detects profane speech (opt-in) and applies moderation policy | Could |
| FR-9 | System computes and surfaces a per-participant focus signal to the host | Should |
| FR-10 | Host can view a moderation audit log for the meeting | Should |
| FR-11 | External developers can embed the calling client via SDK with a documented token-issuing flow | Must |
| FR-12 | System supports self-hosted deployment via a documented Docker/Kubernetes setup | Must |

---

## 5. Non-Functional Requirements

| Category | Requirement |
|---|---|
| Performance | Media latency under 300ms for direct P2P/SFU-routed connections under normal network conditions |
| Scalability | SFU nodes must scale horizontally based on active room/participant count |
| Availability | Target 99% uptime for the reference-hosted instance (not a guaranteed SLA — self-hosted deployments are the user's own responsibility) |
| Security | All media encrypted end-to-end via DTLS-SRTP (WebRTC default); room-scoped, short-lived access tokens; rate-limited auth endpoints |
| Privacy | No raw video/audio persisted without explicit recording consent flow; focus-tracking data never leaves the client as raw imagery |
| Compliance | Clear in-app disclosure whenever focus tracking or voice moderation is active, to support GDPR/workplace-monitoring-law compliance |
| Portability | Full stack deployable via Docker Compose for self-hosting; documented for at least one major cloud free tier |

---

## 6. System Architecture (Summary)

- **SFU (media routing):** mediasoup or LiveKit.
- **Signaling:** WebSocket-based, custom or reused from chosen SFU.
- **Backend API:** REST/GraphQL for auth, rooms, scheduling.
- **Database:** PostgreSQL (durable data), Redis (session/room state).
- **TURN/STUN:** coturn, self-hosted or via a free-tier relay provider.
- **Clients:** React (web), React Native or Flutter (mobile), Tauri or Electron (desktop) wrapping the web client.

*(Full technical breakdown and role-by-role build steps are covered in the separate execution guide documents already produced.)*

---

## 7. Team & Ownership

Roles and their assigned responsibilities (Project Lead, Real-Time Media Engineer, Backend Engineer, Web/Mobile/Desktop Engineers, DevOps, Security, QA, Docs/Community) are defined in the companion **Role Execution Guide**. This PRD assumes that structure; it is not repeated in full here to avoid duplication.

---

## 8. Hosting & Cost Model

The product ships with **two hosting paths**, not one:

### 8.1 Centrally-hosted free instance (primary, for general users)
- Anyone can create and join meetings at `app.openmeet.org` (or similar) with **no cost to the user** — no account limits, no meeting-duration caps, no participant caps by design.
- The architecture is built for **horizontal, uncapped scaling**: additional SFU nodes, TURN relays, and signaling servers can be added automatically as demand grows, with no hard technical ceiling on total users or concurrent rooms (see 8.4).
- This instance is operated and paid for by the project, not the end user. Being free and fully scalable to users does not mean the underlying compute/bandwidth is free — it means the project has committed to a funding model that scales alongside usage (see 8.3). This distinction should stay explicit in planning: **user-facing free ≠ zero operating cost.**

### 8.2 Self-hosted deployments (secondary, for teams who want full control)
- The full stack remains open-source and self-hostable via the documented Docker/Kubernetes setup, for teams who want their own infrastructure, data control, or independent scaling.
- Self-hosters absorb their own infra cost, independent of the central instance's budget.

### 8.3 Funding a fully scalable free instance
This is the central instance's core dependency, not a side risk — uncapped free hosting requires a funding source that grows with usage, not a one-time grant. Realistic paths, likely needed **in combination**:
- **Cloud sponsorship / open-source infra credits** — AWS Activate, Google for Startups, Oracle for Research, Cloudflare's open-source program. Worth applying to multiple simultaneously, and re-applying/renewing as usage grows, since most credit programs are capped or time-limited rather than indefinite.
- **Donations / Open Collective / GitHub Sponsors** — recurring community funding; realistically covers a fraction of costs at scale but adds resilience.
- **Grants from digital-rights or OSS foundations** — NLnet, Mozilla Open Source Support, Sovereign Tech Fund — some specifically fund free public communication infrastructure.
- **Optional paid support/enterprise/managed-hosting tier** — software and the public instance both stay free; a paid tier for orgs wanting SLAs, custom domains, or dedicated capacity cross-subsidizes the free instance without gating any core feature.
- **Corporate/foundation backer** — the model Jitsi Meet uses (backed by 8x8). If growth is the goal, actively seeking a backing organization is worth planning for, not just organic donations.
- Treat this as an ongoing operational responsibility, not a launch checkbox: cost scales with adoption, so funding needs to be revisited continuously, not solved once.

### 8.4 Engineering for uncapped scale
- SFU and TURN nodes scale horizontally via Kubernetes autoscaling, triggered on real-time load metrics (active rooms, per-node CPU/bandwidth) rather than fixed instance counts.
- Aggressive bandwidth efficiency (simulcast, SVC, selective forwarding) is treated as a cost-control requirement, not just a quality feature — at scale, wasted bandwidth is the direct cost driver.
- Multi-region SFU/TURN deployment as usage grows, to keep latency low and avoid unnecessary cross-region relay cost.
- Real-time cost/usage monitoring (Section 5, Non-Functional Requirements) feeds directly into funding decisions in 8.3 — infra spend should be visible and tracked against available funding, not discovered after the fact.

---

## 9. Success Metrics

| Metric | Target (v1, first 3 months post-launch) |
|---|---|
| Successful call connection rate | >95% of join attempts result in a working call |
| Self-hosting adoption | 50+ independent self-hosted deployments (tracked via docs analytics/community reporting, opt-in) |
| SDK integrations | 5+ third-party apps embedding the SDK |
| Moderation false-positive rate | <5% of auto-flagged incidents overturned by leader re-admit |
| Community engagement | Median issue/PR response time under 48 hours |

---

## 10. Open Questions / Explicitly Deferred

- **UI/UX direction:** not yet defined. No wireframes, visual design system, or interaction patterns exist yet — this is the next major piece of work and should be scoped separately once this PRD is validated.
- Maximum participant count for v1 (currently assumed 25 — needs load-testing data to confirm).
- Whether voice moderation ships in v1 or is deferred to v2 given its transcription cost.
- SSO/enterprise auth — deferred until there's demand signal from self-hosters.
- Recording feature — not yet scoped; raises additional consent/storage questions beyond this PRD's current coverage.

---

## 11. Risks & Assumptions

| Risk | Mitigation |
|---|---|
| Central free instance's bandwidth/compute cost scales with adoption but funding doesn't keep pace | Pursue multiple funding sources in parallel (8.3) before public launch, revisit continuously as usage grows, monitor spend in real time against available funding |
| Profanity/moderation false positives harm user trust | Default to warn-then-remove, not instant hard ban; leader override always available |
| Cross-platform codebase divergence (web/mobile/desktop UI drift) | Shared design system planned once UI phase begins; desktop wraps web client directly |
| Focus tracking perceived as invasive surveillance | Opt-in for webcam-based tier, mandatory in-app disclosure, no raw video ever leaves the client |
| Small open-source team can't sustain scope | MVP scope deliberately excludes enterprise-scale features (Section 1.4) |
