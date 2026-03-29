# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MiroFish is a multi-agent AI prediction engine. It extracts seed information from uploaded documents (PDF/TXT/MD), builds a knowledge graph via Zep Cloud, generates agent personas, runs social media simulations via OASIS, and produces analytical reports. The UI is a 5-step workflow: Graph Build → Environment Setup → Simulation → Report → Interaction.

## Commands

```bash
# Install all dependencies (Node + Python)
npm run setup:all

# Run both frontend and backend concurrently
npm run dev

# Run individually
npm run backend           # Flask on :5001
npm run frontend          # Vite dev server on :3000

# Build frontend for production
npm run build

# Backend dependency management (uses uv)
cd backend && uv sync

# Docker
docker compose up -d

# Tests (backend)
cd backend && uv run pytest
cd backend && uv run pytest path/to/test.py::test_name   # single test
```

## Architecture

**Backend**: Python/Flask on port 5001. Entry point: `backend/run.py`. App factory in `backend/app/__init__.py`.

**Frontend**: Vue 3 + Vite on port 3000. Proxies `/api` to backend. Uses D3 for graph visualization, Axios for HTTP (5-min timeout with retry).

**External Services**:
- LLM calls via OpenAI SDK (compatible with any OpenAI-format provider)
- Zep Cloud for knowledge graph storage, entity/edge queries, and memory
- OASIS (camel-ai) for social media simulation, run as subprocess

**Configuration**: All API keys and LLM settings in root `.env` file (see `.env.example`). Backend loads via `python-dotenv` in `backend/app/config.py`.

## Backend Structure

- `backend/app/api/` — Three Flask blueprints: `graph.py`, `simulation.py`, `report.py`
- `backend/app/services/` — Core business logic:
  - `ontology_generator.py` — LLM generates entity/edge type schema from text
  - `graph_builder.py` — Builds Zep knowledge graph from text + ontology
  - `zep_entity_reader.py` — Filters and enriches entities from graph
  - `oasis_profile_generator.py` — Creates agent personas from entities
  - `simulation_runner.py` — Runs OASIS social media simulation (subprocess)
  - `simulation_ipc.py` — Inter-process communication for simulation
  - `report_agent.py` — ReACT-based agent with tools (Search, InsightForge, Panorama, Interview)
  - `zep_tools.py` — Rich query tools for Zep graph
- `backend/app/models/` — `Project` (with status enum: CREATED→ONTOLOGY_GENERATED→GRAPH_COMPLETED) and `Task` (async task tracking)
- `backend/app/utils/` — LLM client wrapper, file parser, logging, retry, pagination helpers
- `backend/app/uploads/` — File storage for projects, simulations, reports

## Frontend Structure

- `frontend/src/views/` — Page components: Home, MainView (5-step process), SimulationView, ReportView, InteractionView
- `frontend/src/components/` — Step components (Step1GraphBuild through Step5Interaction), GraphPanel (D3), HistoryDatabase
- `frontend/src/api/` — Axios client with interceptors (`index.js`) and per-domain modules (`graph.js`, `simulation.js`, `report.js`)
- `frontend/src/router/index.js` — Routes: `/`, `/process/:projectId`, `/simulation/:simulationId`, `/report/:reportId`, `/interaction/:reportId`

## Key Technical Details

- Python 3.11-3.12 required (no 3.13+). Uses `uv` for dependency management.
- Node.js 18+ required.
- Long-running operations (ontology generation, graph building, simulation, report generation) are async with task polling.
- No persistent database — uses in-memory state + file-based persistence in `backend/app/uploads/`.
- CORS is open for all `/api/*` routes.
- The LLM response parsing in `report_agent.py` must handle reasoning model artifacts like `<think>` tags and markdown code fences in content fields.
- Primary language in codebase comments and UI is Chinese; READMEs available in both Chinese and English.
