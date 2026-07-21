# CHKT Security Review

**Date:** 2026-07-21

Reviewed: `server.js`, `public/index.html`, `Dockerfile`,
`docker-compose.yml`, `.dockerignore`/`.gitignore`,
`.github/workflows/docker-publish.yml`, `package.json` (v1.6.1).

## The big picture first

CHKT is built as a **single-user, self-hosted tool for a trusted
network** — that design assumption is the single most important
thing to keep in mind below. It has no login screen by design. That's
fine *if* it's only ever reachable by you (behind your home network,
a VPN, or a reverse proxy with its own auth in front). It becomes a
real problem the moment the port is exposed to the open internet or
an untrusted network, because there is currently nothing standing
between "can reach the port" and "can read, add, edit, and delete
every task."

Everything below is ranked by real-world impact, assuming the
"trusted network only" model. If you're planning to expose this
publicly, the first finding is the one to fix before anything else.

---

## Findings

### 1. No authentication on the API (High, if exposed publicly)
`GET/POST/PUT/DELETE /api/tasks*` and `/api/import` accept requests
from anyone who can reach the port — no password, token, or session
check anywhere in `server.js`.

**Impact:** Full read/write/delete access to your task list for
anyone on the same network, or anyone on the internet if the port is
forwarded/exposed without a proxy in front.

**Fix options, roughly in order of effort:**
- Simplest: don't expose the port publicly. Keep it behind your home
  network or a VPN (Tailscale/WireGuard).
- Put a reverse proxy (Caddy, Nginx, Traefik) in front with HTTP Basic
  Auth if you do want it reachable from outside.
- If you want it built into the app itself, a lightweight
  session-cookie or API-key check added to `server.js` would do it —
  happy to add this if you want it.

### 2. No CSRF protection (Medium, same root cause as #1)
Because there's no auth token, any web page you happen to have open
in the same browser could silently fire a `POST`/`DELETE` to your
CHKT server if it's reachable from that browser (classic CSRF). Right
now nothing distinguishes "a request from your own app" from "a
request some other page tricked your browser into sending."

