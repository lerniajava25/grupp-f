# Frontend

Intentionally empty. **The frontend owner chooses the stack** (framework, build tool, test runner,
linter). Nothing has been scaffolded so that choice stays open.

When you add the frontend:

- Put it in this folder with a `package.json` and a committed lockfile (`package-lock.json`).
- CI (`.github/workflows/ci.yml`) activates automatically once `frontend/package.json` exists. It runs
  `npm ci`, then `npm run lint`, `npm test` and `npm run build`, each **only if that script exists**.
- Never put API keys, S3 credentials or AI keys in frontend code or frontend env files. Everything in
  the browser is public. Talk to the backend (`/api/...`) instead.
- Update the root `README.md` quick start with how to run the frontend.
