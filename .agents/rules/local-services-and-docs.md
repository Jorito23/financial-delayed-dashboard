# Local Services and Documentation

## Scope

Local setup, service troubleshooting, and documentation of runtime behavior.

## Repository Evidence

- `docker-compose.yml` defines `frontend` and `backend`, publishing host ports 5173, 8000, and 5678.
- `frontend/vite.config.ts` reads `VITE_API_PROXY_TARGET` and falls back to `http://backend:8000`.
- `docker-compose.yml` sets that target to `http://host.docker.internal:8000` and maps `host.docker.internal` to `host-gateway`.
- `backend/Dockerfile` exposes 5678 for debugpy; it is not an HTTP endpoint.
- `backend/app/routes.py` implements `GET /health` and the metrics API.

## Guidance

- Use `docker compose up --build` as the documented local startup path. Confirm actual state and host port mappings with `docker compose ps`.
- Keep the Compose `VITE_API_PROXY_TARGET`, `extra_hosts`, and published backend port in sync. The Compose override intentionally routes through the host gateway; the Vite fallback remains `backend:8000` for other environments.
- To document or troubleshoot health, check the frontend root and `GET /health` on the backend's published port. Do not describe port 5678 as an API.
- If a published port is unavailable, report the conflict and inspect the Compose mapping before suggesting an alternate URL; do not claim the default URL is serving successfully without an HTTP check.
- Keep setup and product claims in docs tied to tracked configuration or code. In particular, verify date ranges against the generator before describing the UI's hardcoded period label as actual API coverage.