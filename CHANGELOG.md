# Changelog

All notable changes to CHKT are documented in this file.

## 1.7.1 - 2026-07-21

### Fixed
- Docker build was failing (`addgroup -g 1000` exit code 1) because
  `node:20-alpine` already ships with a built-in `node` user at UID/GID
  1000 — the Dockerfile was trying to create a second, conflicting
  user at the same ID. Now reuses the image's existing `node` user
  instead of creating a new one. No change to the UID (still 1000),
  so the `sudo chown -R 1000:1000 /opt/chkt` step from 1.7.0 is
  unaffected.

## 1.7.0 - 2026-07-21

### Security
- Dockerfile rebuilt as a multi-stage build: the final image no
  longer includes the npm CLI (only `node` is needed at runtime),
  which removes a large set of vulnerable transitive dependencies
  bundled with npm (`tar`, `glob`, `minimatch`, `cross-spawn`,
  `sigstore`, `brace-expansion`, `ip-address`, `diff`, etc.) from the
  shipped image entirely.
- Added `apk update && apk upgrade` at build time so the image picks
  up currently available OS package patches (OpenSSL/`libssl3`,
  `libcrypto3`) instead of whatever was baked into the base image
  when it was published.
- Container now runs as a non-root user (fixed UID 1000) instead of
  root. **If you're upgrading an existing install**, run
  `sudo chown -R 1000:1000 /opt/chkt` on the host so the container
  can still write to the data folder.
- Added a weekly scheduled rebuild (Mondays 06:00 UTC) to the GitHub
  Actions workflow, so OS-level patches keep getting picked up even
  when there's no app code change to trigger a push.
- `PUT /api/tasks/:id` now only applies a whitelist of fields
  (`text`, `dueDate`, `completed`, `completedDate`) instead of
  applying the entire request body onto the stored task, closing a
  mass-assignment gap where a client could overwrite a task's `id` or
  inject arbitrary fields.

## 1.6.1 - 2026-07-21

### Changed
- Shortened the tagline from "A stupidly simple todo list, sorted by
  due date." to "A stupidly simple todo list." — updated in the app
  footer, `package.json`, `manifest.json`, and README.

## 1.6.0 - 2026-07-20

### Changed
- Delete button is now a small red dot instead of an X icon in a
  square outline.
- Deleting a task now requires confirmation: clicking the dot turns
  it into a "Confirm" pill; a second click deletes the task.
  Clicking anywhere else, pressing Escape, or letting a background
  sync happen cancels the pending delete and reverts to the dot.

## 1.5.0 - 2026-07-20

### Added
- New `GET /api/version` endpoint that reads the version straight from
  `package.json`.
- Footer credit line now shows the running version (e.g. "CHKT
  v1.5.0: ...") and updates automatically on every release — no more
  manually editing the version number in the HTML.

## 1.4.0 - 2026-07-20

### Added
- Task list now auto-refreshes when a tab/window regains focus or
  becomes visible again (e.g. switching from your phone back to a
  browser tab), plus a 10-second background poll as a backstop for
  tabs left open and idle. Fixes devices appearing out of sync until
  a manual page reload.
- Background refreshes are skipped while a task is being edited, so
  they never interrupt typing, and skip re-rendering if nothing
  actually changed.

## 1.3.1 - 2026-07-20

### Fixed
- Date input's calendar icon was invisible in dark mode; added
  `color-scheme` so the browser renders native date-picker controls
  to match the active theme

### Changed
- Footer's Dark/Light and Clear Completed links reduced to 50% of
  their previous size
- CHKT credit line in the footer ("CHKT: A stupidly simple todo
  list, sorted by due date.") now bold

## 1.3.0 - 2026-07-20

### Changed
- Docker Compose now uses a bind mount (`/opt/chkt:/data`) instead
  of a named volume, so task data lives at a fixed host path
- `docker-publish.yml` fixed to lowercase the repo owner before
  tagging the image, so it builds correctly regardless of the
  GitHub username's casing
- README brought in line with the current Node/Express backend:
  removed leftover references to the old static/nginx/localStorage
  setup, corrected the local dev instructions, and updated the
  project structure listing
- Added a vibe-coded-with-Claude credit line

### Fixed
- Service worker was serving a stale cached copy of `/api/tasks`
  after adding, editing, or completing a task, so changes only
  appeared after a manual page refresh. API requests now always go
  to the network; only static assets are cached

## 1.2.0 - 2026-07-20

### Changed
- Header simplified to icon only; app name and tagline moved into
  the footer
- Footer redesigned: Dark mode / Clear Completed on one row, the
  CHKT credit + tagline + Import/Export links centered on the row
  below, with "CHKT" linking to the GitHub repo
- Tasks are now edited by clicking directly on the task text; the
  separate edit button was removed
- While editing, clicking or tapping anywhere outside the field
  saves the change (same as pressing Enter); the save/cancel
  buttons were removed
- Checkbox replaced with a smaller tick-icon button; both the
  checkbox and delete icons reduced to about half their previous
  size, keeping the same bordered-square style
- "Soon" urgency badge color changed to match the app icon's
  yellow (`#FBC02D`)
- Tagline updated to "A stupidly simple todo list, sorted by due
  date."

## 1.1.0 - 2026-07-19

### Added
- Node/Express backend (`server.js`) storing tasks in a single
  JSON file, so the same task list appears on every device that
  opens the app
- REST API: `GET/POST /api/tasks`, `PUT/DELETE /api/tasks/:id`,
  `POST /api/import`
- Dockerfile and docker-compose.yml for running CHKT as a
  container, with a named volume for persistent task storage
- GitHub Actions workflow to build and publish the image to GitHub
  Container Registry on every push to `main`

### Changed
- Frontend rewired from `localStorage` to call the new backend API
  for every add, edit, complete, delete, and import/export action

## 1.0.0 - 2026-07-19

### Added
- Initial release: a static, dependency-free todo list PWA
- Every task requires a due date; active tasks sort soonest-first
- Colour-coded urgency badges (overdue / today / soon / later)
- Completion date recorded automatically when a task is checked off
- Dark/light theme toggle, JSON import/export, installable as a PWA
- Storage in browser `localStorage`
