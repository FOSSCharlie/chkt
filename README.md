<p align="center">
  <img src="public/chkt-logo-small.png" width="96" alt="CHKT logo">
</p>

# CHKT

A stupidly simple todo list.

CHKT is a small, dependency-free todo list. Every task has a due
date, active tasks are sorted soonest-first, and completing a task
records the completion date.

> ⚠️ Vibe coded with [Claude](https://claude.ai).

See [CHANGELOG.md](./CHANGELOG.md) for version history.

## Features

- ✨ Clean, minimal interface
- 🌓 Dark/Light mode with system preference detection
- 📅 Every task has a due date; tasks are sorted nearest-due first
- 🚦 Colour-coded urgency (overdue / due today / due soon / later)
- ✅ Completion date recorded automatically when you check a task off
- 📱 Installable PWA (works offline, "Add to Home Screen")
- 🔄 Server-side storage - same task list on every device

## Storage

CHKT stores tasks server-side in a single JSON file
(`tasks.json`), written by a small Node/Express backend. There is
no database and no accounts, just one file. Because storage lives
on the server rather than in the browser, every device that opens
the same URL sees the same task list.

## Quick Start

### Running locally without Docker

Requires Node.js 20+.

```bash
git clone https://github.com/FOSSCharlie/chkt.git
cd chkt
npm install
node server.js
```

Open <http://localhost:3000>. Tasks are written to `tasks.json`,
whose location defaults to `/data/tasks.json` but can be overridden
with the `DATA_FILE` environment variable, e.g.
`DATA_FILE=./tasks.json node server.js`.

> Note: service workers require HTTPS (localhost is exempt for
> testing), so offline support and "Add to Home Screen" will only
> work once this is served over TLS on a real domain.

### Using Docker

1. Pull from GitHub Container Registry (after the Actions workflow
   has published it at least once)

   ```bash
   docker pull ghcr.io/fosscharlie/chkt:latest
   sudo mkdir -p /opt/chkt
   docker run -p 4080:3000 -v /opt/chkt:/data ghcr.io/fosscharlie/chkt:latest
   ```

2. Or build locally

   ```bash
   docker build -t chkt .
   sudo mkdir -p /opt/chkt
   docker run -p 4080:3000 -v /opt/chkt:/data chkt
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
         - /opt/chkt:/data
   ```

   ```bash
   sudo mkdir -p /opt/chkt
   docker compose up -d
   ```

> **Note:** since v1.7.0 the container runs as a non-root user (UID
> 1000) rather than root. After creating `/opt/chkt` (or if you're
> upgrading an existing install), run:
> ```bash
> sudo chown -R 1000:1000 /opt/chkt
> ```
> so the container can write to the mounted data folder. Without
> this, the container will fail to create or update `tasks.json`.

Open <http://localhost:4080>. Tasks are stored server-side in a JSON
file inside `/opt/chkt`, so every device that opens this URL sees
the same task list.

## Automatic image publishing

`.github/workflows/docker-publish.yml` builds and pushes the image to
`ghcr.io/<your-github-username>/chkt` on every push to `main`, using
the repo's built-in `GITHUB_TOKEN` - no extra secrets needed. Make
sure the resulting package is set to public in the repo's Packages
tab if you want to `docker pull` it without authenticating.

## Project Structure

```
chkt/
├── public/                 # Static frontend served by the app
│   ├── index.html
│   ├── manifest.json
│   ├── service-worker.js
│   ├── chkt-logo-small.png
│   └── chkt-logo-large.png
├── server.js                # Node/Express backend, reads/writes tasks.json
├── package.json
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .gitignore
└── .github/workflows/docker-publish.yml
```

## Development

Keep it dumb, keep it simple - if you're writing complex code to
solve a simple problem here, you're probably doing it wrong.

## License

CHKT is free software, licensed under the MIT License. See the
[LICENSE](./LICENSE) file for the full text.

## Disclaimer

This software is provided "as is", without warranty of any kind,
express or implied - see the LICENSE file for the exact legal
terms. You use it entirely at your own risk.
