# Repo notes for agents

## What this is

Tailspin Shelter workshop app (fictional dog shelter) — Astro SSR client in
`app/client` + Flask/SQLite API in `app/server`. The app is read-only: the API
only exposes `GET /api/dogs` and `GET /api/dogs/<id>`.

## Running it here

```sh
docker compose -f docker-compose.base44.yml up -d
```

- `web` (host port 3000) is `astro dev`, so page edits hot-reload.
- `api` (host port 5100) is the Flask dev server with the reloader on; edits to
  `app/server` restart it.
- The Astro server renders on the server and calls the API over the compose
  network via `API_SERVER_URL=http://api:5100`. Without that variable it falls
  back to `http://localhost:5100`, which does not resolve inside the `web`
  container — that is a setup problem, not an app bug.

## Quirks

- `app/client/package-lock.json` is out of sync with `package.json`
  (`npm ci` fails: "Missing: typescript… from lock file"). Compose therefore
  runs `npm install --no-save` so the committed lockfile stays unchanged.
  `npm run build`/`npm ci` will keep failing until the lockfile is regenerated.
- `app/server/dogshelter.db` is committed and already seeded (20 breeds,
  100 dogs). The API's startup runs `utils/seed_database.py`, which is
  idempotent and skips seeding when rows exist.
- `app/server/app.py` calls `app.run(port=5100)`, which binds 127.0.0.1 only —
  inside Compose it is started with `flask --app app run --host 0.0.0.0`.
- Vite's dev server host allowlist is fed by the `__VITE_ADDITIONAL_SERVER_ALLOWED_HOSTS`
  env var passed through Compose (the sandbox host contains a rotating sandbox
  id), so no source config override is needed.

## Verifying

```sh
curl -s localhost:3000/ | grep dog-name        # rendered cards from the API
curl -s localhost:5100/api/dogs?per_page=1     # API alone
docker compose -f docker-compose.base44.yml exec -T api python -m unittest -v test_app
```
