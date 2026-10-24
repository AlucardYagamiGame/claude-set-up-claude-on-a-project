# Notes

## What's in CLAUDE.md, and what I left out

`CLAUDE.md` has a one-line description and the commands I actually run: dev, test, lint, and how to run a single test file or test. It also lists conventions Claude can't safely guess: CommonJS rather than ESM, `node:test` + `supertest` rather than Jest, and all data access through `db/store.js`. The last section covers the architecture details that take reading several files to see. `server.js` only listens when run directly, which is what lets tests import `app`. Each resource gets its own router. The in-memory store is shared across tests, so tests shouldn't depend on exact counts or IDs.

I left out a file-by-file listing, the route list and the seed users, because Claude can find all of these by reading the code and they would go stale. I also left out generic advice ("write clean code"), the course instructions from the README and any secrets. Anything that doesn't change what Claude does in this repo only costs context on every session.

## Permission rules

- **allow:** `npm test`, `npm run lint`, `node --test`. They're read-only and safe to run without asking, and Claude runs them constantly while checking its work.
- **ask:** `git push`. Pushing sends work to a shared remote, so I want to approve each one.
- **deny:** reading `.env`, and `git push --force` / `git push -f`.

Without the `.env` deny rule, Claude could read real secrets into the conversation. From there they could end up in a log, a commit message or a pasted snippet. Without the force-push deny, one mistaken command could overwrite the remote branch and destroy commits that other people or the reviewer depend on. A deny rule wins over ask and allow, so these hold even if someone later broadens the allow list.

The `Read(./.env)` rule only blocks Claude's file-reading tool. A shell command such as `cat .env` isn't covered by it, so it lowers the risk without fully removing it.

While setting this up I also found that `.gitignore` had leading spaces on every line after `node_modules/`. Because of that, `.env` and `.claude/settings.local.json` weren't actually being ignored. I removed the spaces, and `git check-ignore` now confirms both are ignored.
