---
name: Onboarding
about: Get the Stepping Stones stack running locally with Docker Compose
title: "Onboarding: run the Stepping Stones stack with Docker Compose"
labels: onboarding, good first issue
---

## Goal

Get the full stack (Postgres, FastAPI, React) running locally with `docker compose` and confirm the three services can talk to each other. Expected time: 30–45 minutes, mostly the first build.

## Prerequisites

- [ ] Docker Desktop (or Docker Engine + Compose v2) is installed and running. Check with `docker compose version`.
- [ ] You have access to the repo and have read the README overview and the "How we work" section.

## Steps

1. [ ] Clone the repo and start the stack:
   ```bash
   git clone https://github.com/imperativelyfunctional/stepping-stones.git
   cd stepping-stones
   docker compose up --build
   ```
   The first build takes a few minutes.
2. [ ] In another terminal, run `docker compose ps`. `db`, `backend` and `frontend` should all be running, and `db` should be healthy.
3. [ ] Check the API: `curl localhost:8000/api/health` should return `{"status":"ok","database":"connected"}`. Also open http://localhost:8000/docs and confirm the interactive docs load.
4. [ ] Check the frontend: open http://localhost:5173. The page should show **Backend health: ok (database: connected)**.
5. [ ] Check live reload. Change the heading in `frontend/src/App.jsx` and confirm the browser updates without a rebuild. Then edit `backend/app/main.py` (for example, add a field to the health response) and confirm the API reloads. Revert both changes.
6. [ ] Shut down cleanly with `docker compose down`. Know what `docker compose down -v` does: it also wipes the database.

## If something fails

- **Port 8000 already allocated:** copy `.env.example` to `.env`, set `BACKEND_PORT=8001`, and re-run. The API is then at http://localhost:8001/docs, and the frontend follows automatically.
- **Port 5432 or 5173 in use:** stop whatever is using it (often a local Postgres), or change the left-hand port in `docker-compose.yml`.
- **Frontend says "backend unreachable":** run `docker compose logs backend` and check that `db` is healthy.
- **Anything else:** run `docker compose logs -f <service>` and post the output in the team channel.

## Definition of done

- [ ] Steps 1–6 pass on your machine.
- [ ] Comment on this issue with your OS, Docker version and a screenshot of the frontend showing "Backend health: ok".
- [ ] If you hit a problem that isn't in `docs/SETUP.md`, open a small `docs/…` PR adding it to the Troubleshooting section (branch `docs/<issue#>-short-name`, PR body starting with `Closes #N`).
