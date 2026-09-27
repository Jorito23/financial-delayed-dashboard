# API Contracts and Tests

## Scope

Changes to API payloads, query parameters, or frontend financial calculations.

## Repository Evidence

- FastAPI response models and endpoint handlers are in `backend/app/routes.py`.
- The frontend movement contract is mirrored in `frontend/src/lib/financial-types.ts` and consumed from `frontend/src/App.tsx`.
- Backend endpoint tests use `TestClient(app)` in `backend/tests/test_routes.py`.
- Frontend calculations are in `frontend/src/lib/financial-utils.ts`, with Vitest coverage in `frontend/src/lib/financial-utils.test.ts`.

## Guidance

- When an API response or query contract changes, inspect both the backend model/handler and the corresponding frontend type or request. Update each affected side together; do not infer one contract from the other.
- Add or adjust backend behavior coverage in `backend/tests/test_routes.py` for changed routes and filters.
- Add or adjust calculation coverage beside the frontend utility in `frontend/src/lib/` when derived financial values change.
- Run backend tests with `docker compose exec -T backend pytest -q` and frontend tests with `docker compose exec -T frontend npm test`.
- Preserve the current distinction between generated mock data and persisted financial data unless the implementation itself changes.