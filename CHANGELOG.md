# Changelog

All notable changes to CHKT are documented in this file.

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
