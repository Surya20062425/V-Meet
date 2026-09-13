# OpenMeet v1 — Step-by-Step Build Process

Follow this in order. Each step builds on the one before it — don't jump ahead to SDK packaging before group calls work, don't start moderation before chat exists.

---

## Phase 1: Foundation (Weeks 1-2)

**Step 1 — Set up the repo and project infra**
*Assigned to: Lead*
```bash
# Create the repo
git init openmeet && cd openmeet
```
- Create the GitHub repo, add `README.md`, `LICENSE` (MIT or Apache 2.0), `CONTRIBUTING.md`.
- Set up a GitHub Project board with columns: Backlog / In Progress / Review / Done.

**Step 2 — Get a dev VM running**
*Assigned to: Lead*
- Sign up for Oracle Cloud "Always Free" tier, spin up a VM.
- Install Docker + Docker Compose on it.
```bash
curl -fsSL https://get.docker.com | sh
sudo apt install docker-compose-plugin
```

**Step 3 — Deploy the SFU locally first (not on the VM yet)**
*Assigned to: Media Engineer*
```bash
# LiveKit local dev server
curl -sSL https://get.livekit.io | bash
livekit-server --dev
```
- Confirm it runs and you can reach its admin/health endpoint.

**Step 4 — Get a 2-person call working using LiveKit's own demo client**
*Assigned to: Media Engineer*
- Don't write any custom code yet — use LiveKit's example app to confirm audio+video works end-to-end between two browser tabs.
- This validates your SFU setup before you build anything on top of it.

**✅ Phase 1 done when:** two browser tabs can see/hear each other through your locally-running SFU.

---

## Phase 2: Backend & Real Group Calls (Weeks 3-5)

**Step 5 — Scaffold the backend**
*Assigned to: Lead (backend)*
```bash
npx @nestjs/cli new backend
cd backend
npm install @nestjs/websockets @nestjs/platform-socket.io livekit-server-sdk
```

**Step 6 — Build the meeting-creation endpoint**
*Assigned to: Lead (backend)*
- `POST /meetings` → generates a room ID, returns a shareable join link (`yourapp.com/join/{roomId}`).
- This is where the "generate meeting URL" requirement from the PRD gets implemented.

**Step 7 — Build the join endpoint + auth token issuance**
*Assigned to: Lead (backend)*
- `POST /meetings/:id/join` → validates the room exists, issues a short-lived LiveKit access token scoped to that room.
- Add basic email/password auth (JWT) so users have accounts — keep it simple, no SSO yet.

**Step 8 — Stand up Postgres + Redis**
*Assigned to: Lead (backend)*
```yaml
# docker-compose.yml (add to what you have)
services:
  postgres:
    image: postgres
    environment:
      POSTGRES_PASSWORD: devpass
  redis:
    image: redis
```
- Schema: `users`, `meetings` tables. Keep it minimal — you can add columns later.

**Step 9 — Wire the signaling layer**
*Assigned to: Lead (backend)*
- Socket.IO events for join/leave/mute — broadcast state changes to everyone in a room.

**Step 10 — Test group calls (not just 2-person)**
*Assigned to: Media Engineer + Lead*
- Get 8-10 simultaneous participants working. This is your v1 cap — don't chase higher numbers yet.
- If quality degrades past a certain count, that's useful data for where your real cap should be.

**✅ Phase 2 done when:** you can create a room via the API, get a real join link, and have 8-10 people on a call reliably.

---

## Phase 3: Web Client Polish (Weeks 6-8)

**Step 11 — Scaffold the real web client**
*Assigned to: Frontend Engineer*
```bash
npm create vite@latest web -- --template react-ts
cd web
npm install livekit-client
```

**Step 12 — Build the pre-join screen**
*Assigned to: Frontend Engineer*
- Camera/mic device picker, self-preview, "Join" button hitting your Step 7 endpoint.

**Step 13 — Build the in-call view**
*Assigned to: Frontend Engineer*
- Video grid (responsive to participant count), controls bar (mute/camera/screen share/leave).

**Step 14 — Add screen share**
*Assigned to: Frontend Engineer*
```javascript
const stream = await navigator.mediaDevices.getDisplayMedia({ video: true });
```

**Step 15 — Handle error states**
*Assigned to: Frontend Engineer*
- No camera/mic permission, room not found, room full — each needs a clear message, not a silent failure.

**Step 16 — Deploy the web client**
*Assigned to: Frontend Engineer*
- Push to Vercel or Cloudflare Pages (free tier) — get a real public URL, even before launch, so you're testing against production infra early.

