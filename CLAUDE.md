# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

A small Express REST API (users + health check) backed by an in-memory store, used as the starter project for a Claude Code course.

## Commands

- `npm run dev` — start the API with `node --watch` on http://localhost:3000 (`PORT` overrides)
- `npm test` — run all tests with the built-in Node test runner (`node --test`)
- `node --test tests/users.test.js` — run one test file; add `--test-name-pattern="GET /health"` to run a single test
- `npm run lint` — ESLint (`eslint:recommended`) over the whole repo

CI (`.github/workflows/ci.yml`, Node 22) runs `npm run lint` then `npm test` on every push and PR; both must pass.

## Conventions

- CommonJS only: `require` / `module.exports`, not `import` / `export` (ESLint is configured with `sourceType: "script"`).
- Tests use `node:test` + `node:assert` + `supertest`, not Jest or Mocha. Import `app` from `../server` and call `request(app)`; never start a real server in tests.
- Route handlers never touch the `users` array directly; all data access goes through functions exported from `db/store.js`.
- Error responses are JSON shaped `{ error: "<message>" }` with the matching status (400 for validation, 404 for missing resources); creates return 201.
- Unused `req`, `res`, `next`, or `_` params are fine — the lint rule ignores them.
- The task for this repo is setting up Claude (`CLAUDE.md`, `.claude/settings.json`, `NOTES.md`); do not change app code unless explicitly asked.

## Architecture

- `server.js` builds the Express app, mounts routers, and exports `app`. It only calls `listen()` when run directly (`require.main === module`), which is what lets tests import it.
- `routes/` holds one `express.Router()` file per resource, mounted in `server.js` under its path (`/users`, `/health`). A new resource = new route file + one `app.use` line.
- `db/store.js` is a module-level in-memory array that resets on restart. State is shared across all tests in a process, so tests should not assume exact counts or IDs after a `POST`.
- `.env` holds real config and must never be read or committed; `.env.example` documents the variables (currently only `PORT`).
