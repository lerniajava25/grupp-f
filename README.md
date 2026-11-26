# KommunFix

Citizens report local issues to their municipality, with location, description and images.
Images are analysed by AI, stored privately in S3-compatible storage (Garage), and locations are
handled with PostGIS.

> **Status:** project foundation only. No features yet. The backend exposes `GET /api/health`.

## Repository layout

```
backend/            Spring Boot 3 (Java 21, Maven wrapper), package se.kommunfix
frontend/           Empty on purpose. The frontend owner chooses the stack
infra/garage/       Garage config + bucket setup guide
docker-compose.yml  PostGIS + Garage for local development
.env.example        Template for local config (no real secrets)
AGENTS.md           Architecture, TDD, test rule, security, approval rules (read this!)
CONTRIBUTING.md     Branches, commits, reviews
```

## Prerequisites

- Java 21 (JDK)
- Docker with Docker Compose v2
- Nothing else for the backend: Maven comes via the wrapper (`./mvnw`)

## Quick start

```bash
# 1. Configure local environment
cp .env.example .env
#    then fill in POSTGRES_PASSWORD, GARAGE_RPC_SECRET (openssl rand -hex 32)
#    and GARAGE_ADMIN_TOKEN (openssl rand -base64 32)

# 2. Start PostGIS + Garage
docker compose up -d

# 3. First time only: set up the Garage layout, bucket and backend key
#    see infra/garage/README.md, then put the S3 key into .env

# 4. Run backend tests
cd backend
./mvnw verify

# 5. Run the backend
./mvnw spring-boot:run
curl http://localhost:8080/api/health      # → {"status":"ok"}
```

Stop services with `docker compose down` (add `-v` to wipe all local data).

## How we work

- **TDD:** Red → Green → Refactor. Tests are never changed to make code pass. See [AGENTS.md](AGENTS.md).
- **Branches & commits:** `feature/`, `fix/`, `chore/` + Conventional Commits. See [CONTRIBUTING.md](CONTRIBUTING.md).
- **CI:** every PR and push to `main` runs `./mvnw verify` (and frontend checks once a frontend exists).