**✅ Phase 3 done when:** a stranger with no technical knowledge can open your link, join, and have a working call.

---

## Phase 4: Moderation (Weeks 9-10)

**Step 17 — Add text chat**
*Assigned to: Frontend Engineer (UI) + Lead (backend)*
- Socket.IO-based, room-scoped, not persisted beyond the meeting.

**Step 18 — Add the profanity filter**
*Assigned to: Lead (backend logic) + Frontend Engineer (UI)*
```javascript
import Filter from 'bad-words';
const filter = new Filter();
// on each message: check filter.isProfane(text) before broadcasting
```
- Implement warn-then-remove: first flag = warning shown to sender + logged; second flag = removed from the room.

**Step 19 — Build the leader's moderation view**
*Assigned to: Frontend Engineer*
- Simple list: flagged messages, sender, timestamp, a "re-admit" button if someone was removed.
- No need for a polished dashboard yet — a functional list is enough for v1.

**✅ Phase 4 done when:** a test message with a banned word gets flagged, the leader sees it, and a repeat offense removes the user.

---

## Phase 5: SDK Packaging (Weeks 11-12) — your key differentiator

**Step 20 — Extract the call UI into a standalone package**
*Assigned to: Media Engineer + Frontend Engineer*
- Pull the video grid + controls out of the main app into an importable component/module.

**Step 21 — Define the minimal integration API**
*Assigned to: Media Engineer + Frontend Engineer*
- Something like:
```javascript
import { OpenMeetCall } from '@openmeet/sdk';
<OpenMeetCall roomToken={token} onLeave={() => {}} />
```
- The goal from the PRD: a third-party developer should get a working call view in under ~20 lines of code.

**Step 22 — Document the token-issuing flow**
*Assigned to: Media Engineer*
- Write clear docs for how an external app's backend calls your API to get a room token, then hands it to the SDK.

**Step 23 — Publish the package**
*Assigned to: Media Engineer*
```bash
npm publish --access public
```
- Even a `0.1.0` early version is fine — this is what makes the project embeddable starting now, not "eventually."

**✅ Phase 5 done when:** you can build a tiny separate test app that embeds a working call using only the published package.

---

## Phase 6: Self-Hosting & Security (Weeks 13-14)

**Step 24 — Finalize the Docker Compose setup**
*Assigned to: Lead*
- One `docker-compose.yml` that brings up backend + SFU + coturn + Postgres + Redis together.
- Test it yourself on a completely fresh VM to make sure it actually works from a clean start.

**Step 25 — Write the self-hosting guide**
*Assigned to: Lead*
- Clone repo → configure `.env` → `docker-compose up` → done. Include TURN/domain/TLS setup steps.

**Step 26 — Run a security pass**
*Assigned to: Optional 4th (or Lead)*
```bash
# OWASP ZAP baseline scan against your staging deployment
docker run -t owasp/zap2docker-stable zap-baseline.py -t https://your-staging-url
```
- Confirm DTLS-SRTP is enforced, tokens are short-lived and room-scoped, auth endpoints are rate-limited.

**✅ Phase 6 done when:** a stranger can follow your docs and get their own working instance running from scratch.

---

## Phase 7: Beta & Launch (Weeks 15-16)

**Step 27 — Recruit a small beta group**
*Assigned to: Whole team*
- 5-10 real users/teams, ideally including at least one developer who'll try embedding the SDK.

**Step 28 — Fix what breaks**
*Assigned to: Whole team*
- Real usage always surfaces things load-testing doesn't — prioritize anything that breaks the core call experience first.

**Step 29 — Write launch materials**
*Assigned to: Lead*
- README with a clear demo GIF/video, a short "why this exists" pitch (lead with the embeddable SDK angle from the POV discussion).

**Step 30 — Launch**
*Assigned to: Lead*
- Post to Hacker News, r/selfhosted, r/opensource, relevant Discord/Slack communities for OSS/WebRTC.
- Link the self-hosting guide and SDK docs prominently — those are your two real hooks.

**✅ v1 done.**

---

## After Launch: What Comes Next

Once real usage and (ideally) some funding interest exist, return to the full PRD for: mobile/desktop clients, voice moderation, focus tracking (only if users actually ask for it), a centrally-hosted free instance, and the fully-scaled autoscaling infra. Build those as their own dedicated phases, not as a rush — the whole point of this trimmed path is to earn the right to build those next, with real users validating that they're worth building.
