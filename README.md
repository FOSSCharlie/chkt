# CHKT

A stupidly simple todo list, sorted by due date.

CHKT is a small, dependency-free todo list. Every task has a due
date, active tasks are sorted soonest-first, and completing a task
records the completion date.

## Features

- ✨ Clean, minimal interface
- 🌓 Dark/Light mode with system preference detection
- 📅 Every task has a due date; tasks are sorted nearest-due first
- 🚦 Colour-coded urgency (overdue / due today / due soon / later)
- ✅ Completion date recorded automatically when you check a task off
- 📱 Installable PWA (works offline, "Add to Home Screen")
- 🚀 Fully static - no database, no backend, no accounts

## Storage

CHKT stores its data entirely in the browser (`localStorage`). There
is no server-side database and no API - it's a static site. This
means todos are local to whichever browser/device you use it from.

## Quick Start

### Running locally without Docker

Since this is a fully static site, any static file server works:

```bash
git clone https://github.com/FOSSCharlie/chkt.git
cd chkt
python3 -m http.server --directory public 8080
```

Open <http://localhost:8080> in your browser.

> Note: service workers require HTTPS (localhost is exempt for
> testing), so offline support and "Add to Home Screen" will only
> work once this is served over TLS on a real domain.

### Using Docker

1. Pull from GitHub Container Registry (after the Actions workflow
   has published it at least once)

   ```bash
   docker pull ghcr.io/fosscharlie/chkt:latest
   docker run -p 4080:3000 -v chkt-data:/data ghcr.io/fosscharlie/chkt:latest
   ```

2. Or build locally

   ```bash
   docker build -t chkt .
   docker run -p 4080:3000 -v chkt-data:/data chkt
   ```

3. Docker Compose

   ```yaml
   services:
     chkt:
       image: ghcr.io/fosscharlie/chkt:latest
       container_name: chkt
       restart: unless-stopped
       ports:
         - "4080:3000"
       volumes:
         - chkt-data:/data

   volumes:
     chkt-data:
   ```

   ```bash
   docker compose up -d
   ```

Open <http://localhost:4080>. Tasks are stored server-side in a JSON
file inside the `chkt-data` volume, so every device that opens this
URL sees the same task list.

## Automatic image publishing

`.github/workflows/docker-publish.yml` builds and pushes the image to
`ghcr.io/<your-github-username>/chkt` on every push to `main`, using
the repo's built-in `GITHUB_TOKEN` - no extra secrets needed. Make
sure the resulting package is set to public in the repo's Packages
tab if you want to `docker pull` it without authenticating.

## Project Structure

```
chkt/
├── public/                 # Static app served by the container
│   ├── index.html
│   ├── manifest.json
│   ├── service-worker.js
│   ├── chkt-logo-small.png
│   └── chkt-logo-large.png
├── nginx.conf              # Serves public/, with correct cache headers
├── Dockerfile
├── docker-compose.yml
└── .github/workflows/docker-publish.yml
```

## Development

Keep it dumb, keep it simple - if you're writing complex code to
solve a simple problem here, you're probably doing it wrong.
