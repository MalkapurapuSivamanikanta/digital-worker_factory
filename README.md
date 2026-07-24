# Digital Worker Factory

A multi-layered platform for cloud operations automation. This repository contains the
backend orchestration (Chandra), a FastAPI runtime surface, LangGraph agents, and a
Next.js operations console used to observe, triage, and approve automated remediations.

**Architecture summary**
- **Backend (Chandra)**: LangGraph-based orchestrator under `src/chandra/` that runs
	deterministic detectors, ranks findings with a composer that calls Bedrock, and
	routes actions through a deterministic decision_router → action_executor → escalation
	topology.
- **FastAPI runtime**: Lightweight HTTP/WebSocket surface at the repository root
	(e.g. `fastapi_app.py`, `run.py`) which exposes intake endpoints and wires the
	Digital Worker multi-agent pipeline for real-time interactions with the frontend.
- **Frontend (Next.js)**: A modern React + TypeScript console in `frontend/` that hosts
	onboarding, a live ops dashboard, and the human approval center.

**Code flow (high level)**
- Observers/detectors (pure, deterministic boto3 paginators) run under
	`src/chandra/tools/` and emit findings.
- Findings are analyzed and ranked in `src/chandra/briefing/composer.py` (the only
	module permitted to call Bedrock). The LLM never invents findings — it provides
	ranking and narrative only.
- `decision_router` splits actions into `auto_fixed` (low-risk, may be auto-applied)
	and `pending_writes` (high-risk, require approval). Deterministic `action_executor`
	handles auto-fixes; `escalation` publishes pending actions to the approval queue.
- The `persist` node is the only place that writes to Postgres (via `src/chandra/db/`).

**Key directories**
- `src/chandra/` — backend orchestration, tools, graphs, briefing, escalation.
- `fastapi_app.py`, `app.py`, `run.py` — FastAPI surface and agent orchestrator.
- `frontend/` — Next.js operations console (onboarding, approval center, dashboard).
- `iac/` and `evals/` — infrastructure and evaluation manifests for seeded runs.
- `tests/` — pytest suite (unit + integration harnesses).

**Development & common commands**
Run these from the repository root:

```bash
make install        # install Python (and dev) deps
make db-up          # start Postgres (docker-compose)
make migrate        # alembic upgrade head
make check          # lint + type + test (quality gate)
```

Backend (run FastAPI service):

```bash
uv run fastapi_app.py
# or, if you prefer the provided CLI
make run
```

Frontend (from `frontend/`):

```bash
cd frontend
npm install
npm run dev
```

Run both services concurrently (example, in two terminals):

Terminal 1 — backend:

```bash
uv run fastapi_app.py
```

Terminal 2 — frontend:

```bash
cd frontend
npm run dev
```

Testing examples:

```bash
uv run pytest tests/unit/ -k "decision_router" -v
uv run pytest tests/unit/ --cov=src/chandra/briefing --maxfail=1 -q
```

**Contributing**
- Follow the branch and commit naming conventions in CLAUDE.md.
- Run `make check` before opening a PR. Keep PRs small and single-purpose.

For a complete, developer-focused description of the project architecture and
operational rules (Bedrock usage, AwsClientFactory, LangGraph topologies, and
quality gates), see CLAUDE.md.

---
Short and focused. Open an issue or ask for more detail if you'd like expanded
sections (detectors, graph topology, or frontend integration examples).