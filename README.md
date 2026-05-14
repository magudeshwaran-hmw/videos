# ZenseAI.QI — AI-Powered QA Workflow Platform

ZenseAI.QI automates the entire QA lifecycle by integrating 10+ independent AI agents into a single unified product with a dynamic pipeline builder. From requirement analysis to test case generation, automation scripting, security auditing, and performance testing — all orchestrated through a modern web interface and a tenant-scoped Node/Express API gateway.

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│  FRONTEND (Next.js 16, React 19, port 3000)                         │
│  All UI · in-memory caches · Bearer-JWT to the Common Backend       │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │ HTTP / SSE / multipart
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│  COMMON BACKEND (Node 22 / Express 5 / Prisma 7 / PG, port 3001)    │
│  Auth · CRUD · agent-execution proxy · encrypted LLM-config storage │
│  /api/llm/complete · /api/kb/* · /api/ria/* · /api/agents/:code/*   │
└──┬───────────┬───────────┬───────────┬───────────┬───────┬──────────┘
   │           │           │           │           │       │
 ┌─▼────┐ ┌────▼────┐ ┌────▼────┐ ┌────▼────┐ ┌────▼────┐ ┌▼────────┐
 │Deep  │ │CaseGeni │ │Auto-    │ │ Other-  │ │ Secure  │ │Access.  │
 │Speci │ │  :8001  │ │PlayPilot│ │ Agents  │ │   -Xi   │ │  :8005  │
 │:8000 │ │         │ │  :8002  │ │  :8003  │ │  :8004  │ │         │
 └──────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘
 ┌──────┐ ┌─────────┐ ┌─────────────────┐ ┌─────────┐ ┌────────────┐
 │Game- │ │ Perf-Xi │ │ Knowledge Base  │ │   RIA   │ │   Defect   │
 │  Xi  │ │  :8008  │ │  :8009 + pgvec. │ │  :8010  │ │ Intel.:8011│
 │ :8006│ └─────────┘ └─────────────────┘ └─────────┘ └────────────┘
 └──────┘
```

## Services

| Service | Port | Stack | Purpose |
|-------|------|-------|---------|
| **Frontend** | 3000 | Next.js 16 / React 19 | Single web UI for all features |
| **Common Backend** | 3001 | Node 22 / Express 5 / Prisma 7 / PostgreSQL | Tenant API gateway: auth, CRUD, agent-execution proxy, encrypted LLM config, KB & RIA proxies |
| **DeepSpeci** | 8000 | Python / FastAPI + LangGraph | AI-driven requirement analysis — gap, ambiguity, inconsistency detection |
| **CaseGeni** | 8001 | Python / FastAPI | Transforms requirements into structured, execution-ready test cases |
| **Auto-PlayPilot** | 8002 | Node / Express + MCP / Playwright | Test automation script generation |
| **Other Agents (legacy)** | 8003 | Next.js 14 | Legacy bundled defect / predictive / test-opt / knowledge-graph |
| **Secure-Xi** | 8004 | Python / FastAPI | Security vulnerability detection and threat analysis |
| **Accessibility Intelligence** | 8005 | Node / Express + axe-core | WCAG accessibility auditing and AI remediation |
| **Game-Xi** | 8006 | Python / FastAPI | Gamification testing — player journeys, progression, balance |
| **Perf-Xi** | 8008 | Python / FastAPI | Performance bottleneck detection — greenfield, brownfield, script automation |
| **Knowledge Base** | 8009 | Python / FastAPI + pgvector | RAG service — document ingestion, vector search, domain knowledge |
| **RIA Assistant** | 8010 | Python / FastAPI | In-app conversational helper with dynamic RAG (frontend talks to it via the Common Backend) |
| **Defect Intelligence** | 8011 | Python / FastAPI | Test failure root cause analysis (current implementation) |

### How traffic flows

The browser only ever talks to the Common Backend. The Common Backend
resolves the tenant's encrypted LLM credentials, injects them into upstream
agent requests, and relays JSON / SSE / multipart bodies as appropriate.
**Raw API keys never live in the browser.**

```
Browser ─► Common Backend (3001)
            ├─► /api/llm/complete           (OpenAI / Google / Anthropic / Azure Foundry)
            ├─► /api/kb/*                    → Knowledge Base (8009) + embedding injection
            ├─► /api/ria/chat[/stream]       → RIA (8010) + LLM-config injection
            └─► /api/agents/:code/{execute,stream,execute-multipart}
                                             → upstream agent service (8000-8011)
```

### Pipeline Data Flow

```
Requirements → DeepSpeci → CaseGeni → Auto-PlayPilot → Defect Intelligence
                                                      → Predictive Intelligence (parallel)
                                                      → Test Optimization (parallel)
                                                      → Knowledge Graph (parallel)
```

Secure-Xi, Accessibility Intelligence, and Perf-Xi run independently with no upstream dependencies.

## Prerequisites

### Required

| Tool | Version | Installation |
|------|---------|--------------|
| **Python** | 3.10+ | `brew install python` (macOS) · [python.org](https://python.org) |
| **Node.js** | 22+ (Common Backend uses ESM/tsx) | `brew install node` (macOS) · [nodejs.org](https://nodejs.org) |
| **npm** | 9+ | Included with Node.js |
| **Podman** | 4.0+ | `brew install podman` — runs both PostgreSQL containers |

> **Two PostgreSQL containers (both Podman-managed):**
> `make install` creates and starts each one for you.
> - `zenseai-backend-db` → port **5432**, db `zense_ai_qi` — Common Backend (auth, projects, pipelines, encrypted LLM configs)
> - `zenseai-knowledge-db` → port **5433**, db `zenseai_knowledge` — Knowledge Base (pgvector embeddings)

### Optional

| Tool | Purpose | Installation |
|------|---------|--------------|
| **Tesseract OCR** | DeepSpeci document parsing (PDF/image OCR) | `brew install tesseract` (macOS) · `sudo apt install tesseract-ocr` (Linux) |
| **Ollama** | Local LLM inference (no API key needed) | [ollama.ai](https://ollama.ai) |

### LLM API Key (at least one provider)

| Provider | Where to get a key |
|----------|--------------------|
| **Google Gemini** (recommended) | [aistudio.google.com](https://aistudio.google.com/app/apikey) |
| **Azure AI Foundry** | Your Azure OpenAI resource endpoint + key |
| **Anthropic Claude** | [console.anthropic.com](https://console.anthropic.com) |
| **OpenAI** | [platform.openai.com](https://platform.openai.com) |
| **Ollama** | No key required (runs locally) |

LLM credentials are entered through the in-app **Integrations** page once
the stack is running. The Common Backend stores them encrypted (AES-256-GCM)
and injects them server-side into agent requests.

## Getting Started

### 1. Clone the repository

```bash
git clone <repo-url>
cd zense-ai-qi-demo
```

### 2. Configure agent env (optional)

```bash
cp agents/.env.example   agents/.env       # legacy LLM keys for Python agents (optional now — backend resolves credentials)
```

**You do not need to touch `backend/.env` yourself.** Step 3 creates it
automatically with a freshly generated `ENCRYPTION_KEY`, JWT secrets, and a
`DATABASE_URL` that matches the Podman Postgres container it spins up.

### 3. Install all dependencies

```bash
make install
```

This will:
- Install Node dependencies for the Common Backend, frontend, and all Node-based agents
- Create Python virtual environments (`.venv/`) for each Python agent and install their packages
- Install Playwright browser binaries (Chromium)
- Start the **`zenseai-backend-db` Podman Postgres container** for the Common Backend (port 5432)
- Scaffold `backend/.env` from `backend/.env.example`, generate fresh
  `ENCRYPTION_KEY` / `JWT_ACCESS_SECRET` / `JWT_REFRESH_SECRET`, and set
  `DATABASE_URL` to match the Podman container creds (skipped if
  `backend/.env` already exists)
- Run `prisma migrate` and `prisma seed` on the Common Backend database
- Start the **`zenseai-knowledge-db` Podman Postgres container** (pgvector) for the Knowledge Base (port 5433) and restore its DB if needed

### 4. Start all services

```bash
make dev
```

This launches everything in the foreground with prefixed log streams. Press `Ctrl+C` to stop.

| Service | URL |
|---------|-----|
| Frontend | http://localhost:3000 |
| Common Backend | http://localhost:3001 |
| DeepSpeci | http://localhost:8000 |
| CaseGeni | http://localhost:8001 |
| Auto-PlayPilot | http://localhost:8002 |
| Other Agents (legacy) | http://localhost:8003 |
| Secure-Xi | http://localhost:8004 |
| Accessibility | http://localhost:8005 |
| Game-Xi | http://localhost:8006 |
| Perf-Xi | http://localhost:8008 |
| Knowledge Base | http://localhost:8009 |
| RIA Assistant | http://localhost:8010 |
| Defect Intelligence | http://localhost:8011 |
| Backend Postgres | localhost:5432 (Podman: `zenseai-backend-db`, db `zense_ai_qi`) |
| KB Postgres (pgvector) | localhost:5433 (Podman: `zenseai-knowledge-db`) |

### 5. Stop all services

If anything was left running from a previous session:

```bash
make kill
```

This kills processes on every service port (3000, 3001, 8000–8011) and `tsx watch` workers. Both Podman Postgres containers (`zenseai-backend-db`, `zenseai-knowledge-db`) are left running by design — stop them manually if you need to:

```bash
podman stop zenseai-backend-db zenseai-knowledge-db
```

## Project Structure

```
├── frontend/                 # Next.js 16 — single web UI; talks only to backend
│   └── src/
│       ├── app/              # App Router routes (thin wrappers)
│       ├── modules/          # Domain-driven feature modules
│       ├── components/       # Shared UI components
│       └── lib/              # apiClient · agentClient · llmClient · kbClient
├── backend/                  # Node 22 / Express 5 / Prisma 7 — Common Backend
│   ├── src/
│   │   ├── modules/          # auth, project, pipeline, pipeline-run, agent,
│   │   │                     #  agent-output, llm-integration, llm, kb, ria, …
│   │   ├── orchestrator/     # step-runner (JSON/SSE/multipart relay)
│   │   ├── integrations/     # ai/* (HTTP/SSE/multipart relays) · llm-completion/*
│   │   ├── middlewares/      # auth, validate, error
│   │   └── shared/           # crypto, params, response helpers
│   └── prisma/               # schema · migrations
├── agents/
│   ├── .env.example          # Optional shared env for Python agents
│   ├── deepspeci/            # Python/FastAPI + LangGraph
│   ├── casegeni/             # Python/FastAPI
│   ├── auto-playpilot/       # Node/Express + MCP/Playwright
│   ├── other-agents/         # Next.js 14 (legacy bundle)
│   ├── secure-xi/            # Python/FastAPI
│   ├── Accessibility/        # Node/Express + axe-core
│   ├── game-xi/              # Python/FastAPI
│   ├── perf-xi/              # Python/FastAPI
│   ├── knowledge/            # Python/FastAPI + pgvector
│   ├── ria/                  # Python/FastAPI (proxied via backend)
│   └── defect-intelligence/  # Python/FastAPI
├── docs/                     # Architecture, guidelines, migration status
├── install.sh                # Dependency installer (make install)
├── dev.sh                    # Service launcher (make dev)
├── kill.sh                   # Service killer (make kill)
├── Makefile                  # Convenience commands
└── podman-compose.yml        # Knowledge Base Podman container
```

## Tech Stack

**Frontend:** Next.js 16 · React 19 · TypeScript · Tailwind CSS 4 · Radix UI · Lucide Icons

**Common Backend:** Node 22 · Express 5 · TypeScript (ESM) · Prisma 7 · PostgreSQL · bcryptjs · jsonwebtoken · zod · multer · AES-256-GCM

**Backend Agents:** FastAPI (Python) · Express (Node.js) · Next.js 14 · LangGraph · axe-core · Playwright

**LLM Providers:** Google Gemini · OpenAI · Anthropic Claude · Azure AI Foundry · Meta Llama · Mistral · Cohere · Ollama

**Infrastructure:** PostgreSQL × 2 (system + Podman pgvector) · FAISS · ChromaDB

## Make Commands

| Command | Description |
|---------|-------------|
| `make install` | Install all dependencies (Common Backend npm + Prisma migrate + seed, frontend npm, Python venvs, Playwright, KB Podman DB) |
| `make dev` | Start all services (frontend + Common Backend + all agents + Knowledge Base Postgres) |
| `make kill` | Kill all running services (leaves the KB Podman container running) |

## Migration Status

This repo is the result of an in-progress migration from a frontend-only
localStorage app to a tenant-scoped Node backend. See
[`docs/migration-status.md`](docs/migration-status.md) for the live punch
list of what's done, what's deferred, and what's open.

## License

Proprietary — All rights reserved.
"# videos" 
"# videos" 
