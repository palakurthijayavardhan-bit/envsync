# EnvSync — Environment Configuration Sync & Missing Variable Validator
Catches missing, invalid and mismatched env vars across Local/Staging/Production **before** deploy, without ever exposing secrets.

## Run (Node 18+, zero dependencies)
    cd backend && npm test       # unit tests
    cd backend && npm start      # API + dashboard at http://localhost:4000
    node backend/bin/envsync.js check --demo     # pre-push demo: exits 1 => "Push blocked."
    node backend/bin/envsync.js report --demo | example --demo
    node backend/bin/envsync.js check --dir .envsync   # real files: local.env staging.env production.env schema.yml
    git config core.hooksPath .githooks        # enable the pre-push hook

## Architecture
`backend/src/core.js` (pure logic: parsers, validate, compare, generate, report) -> used by `backend/src/server.js` (REST, node:http), `backend/bin/envsync.js` (CLI) and tests. `frontend/index.html` is the dashboard.

## API (JSON)
GET /api/environments (redacted: presence + fingerprint) · POST /api/validate · POST /api/compare · POST /api/generate-env-example · GET /api/report · POST /api/prepush.
POST bodies may include `{envs, schema}`; omitted = demo data.

## Security approach
Secrets (schema `secret: true` or name matches API_KEY/SECRET/PASSWORD/TOKEN/PRIVATE_KEY/DATABASE_URL) are shown only as `[SECRET PRESENT]`; messages/logs/reports contain variable names, never values. Comparison uses SHA-256 fingerprints (8-hex prefix). Server binds to localhost; run CLI in CI so raw values never leave the machine. K8s Secret values are discarded at parse time. Never pass secrets as CLI args (shell history) - use files.

## Docker / Kubernetes
`parseCompose` reads `environment:` lists/maps (bare `- KEY` = inherited); `parseK8s` reads ConfigMap data, Secret keys (redacted) and container `env` names. Library functions - wire into the CLI as needed.

## CI
See `.github/workflows/envsync.yml`: push -> validate -> deploy job runs only if validation passes.

## Stack & run (Express + React/TS/Tailwind)
    cd backend  && npm install && npm start        # API on :4000 (npm test needs no install)
    cd frontend && npm install && npm run dev      # UI on :5173, proxies /api to :4000
    cd frontend && npm run build                   # then the backend also serves frontend/dist on :4000
`frontend/legacy-dashboard.html` is the earlier zero-dependency dashboard (open it while the backend runs via a static server).
