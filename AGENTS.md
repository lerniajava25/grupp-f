# AGENTS.md: rules for AI coding agents (and humans)

This file is the single source of truth for how AI agents (Claude, Copilot, Codex, Cursor, …) work in
this repo. `CLAUDE.md` only points here. Read it fully before making changes.

## 1. What KommunFix is

Citizens report local issues (potholes, broken street lights, graffiti, …) to their municipality.
A report has a description, a location and one or more images. Images are analysed by an AI service
(e.g. to suggest a category), stored privately in S3-compatible storage, and locations are stored
and queried with PostGIS.

## 2. Architecture overview

```
 Browser ──HTTPS──▶  frontend/  (stack chosen by the frontend owner, not decided yet)
                          │  calls /api/...  (no secrets in the frontend)
                          ▼
                     backend/   Spring Boot 3, Java 21, package se.kommunfix
                      │      │            │
             JDBC/SQL │      │ S3 API     │ HTTPS
                      ▼      ▼            ▼
                PostGIS    Garage      AI analysis service
                (db)       (images,    (free provider, NOT chosen yet;
                           private)     backend-only API key)
```

| Part | Where | Notes |
|------|-------|-------|
| Backend API | `backend/` | Spring Boot 3.5, Java 21, Maven wrapper. Deps today: web, validation, actuator. |
| Frontend | `frontend/` | Empty on purpose. The frontend owner picks the stack. Do not scaffold one. |
| Database | `docker-compose.yml` → `db` | `postgis/postgis`. Schema/migrations not decided yet. |
| Object storage | `docker-compose.yml` → `garage` | Garage S3. Setup: `infra/garage/README.md`. |
| AI analysis | n/a | Provider undecided (should be a free service). Keep integrations provider-neutral behind an interface. Config via `AI_*` env vars. |
| CI | `.github/workflows/ci.yml` | Backend `./mvnw verify`; frontend job only once `frontend/package.json` exists. |

Current state: **foundation only**. The only endpoint is `GET /api/health` → `{"status":"ok"}`.

## 3. Development workflow: TDD (Red → Green → Refactor)

All production code is written test-first:

1. **Red:** write a small test for the next piece of behaviour. Run it and **see it fail** for the
   right reason (e.g. 404, assertion failure, not a compile error in unrelated code).
2. **Green:** write the *minimum* production code to make that test pass. Run the full suite.
3. **Refactor:** clean up code and tests while keeping everything green.
4. Repeat in small steps. Commit at green.

Run backend tests: `cd backend && ./mvnw verify`

### The test rule (non-negotiable)

- **Never modify, delete, skip, disable or weaken a test to make code pass.** Change the production
  code instead. This includes `@Disabled`, `assumeTrue`, commenting out, loosening assertions,
  `-DskipTests`, `-Dmaven.test.failure.ignore`, excluding tests in build config, or catching the
  failure.
- Tests may only change when **requirements change**, and only with **explicit human approval**.
  State in the PR which tests changed and which requirement change justifies it.
- Adding *new* tests is always welcome.
- If a test seems wrong, stop and ask a human. Don't "fix" it yourself.

## 4. Security & privacy

- **No secrets in the frontend.** No API keys, S3 credentials or AI keys in frontend code, frontend
  env files or anything shipped to the browser. The browser only talks to `/api/...`.
- **No secrets in git.** Real values live in `.env` (git-ignored). `.env.example` contains
  placeholders only. If a secret is committed by mistake: tell a human, rotate it, and don't just
  delete the line.
- **Images are private by default.** Never enable public/website access on the Garage bucket. Serve
  images through authorised backend endpoints or short-lived presigned URLs.
- **GDPR.** Reports may contain personal data: names, contact details, precise locations, and faces
  or licence plates in images. Collect only what is needed, document why, keep it no longer than
  needed, support deletion, never log personal data or image contents, and only send to the AI
  service the minimum required (prefer providers/settings that don't retain or train on data).
  Strip EXIF/GPS metadata from images unless explicitly needed.
- Validate all input on the backend (`spring-boot-starter-validation`); never trust the client.
- Expose only the actuator endpoints that are needed (currently `health`).

## 5. Changes that require explicit human approval

An agent must **stop and ask** before:

- Adding, removing or upgrading **dependencies** (Maven, npm, Docker images, GitHub Actions).
- Any **database schema** change (tables, columns, indexes, migrations, extensions).
- Changing **CI** (`.github/workflows/`) or build configuration that affects CI.
- Changing **security configuration** (auth, CORS, CSP, actuator exposure, bucket policies, secrets
  handling, `garage.toml`).
- Changing, deleting or skipping **tests** (see §3).

When approval is needed: explain what and why in the PR description and wait.

## 6. Conventions

- Branches, commits and reviews: see `CONTRIBUTING.md` (`feature/`, `fix/`, `chore/`, Conventional
  Commits, review required, never push to `main`).
- Java package root: `se.kommunfix`, organised by feature (e.g. `se.kommunfix.health`,
  `se.kommunfix.report`) rather than by layer.
- Formatting follows `.editorconfig`.
- Keep PRs small and focused; fill in the PR template checklist.
