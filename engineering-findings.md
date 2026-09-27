# Engineering Findings and Proposed Rules

## Architecture

- `backend/app/main.py` creates the FastAPI application and registers the router from `backend/app/routes.py`. The route module currently owns the Pydantic response models, generated data, filtering, aggregation, and endpoint handlers.
- The API's `FinancialMovement` model is mirrored by `frontend/src/lib/financial-types.ts`. `frontend/src/App.tsx` fetches `/api/metrics`; `frontend/src/lib/financial-utils.ts` computes the displayed KPIs and monthly chart values in the browser. This duplication makes contract drift a concrete risk.
- `generate_mock_movements(seed=42)` supplies the metrics endpoints in `backend/app/routes.py`. The current API data is generated mock data, not a persisted financial feed.

## Testing

- Backend endpoint behavior is tested with `TestClient(app)` in `backend/tests/test_routes.py`; pytest and httpx are in `backend/requirements.txt`.
- Frontend financial calculations have Vitest coverage in `frontend/src/lib/financial-utils.test.ts`, alongside the utility they exercise. `frontend/package.json` defines `test`, `lint`, and `build` scripts.
- The verified baseline is 15 backend tests and 5 frontend tests passing; `npm run lint` also passes. Backend pytest currently emits a Starlette/httpx deprecation warning.

## Runtime and Documentation

- `docker-compose.yml` is the source of truth for the `frontend` and `backend` services and published ports 5173, 8000, and 5678. Port 5678 is exposed for debugpy by `backend/Dockerfile`, not an HTTP API.
- `frontend/vite.config.ts` reads `VITE_API_PROXY_TARGET` (falling back to `http://backend:8000`); Compose overrides it to the backend's published host port through `host.docker.internal`.
- `backend/app/routes.py` implements `/health`, which is a concrete backend smoke check. The Compose stack and README provide the supported launch command.

## Agent Workflow

- Root `AGENTS.md` requires agents to inspect `.agents/rules`, `.agents/skills`, and `memory-bank` before acting. None of those directories were present in the tracked tree at handover.
- `frontend/src/App.tsx` hardcodes `2024 - Full Year`, while the date generator in `backend/app/routes.py` anchors its generated dates to the current date. The mismatch is visible in the running response and should not be repeated as product context without rechecking.

## Proposed Rules

| Proposed rule | Evidence and rationale | Planned validation |
| --- | --- | --- |
| Keep API and frontend data contracts synchronized. When changing a backend response or query contract, inspect the Pydantic models in `backend/app/routes.py` and mirrored frontend types in `frontend/src/lib/financial-types.ts`; update the route and relevant frontend tests with the change. | The same movement fields are declared on both sides and consumed by the frontend. | Add project-specific smoke-check instructions to the Spanish README using the actual Compose ports and `/health`; verify the documented calls against the running services. |
| Test behavior in the owning layer. Add backend route behavior tests to `backend/tests/test_routes.py` with `TestClient`; add frontend calculation tests beside the utility in `frontend/src/lib/` using Vitest. Run the scripts declared in `frontend/package.json` and backend pytest. | These are the existing test locations, libraries, and package scripts. | Run focused API/frontend test commands after the documentation check and report baseline results. |
| Derive setup and service addresses from `docker-compose.yml` and `frontend/vite.config.ts`. Keep the proxy target, host-gateway alias, and published backend port synchronized; use `/health` to verify the backend. | Compose publishes backend port 8000 and configures the host-gateway route; the API implements `/health`. | Document and call the host URLs from the Compose mappings; don't describe debugpy port 5678 as an HTTP service. |
| Before work, follow root `AGENTS.md` and inspect existing project rules, skills, and memory. | Root guidance explicitly names these locations; their absence at handover was verified from the tracked tree. | The new rules will remain inside `.agents/rules` and future memory inside `memory-bank`, matching the project guidance. |

## Rule Validation

- `local-services-and-docs.md` guided a smoke-check addition to `README.es.md`. The documented `docker compose ps` and HTTP checks succeeded: frontend root returned HTTP 200 and `/health` returned `{"status":"ok"}`.
- `api-contracts-and-tests.md` guided `test_metrics_endpoint_returns_frontend_movement_contract` in `backend/tests/test_routes.py`, which checks the movement response keys and operation types.
- After that test was added, backend pytest passed (16 tests), frontend Vitest passed (5 tests), and `npm run lint` passed. Pytest continues to report the existing Starlette/httpx deprecation warning.