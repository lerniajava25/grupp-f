## What & why

<!-- What does this PR change, and why? Link the issue: Closes #123 -->

## How to test

<!-- Steps a reviewer can follow -->

## Checklist

- [ ] Branch is `feature/…`, `fix/…` or `chore/…` and targets `main`
- [ ] Commits follow Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `test:`, `refactor:` …)
- [ ] Written test-first (Red → Green → Refactor); new behaviour is covered by tests
- [ ] `./mvnw verify` (from the repo root) passes locally (and frontend lint/test/build if touched)
- [ ] **No existing tests were modified, deleted or skipped** (if they were: requirement change + human approval linked below)
- [ ] No secrets, keys or real personal data committed; nothing secret reaches the frontend
- [ ] Images stay private; personal data handled per GDPR notes in `AGENTS.md`
- [ ] Docs updated (README / AGENTS.md / `.env.example`) if setup or behaviour changed

### Needs explicit human approval? (tick and explain if yes)

- [ ] New/changed dependencies
- [ ] Database schema change
- [ ] CI / build pipeline change
- [ ] Security configuration change
- [ ] Test changes (modified/removed tests)

<!-- If any box above is ticked, explain here and wait for approval before merging. -->