**Fix:** Once any form of auth/token exists (see #1), this mostly
resolves itself, since a third-party page won't have your credential
to attach.

### 3. `PUT /api/tasks/:id` allows overwriting any field (Medium)
```js
Object.assign(task, req.body);
```
This applies whatever the client sends directly onto the stored task
object — including fields you didn't intend to expose, like `id`. A
client could rewrite a task's `id` and break lookups, or add
unexpected extra fields into the stored JSON.

**Fix:** Whitelist the fields you actually intend to accept:
```js
const { text, dueDate, completed, completedDate } = req.body;
if (text !== undefined) task.text = text;
if (dueDate !== undefined) task.dueDate = dueDate;
if (completed !== undefined) task.completed = completed;
if (completedDate !== undefined) task.completedDate = completedDate;
```

### 4. `/api/import` writes unvalidated data straight to disk (Low)
```js
const { tasks } = req.body;
if (!Array.isArray(tasks)) { ... }
saveTasks(tasks);
```
Only checks that `tasks` is an array — not that each item looks like
a real task. A malformed import could corrupt the data file or leave
tasks with missing/invalid fields that break the frontend.

**Fix:** Validate each item has the expected shape (`text: string`,
`dueDate: string`, etc.) before saving, and generate a fresh `id` for
any item missing one rather than trusting client-supplied IDs.

### 5. No rate limiting (Low)
Nothing stops a script from hammering `/api/tasks` with rapid
requests. Each write does a full synchronous file rewrite
(`fs.writeFileSync`), so a burst of requests could cause disk
thrashing or (with concurrent writes) a lost update if two requests
race each other.

**Fix:** Not urgent for personal use. If you ever expose this beyond
a trusted network, a simple rate-limit middleware
(`express-rate-limit`) is a one-line addition.

### 6. Container runs as root (Low)
The `Dockerfile` never sets a non-root `USER`, so the Node process
inside the container runs as root by default.

**Fix:**
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --omit=dev
COPY server.js ./
COPY public ./public
RUN addgroup -S chkt && adduser -S chkt -G chkt && chown -R chkt:chkt /app
USER chkt
EXPOSE 3000
CMD ["npm", "start"]
```
Note: your bind mount is `/opt/chkt:/data` — if you add this, make
sure the `chkt` user can write to that mounted folder (you may need
to `chown` it on the host, or keep the data directory writable by the
container's UID).

### 7. No security headers (Low)
There's no `helmet` (or manual headers) setting things like
`X-Frame-Options`/`frame-ancestors`, so the app could technically be
loaded inside an `<iframe>` on another site for a clickjacking-style
UI overlay attack.

**Fix:** Optional for a personal tool, but a one-line addition if you
want it:
```js
const helmet = require('helmet');
app.use(helmet());
```

---

## What's already solid

- **No XSS via task text.** Every place task text is rendered uses
  `textContent`, never `innerHTML`, on the frontend — so a task named
  `<script>...</script>` just displays as literal text instead of
  executing. This was the thing I'd have expected to find broken, and
  it isn't.
- **Dependencies are clean.** The only runtime dependency is Express,
  pinned as `^4.18.2`. Running `npm audit` against that range
  resolves to Express 4.22.2 with **0 known vulnerabilities** — the
  caret range is correctly picking up patched versions automatically.
- **GitHub Actions permissions are properly scoped** —
  `contents: read`, `packages: write`, nothing broader than it needs.
- **`.dockerignore` excludes `.git`**, so your git history doesn't
  leak into image layers.
- **No secrets in the repo** — nothing hardcoded that shouldn't be
  there.

---

## Bottom line

For what CHKT is — a personal, self-hosted todo list you run on your
own network — there's nothing here that needs fixing today. The one
finding worth actually acting on is **#1: don't expose the port to
the open internet without something (a proxy with auth, or a VPN) in
front of it.** Everything else is small, cheap-to-fix hardening you
can pick up if/when you want it, not urgent problems.

## Addendum — OS and npm-tooling CVEs (v1.7.0)

A container scan (Arcane) surfaced a large batch of CVEs that the
original review above didn't cover, because `npm audit` only checks
the app's own declared dependencies (just Express) — it doesn't scan
the base OS image or the npm CLI's own bundled tooling. Both are real
gaps, both are now fixed:

- **`libcrypto3` / `libssl3` (OpenSSL, High/Medium/Low)** — patched by
  adding `apk update && apk upgrade` to the Dockerfile at build time,
  plus a weekly scheduled CI rebuild so future patches keep landing
  automatically even without an app code change.
- **`cross-spawn`, `glob`, `minimatch`, `tar`, `sigstore`,
  `@sigstore/core`, `brace-expansion`, `ip-address`, `diff` (High/
  Medium/Low)** — all of these are dependencies bundled with the npm
  CLI itself, not the app. The app only ever needs `node` to run
  (`node server.js`), not `npm start`. The Dockerfile now builds in a
  separate stage and removes the npm CLI entirely from the final
  image, which removes this whole category of findings rather than
  chasing each package version individually.
- Container also now runs as a **non-root user** (fixed UID 1000)
  instead of root, addressing finding #6 from the original review at
  the same time.

If you re-run Arcane (or Trivy/Grype) against the rebuilt image, the
OS-level findings should drop to whatever's still unpatched upstream
in Alpine at build time, and the npm-tooling findings should
disappear entirely since that tooling is no longer present in the
image.

**Verified 2026-07-21:** re-scanned the rebuilt image (v1.7.2) with
Arcane/Trivy — **0 vulnerabilities**, down from the original 48
(High/Medium/Low across `libcrypto3`, `libssl3`, and the npm-bundled
packages listed above).

