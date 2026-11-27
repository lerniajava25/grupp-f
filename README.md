# KommunFix

> **Status:** project foundation only. No features yet. The backend exposes `GET /api/health`.

## The idea

KommunFix is a **national web app** where citizens report problems in public spaces, such as
potholes, broken streetlights or fallen trees, simply by **taking a photo**.

The app does the heavy lifting. It reads the image's location, lets AI suggest a title, description
and category, works out **which municipality** is responsible from official boundaries, and sends
the case to the right recipient. Citizens don't need to know which municipality or department to
contact, and municipalities receive structured, well-described reports.

## User flow

1. **Photograph** the problem (or upload an existing photo).
2. The image is **stored privately in Garage (S3)**.
3. **EXIF/GPS metadata** is extracted from the image (location, time).
4. **Spring AI** analyses the image and **suggests a title, description and category**.
5. The user **confirms or adjusts the position on a map** (required if the image has no GPS data).
6. **PostGIS** looks up the **responsible municipality** from municipality boundaries.
7. The user **reviews and approves** the case (and can edit the AI suggestions).
8. The case is **sent by email** to the municipality (or to a **simulated recipient** in dev/MVP).
9. The case is **saved with a history** of status changes and events.

## Architecture

A Spring Boot backend organised into clearly separated services (packages under `se.kommunfix`),
a frontend whose stack is not decided yet, PostGIS for data and geography, and Garage for images.

```
 Browser ──▶ frontend/ (stack TBD) ──/api/...──▶ backend/ (Spring Boot, se.kommunfix)

 Backend services and what they talk to:

   User ──────────┐
   Case ──────────┤
   Geo ───────────┼──▶ PostgreSQL + PostGIS  (users, cases, history, boundaries)
   Municipality ──┘
   Storage ───────────▶ Garage S3            (private images)
   Metadata ──────────▶ EXIF/GPS from the uploaded image
   AI ────────────────▶ Spring AI ──▶ AI provider (free, TBD)
   Notification ──────▶ Email / simulated recipient
```

| Service | Responsibility |
|---------|----------------|
| **User** | Citizen identity/session, consent, own cases. Minimal personal data. |
| **Case** | Core domain: creating, approving and storing cases, statuses and history. |
| **Storage** | Upload/download of images to/from Garage. Private bucket, presigned URLs. |
| **AI** | Image analysis via Spring AI → suggested title, description, category. Provider-neutral (free provider, not chosen yet). |
| **Geo** | Coordinates, map position validation, spatial queries in PostGIS. |
| **Municipality** | Municipality boundaries and contact details; "which municipality owns this point?". |
| **Metadata** | Extracts EXIF/GPS from images and strips sensitive metadata before storage/sharing. |
| **Notification** | Sends the approved case by email, or to a simulated recipient. |

More detail, including the development rules, is in [AGENTS.md](AGENTS.md).

## Tech stack

| Area | Choice |
|------|--------|
| Backend | Java 21, Spring Boot 3.5, Maven (wrapper) |
| Database | PostgreSQL + PostGIS |
| Image storage | Garage (S3-compatible) |
| AI | Spring AI (model/provider TBD, should be free) |
| Frontend | **TBD.** The frontend owner chooses the stack (incl. map library) |
| CI/CD | GitHub Actions |
| Local dev | Docker Compose |

## Team roles

| Role | Focus |
|------|-------|
| **Backend / business logic** | Case, User, Municipality and Notification services, API design, database |
| **Frontend / GeoData** | Frontend stack and UI, map and position confirmation, Geo, municipality boundary data |
| **AI / integration / CI/CD** | AI and Metadata services, Garage/Storage integration, CI/CD pipeline, Docker |

## MVP vs later

**MVP**

- Upload a photo → stored privately in Garage
- EXIF/GPS extraction, with manual map positioning as fallback
- AI suggestions for title, description and category, editable by the user
- Municipality lookup from boundaries in PostGIS
- User approves → case sent to a **simulated recipient** (or a test mailbox)
- Case saved with status history

**Later**

- Real email/API delivery to municipalities, per-municipality routing
- Logged-in "my cases" with status updates
- Municipality-side view to receive and update cases
- Duplicate detection (same problem reported nearby)
- Multiple images per case, accessibility improvements, multiple languages

## Known limitations

- **Municipality integration:** municipalities have no common intake API. Email (or a simulated
  recipient) is used for now, and contact details must be maintained manually.
- **Missing GPS:** many images lack location data (stripped by the phone, app or screenshot). The user
  must then set the position manually on the map.
- **AI misreads:** the AI can suggest the wrong category or description. Suggestions are only
  suggestions, and the user always reviews and approves before sending.
- **GDPR:** images can contain faces, licence plates and homes, and the GPS position can reveal where
  someone lives. Images are private by default, metadata is minimised, personal data is never logged,
  and AI analysis should use providers that don't retain or train on the data. See
  [AGENTS.md](AGENTS.md#4-security--privacy).

## Quick start

**Prerequisites:** IntelliJ IDEA (Community or Ultimate), JDK 21, Docker with Docker Compose v2.
Maven comes via the wrapper (`./mvnw`), so there is nothing else to install.

### Open in IntelliJ

1. **File → New → Project from Version Control…** (or **Get from VCS** on the welcome screen),
   paste the repository URL and clone.
2. IntelliJ detects the root `pom.xml` and imports the Maven project automatically
   (if asked, choose **Trust Project** / **Load Maven Project**).
3. Pick **JDK 21**: **File → Project Structure → Project → SDK** (use *Download JDK…* if you
   don't have one, e.g. Eclipse Temurin 21).
4. In the run configuration dropdown (top right), choose **Backend** and press Run.
   Open <http://localhost:8080/api/health> → `{"status":"ok"}`.
5. Run **All tests** to run the whole test suite with Maven.

The shared run configurations live in `.run/` and are committed; the rest of `.idea/` is ignored.

### Local services (database and image storage)

```bash
# 1. Configure local environment
cp .env.example .env
#    fill in POSTGRES_PASSWORD, GARAGE_RPC_SECRET (openssl rand -hex 32)
#    and GARAGE_ADMIN_TOKEN (openssl rand -base64 32)

# 2. Start PostGIS + Garage
docker compose up -d

# 3. First time only: set up the Garage layout, bucket and backend key
#    see infra/garage/README.md, then put the S3 key into .env
```

Stop services with `docker compose down` (add `-v` to wipe all local data).

### From the command line

Run from the repository root:

```bash
./mvnw verify                          # build + all tests (same as CI)
./mvnw -pl backend spring-boot:run     # run the backend
curl http://localhost:8080/api/health  # → {"status":"ok"}
```

## Repository layout

```
pom.xml             Root Maven aggregator (modules: backend)
mvnw, .mvn/         Maven wrapper, run from the repo root
.run/               Shared IntelliJ run configurations (Backend, All tests)
backend/            Spring Boot 3 (Java 21), package se.kommunfix
frontend/           Empty on purpose. The frontend owner chooses the stack
infra/garage/       Garage config + bucket setup guide
docker-compose.yml  PostGIS + Garage for local development
.env.example        Template for local config (no real secrets)
```

## How we work

- **[AGENTS.md](AGENTS.md):** architecture, TDD (Red → Green → Refactor), the test rule, security/GDPR
  and what needs human approval. Applies to humans and AI agents.
- **[CONTRIBUTING.md](CONTRIBUTING.md):** branches (`feature/`, `fix/`, `chore/`), Conventional
  Commits, required review.
