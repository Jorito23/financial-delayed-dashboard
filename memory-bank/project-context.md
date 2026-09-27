# Project Context

Last verified: 2026-09-27

## Product Overview

This repository contains a financial metrics dashboard. The React application requests movements from `/api/metrics`, then calculates KPI totals and monthly chart data in the browser. The FastAPI backend exposes health and metrics endpoints, including filters, summaries, comparisons, and alerts (`../frontend/src/App.tsx`, `../frontend/src/lib/financial-utils.ts`, `../backend/app/routes.py`).

The backend currently generates deterministic mock movements with seed 42; it does not load them from a financial database (`../backend/app/routes.py`). Do not describe these values as live or persisted financial records.

## Technology Stack

- Frontend: TypeScript, React 19, Vite 8, Tailwind CSS 4, Recharts 3, and lucide-react. Dependencies and scripts are declared in `../frontend/package.json`; pure financial calculations are tested with Vitest.
- Backend: Python 3.13 container, FastAPI, Uvicorn, Pydantic models, and debugpy. Dependencies are listed in `../backend/requirements.txt`; endpoint behavior is tested with pytest and `TestClient`.
- Local runtime: Docker Compose defines `frontend` and `backend`. It publishes frontend port 5173, API port 8000, and debugpy port 5678 (`../docker-compose.yml`, `../backend/Dockerfile`). Vite reads `VITE_API_PROXY_TARGET`; Compose directs `/api` to `host.docker.internal:8000` through a `host-gateway` alias (`../frontend/vite.config.ts`, `../docker-compose.yml`).
- Frontend scripts: `npm test`, `npm run lint`, and `npm run build` (`../frontend/package.json`). Backend test command in the container: `pytest -q`.

## Current State

- `docker compose up --build -d` succeeded in the verified environment. Both services were running; the frontend root and backend `/health` returned HTTP 200. Compose publishes the URLs documented in `../README.es.md` when those host ports are available.
- The configured frontend proxy was verified with `GET /api/metrics` through port 5173; it returned valid JSON containing 360 movements. Frontend build, 5 Vitest tests, and ESLint passed after the proxy adjustment.
- Last verified suites: backend 16 passed, frontend 5 passed, and frontend ESLint passed. Pytest reports a Starlette/httpx deprecation warning.
- Known display mismatch: the frontend hardcodes `2024 - Full Year`, but the API generates dates relative to the current date. On the verification date, its 360 movements spanned `2025-09-02` to `2026-08-28` (`../frontend/src/App.tsx`, `../backend/app/routes.py`). Recheck the generated range before describing it.
- Follow-up grounded in current behavior: reconcile the displayed period with the API range. Separately, investigate the existing Starlette/httpx warning. No broader product roadmap is established by the tracked code or handover.
- Runtime network finding: in this Codespace's Docker bridge, containers resolve each other but connections between their bridge IPs time out. The backend remains healthy locally, and the frontend can reach its published port through the host gateway. Compose configures the Vite proxy to use that verified route.
- Workspace caveat: the requested GitHub fork could not be created through the available integration (HTTP 403). This local checkout tracks the upstream repository; commits are local and have not been pushed to a fork.

For repository-specific editing and runtime guidance, first read `../AGENTS.md` and the files under `../.agents/rules/`.