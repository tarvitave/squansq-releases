# Squansq

**Multi-Agent Development Platform** — orchestrate a team of AI coding agents working in parallel, each isolated in its own git worktree.

Instead of one Claude Code terminal doing tasks sequentially, Squansq lets you run many **WorkerBees** simultaneously, coordinated by **Mayor Lee** (an AI orchestrator), tracked on a **Kanban board**.

---

## What it does

- **WorkerBees** — ephemeral Claude Code agents, each with their own git branch and working directory
- **Mayor Lee** — an AI orchestrator that breaks work into tasks, dispatches agents, and monitors progress
- **Kanban Board** — visual workflow: Open → In Progress → Landed, with one-click dispatch
- **Standbys** — saved templates for repeatable tasks (security review, design audit, etc.)
- **Live Terminals** — watch every agent in real time, step through session replay frames
- **Cost Tracking** — activity dashboard, API key billing (pay-as-you-go via your Anthropic key)

---

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose
- An [Anthropic API key](https://console.anthropic.com/settings/keys) (pay-as-you-go, avoids subscription limits)
- Git installed on the host machine (used for worktree isolation)

---

## Quick Start

### 1. Clone this repo

```bash
git clone https://github.com/tarvitave/squansq-releases
cd squansq-releases
```

### 2. Configure environment

```bash
cp .env.example .env
```

Edit `.env`:

```env
# Path to the directory containing your projects
PROJECTS_PATH=/home/yourname/projects

# Random secret for JWT tokens — generate with: openssl rand -hex 32
JWT_SECRET=your-random-secret-here

# Port to access the UI (default 80)
PORT=80
```

### 3. Start Squansq

```bash
docker compose up -d
```

Open **http://localhost** (or your server IP) in your browser.

### 4. First-time setup

1. **Create an account** — click Register on the login screen
2. **Add your Anthropic API key** — click your email (top of sidebar) → paste your key → Save
3. **Create a workspace** — click the workspace dropdown → New Workspace → point it at your `PROJECTS_PATH`
4. **Add a project** — in the Projects section → Add Project → pick your repo from the auto-detected list
5. **Dispatch a task** — go to Kanban → click **+** → describe what you want built → Create & Dispatch

WorkerBees will start automatically and you can watch them in the Terminals tab.

---

## Authenticating Claude Code

WorkerBees run Claude Code inside the container. They will use your **Anthropic API key** automatically once you save it in account settings — no separate authentication needed.

If you prefer Claude Pro / OAuth login instead:

```bash
# Open a shell in the running container
docker compose exec server bash

# Run Claude and complete the OAuth login once
claude

# Exit — the auth is persisted in the claude_auth volume
exit
```

---

## Updating

```bash
docker compose pull
docker compose up -d
```

---

## Architecture

```
Browser (React + xterm.js)
        │  WebSocket + REST
        ▼
Node.js Server
  ├── Express API
  ├── WebSocket (real-time events)
  ├── node-pty (PTY sessions per WorkerBee)
  ├── Signal monitor (DONE: / BLOCKED: detection)
  └── SQLite (projects, agents, tasks, releases)
        │
        ▼
Claude Code (WorkerBee) × N
  └── Isolated git worktree per agent
      └── CLAUDE.md task injection
```

---

## Configuration reference

| Variable | Required | Description |
|----------|----------|-------------|
| `PROJECTS_PATH` | Yes | Host path mounted as `/projects` inside the container |
| `JWT_SECRET` | Yes | Secret for signing auth tokens — use a long random string |
| `PORT` | No | Host port for the UI (default: `80`) |

---

## License

Squansq is proprietary software. The Docker images are provided for use under the [Squansq Terms of Service](https://squansq.com/terms).
