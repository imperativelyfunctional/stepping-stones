# Local setup

You only need [Docker Desktop](https://www.docker.com/products/docker-desktop/) (or Docker Engine + Compose v2).

```bash
git clone https://github.com/imperativelyfunctional/stepping-stones.git
cd stepping-stones
docker compose up --build
```

The first build takes a few minutes. When it settles:

| Service  | URL                          | Port |
| -------- | ---------------------------- | ---- |
| Frontend | http://localhost:5173        | 5173 |
| Backend  | http://localhost:8000/docs   | 8000 |
| Postgres | `localhost`                  | 5432 |

The frontend page should show **Backend health: ok (database: connected)**.
If it does, all three services are talking to each other.

Code changes in `backend/` and `frontend/` reload automatically.

## Handy commands

```bash
docker compose up -d --build     # run in the background
docker compose logs -f backend   # follow one service's logs
docker compose down              # stop everything
docker compose down -v           # stop and wipe the database
```

## Troubleshooting

- **Port 8000 already allocated:** copy `.env.example` to `.env`, set `BACKEND_PORT=8001`, then run `docker compose up --build` again. The frontend follows automatically, and the API is then at http://localhost:8001/docs.
- **Port 5432 or 5173 in use:** stop whatever uses it (often a local Postgres), or change the left-hand port in `docker-compose.yml`.
- **Different DB credentials:** copy `.env.example` to `.env` and edit it.
