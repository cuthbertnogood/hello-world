---
name: cloud-agent-runbook
description: Run, test, and operate this repository in Cloud Agent environments. Use for setup, dev servers, env vars, feature flags, credentials/mocking, CI parity, and area-specific test workflows.
---

# Cloud agent runbook (this repository)

Use this skill at the start of any task that touches runtime behavior, tests, or local workflows. Prefer **discovering truth from the repo** over assuming a stack; the sections below list what to look for and sensible defaults when files are missing.

## Current repository baseline

- Today this workspace may be **minimal** (for example, only a top-level `README.md`). Treat missing manifests (`package.json`, `pyproject.toml`, `go.mod`, `Dockerfile`, and so on) as a signal to **inventory the tree** and **update this skill** once real areas exist.

## Fast path: first 2 minutes in a Cloud agent

1. **Orient**

   ```bash
   git rev-parse --show-toplevel && git status -sb
   ls -la
   ```

2. **Read human entry points** (if present): `README.md`, `CONTRIBUTING.md`, `docs/`, `Makefile`, `justfile`, `Taskfile.yml`.

3. **Detect stack signals** (presence-only checks; do not assume):

   - Node: `package.json`, `pnpm-lock.yaml`, `yarn.lock`, `package-lock.json`
   - Python: `pyproject.toml`, `requirements.txt`, `Pipfile`, `poetry.lock`
   - Go: `go.mod`
   - Rust: `Cargo.toml`
   - JVM: `pom.xml`, `build.gradle*`
   - Containers: `docker-compose*.yml`, `compose.yaml`, `Dockerfile*`
   - IaC / cloud: `terraform/`, `pulumi.*`, `serverless.yml`

4. **Secrets and configuration**

   - Look for `.env.example`, `.env.sample`, `*.env.template`, `config/*.example`, or provider-specific samples.
   - In Cloud agents, **never commit secrets**. Prefer test doubles: env vars pointing at local mocks, `NO_AUTH=1`-style dev flags only if the codebase explicitly supports them, or recorded contract tests.

5. **Install and run** using the **area-specific** commands you discover. If nothing exists yet, stop after inventory and propose the smallest scaffold the task needs.

## Cloud agent workflow expectations (this environment)

- Work on a feature branch (for this org: `cursor/<descriptive-name>-f438` off `master` unless instructed otherwise).
- Commit in logical chunks with clear messages; push with `git push -u origin <branch-name>`.
- Run the same checks you expect CI to run **before** claiming done (see “CI parity” in each area once defined).

## Organized by codebase area

Each subsection follows the same pattern: **Locate → Configure → Run → Test → CI parity**. If a path or tool is not in the repo, skip that subsection and note the gap in the skill’s maintenance section.

### Area: Repository metadata and scripts

**Locate:** `README.md`, `Makefile`, `justfile`, `scripts/`, `.github/workflows/`.

**Configure:** Usually none.

**Run:** Doc-only repos have nothing to start.

**Test:** If GitHub Actions exist, mirror them locally where possible (for example, `act` if available, otherwise run the underlying commands from the workflow YAML).

**CI parity:** Open `.github/workflows/*` and copy the **job commands** verbatim into your local shell when feasible.

### Area: Single package / app (Node example)

**Locate:** `package.json` at repo root or in a workspace package.

**Configure:**

- Install: prefer the lockfile’s package manager (`npm ci`, `pnpm install --frozen-lockfile`, or `yarn install --immutable`).
- Env: copy `.env.example` → `.env` for local-only values; keep secrets out of git.

**Run:** Typical patterns:

- `npm run dev` / `pnpm dev` / `yarn dev`
- `npm start` for production-like local runs

**Test:**

- Unit: `npm test`, `pnpm test`, or `vitest` / `jest` as defined in `package.json`.
- Lint/types: `npm run lint`, `npm run typecheck` when present.

**Feature flags:** Search the codebase for `launchdarkly`, `split.io`, `unleash`, `growthbook`, `posthog`, `statsig`, or env keys like `FEATURE_*`, `FLAG_*`, `VITE_*`, `NEXT_PUBLIC_*`. Prefer **explicit test values** documented in `.env.example` or test setup files.

### Area: Single package / app (Python example)

**Locate:** `pyproject.toml` or `requirements*.txt`.

**Configure:**

- Prefer a **venv** in Cloud agents: `python -m venv .venv && source .venv/bin/activate`.
- Install: `pip install -e ".[dev]"` or `pip install -r requirements.txt` as documented.

