# Contributing to KommunFix

Read [AGENTS.md](AGENTS.md) first. It describes the architecture, TDD workflow, the test rule and
security/GDPR requirements that apply to everyone (humans and AI agents).

## Branches

Never commit or push directly to `main`. Create a short-lived branch from an up-to-date `main`:

| Prefix | Use for | Example |
|--------|---------|---------|
| `feature/` | New functionality | `feature/report-image-upload` |
| `fix/` | Bug fixes | `fix/health-content-type` |
| `chore/` | Tooling, config, docs, dependencies | `chore/project-foundation` |

Use lowercase and hyphens. One topic per branch.

## Commits: Conventional Commits

Format: `<type>(optional scope): <short summary in imperative mood>`

```
feat(report): accept image upload for a report
fix(health): return JSON content type
test(report): cover missing location validation
chore: add docker-compose for PostGIS and Garage
docs: explain Garage bucket setup
```

Common types: `feat`, `fix`, `test`, `refactor`, `docs`, `chore`, `ci`, `build`.
Breaking changes: add `!` after the type (`feat!: …`) and explain in the body.

## Workflow

1. `git switch main && git pull`
2. `git switch -c feature/<topic>`
3. Work test-first (Red → Green → Refactor). Commit at green.
4. `cd backend && ./mvnw verify` must pass locally.
5. Push and open a pull request to `main`. Fill in the PR template checklist.
6. **At least one approving review from another team member is required** before merging, and CI
   must be green. Authors don't merge their own unreviewed PRs.
7. Prefer squash-merge; delete the branch afterwards.

## Things that need explicit approval

Dependencies, database schema, CI, security configuration and any change to existing tests need
explicit human approval in the PR. See [AGENTS.md §5](AGENTS.md#5-changes-that-require-explicit-human-approval).
