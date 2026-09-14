#V-Meet v1 — Step-by-Step Build Process

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