**Run:** `uvicorn`, `flask run`, `python -m <module>`, or `docker compose up` if defined.

**Test:** `pytest`, `tox`, or `nox` as configured.

**Feature flags:** Same discovery approach as Node; also check `pydantic` settings modules and `django.conf.settings`.

### Area: Backend API service

**Locate:** service directory, OpenAPI/Swagger specs, `docker-compose` service names.

**Configure:** Database URLs, queue URLs, object storage credentials. For agents, prefer **dockerized dependencies** from compose files or in-memory alternatives documented upstream.

**Run:** Compose stacks often use `docker compose up --build <service>`.

**Test:**

- Contract: `pytest`/`jest` against a running server, or `schemathesis` if present.
- Quick smoke: `curl` against `/health` or `/ready` if routes exist.

### Area: Frontend / web client

**Locate:** `vite.config.*`, `next.config.*`, `nuxt.config.*`, `angular.json`, `apps/web`.

**Configure:** `VITE_*`, `NEXT_PUBLIC_*`, and API base URLs. Use `.env.local` only if gitignored and documented.

**Run:** `pnpm dev` / `npm run dev` / `next dev` as defined.

**Test:**

- Unit/component: `vitest`, `jest`, `npm run test`.
- E2E: `playwright test`, `cypress run` — may require a **running backend**; start services in order documented in compose or README.

### Area: Mobile (if present)

**Locate:** `android/`, `ios/`, `flutter`, `react-native.config.js`.

**Configure:** SDKs and emulators are often heavy in Cloud agents; prefer **unit tests** and **CI-emulated** flows unless the task explicitly requires simulators.

**Test:** `./gradlew test`, `xcodebuild test`, `flutter test` as applicable.

### Area: Infrastructure as code

**Locate:** `terraform/`, `cdk/`, `pulumi.*`.

**Configure:** Cloud credentials are usually **not** available to agents; prefer `terraform validate`, `terraform fmt -check`, and policy tests that do not need real providers, unless the task supplies credentials.

**Test:** Static analysis and `plan` in sandboxed accounts only when explicitly allowed.

## Authentication and “logging in”

There is no universal recipe; discover the mechanism:

- **OAuth/OIDC:** look for provider keys and redirect URLs in `.env.example` and integration tests for token exchange.
- **API keys:** header names are usually documented in `README` or OpenAPI `securitySchemes`.
- **Session cookies:** inspect E2E tests for how they seed a session.

For automated testing, prefer **fixtures** provided by the repo (factory functions, test users, mocked auth middleware). If the repo lacks these, add the smallest **test-only** bypass only when product owners agree it is safe.

## Feature flags and runtime toggles

1. Search code and config for vendor SDKs and `FEATURE_` / `FLAG_` prefixes.
2. Read `.env.example` and test harness setup (`vitest.setup`, `conftest.py`, `jest.setup.js`).
3. Set flags **exactly** the way the application reads them (env vs remote config service). In CI, flags are often forced via env in the workflow YAML—mirror that locally.

## Concrete testing workflows (templates)

Copy the block for each area that exists in the repo; delete blocks for stacks you do not have.

**Node single package**

```bash
<package-manager> install
<package-manager> run lint
<package-manager> test
```

**Python**

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt  # or: pip install -e ".[dev]"
pytest -q
```

**Docker compose integration**

```bash
docker compose up -d <deps>
<package-manager> test:e2e
docker compose down -v
```

**GitHub Actions parity**

```bash
# Replace with commands extracted from .github/workflows/ci.yml
```

## When this skill is incomplete

If install scripts, ports, or flag names are undocumented:

1. Derive them from source (entry `main`, dev server config, compose ports).
2. **Update this skill** in the same PR when you add or change a workflow, so the next agent does not rediscover the same details.

## Maintaining and evolving this skill

Whenever you learn a new **repro command**, **mock**, **flag**, **port**, or **CI secret name** (not the value), append or correct this file:

- **Add** a bullet to the relevant **area** section with the exact command and working directory.
- **Add** env vars to a short table in that area (`VAR` — purpose — safe default for agents).
- **Remove** dead instructions to avoid contradictory runbooks.
- Keep **Cloud agent constraints** in mind (no interactive browser logins unless the environment supports them; prefer headless flows).

If the repository grows a monorepo layout, consider splitting **large** reference material into `.cursor/skills/cloud-agent-runbook/references/` and keep `SKILL.md` as the concise checklist agents load first.
