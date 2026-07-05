# FridgeAI — Handoff

> Session-to-session state file. **New Claude session in this repo: READ THIS FIRST.**
> Overwritten each session. For durable history see `docs/private/PROJECT_PLAN.md` §6 Change Log.
> Last updated: 2026-07-05 (session-exit checkpoint, user restarting immediately with fresh context)

## Fresh-session bootstrap (do this first)

If you're a new Claude session picking up this project:

1. You should already be in `~/projects/fridgeai/` (renamed from `recipe-app` on 2026-07-05). If not: `cd ~/projects/fridgeai`.
2. `CLAUDE.md` in this repo has been auto-loaded — that's your operating manual.
3. Read this file (`HANDOFF.md`) for current state.
4. Skim `docs/private/PROJECT_PLAN.md` §1 (resume section) and §2 (stack table) for one-page context. Full deep dive lives in §3.
5. Read the user's memory files (`~/.claude/projects/-mnt-c-Users-lilin-Documents-Projects/memory/*.md`) for user profile and mentor-mode preferences.
6. Then follow the "Next action" section below.

## Current phase

Pre-scaffold — Phase 1 not yet started. Repo initialized with docs only. No backend, no frontend, no Docker services yet.

## What was accomplished last session (2026-07-05)

- Full stack locked end-to-end (backend, frontend, data, AI, ops). See `docs/private/PROJECT_PLAN.md` §2.
- `docs/private/PROJECT_PLAN.md` written: resume bullets, elevator pitch, per-choice deep dive with alternatives + interview Q&A, isolation-model section, Fly.io → AWS migration table, change log.
- Product scope confirmed: recipe catalog seeded from Spoonacular (~500–1,000 rows, no runtime dep), user accounts, virtual fridge (manual + receipt-photo entry), N-missing-ingredients recommendation via SQL, all four dietary features (tags + allergies + macros + cuisines; macros deferred to Phase 3).
- Core AI decisions locked: `anthropic` SDK direct (no LangChain), `tool_use` + Pydantic for structured output, Claude vision for receipts, Haiku for high-volume normalization, Sonnet default. **Recommendation is SQL, not LLM.**
- Auth decision locked: hand-rolled JWT (access + refresh, `bcrypt`) over Clerk/Auth0 — learning-first.
- Doc trio established: `CLAUDE.md` (per-session operating manual) + `HANDOFF.md` (this file) + the plan doc. The plan doc was briefly named `MASTERPLAN.md`; on 2026-07-05 the user renamed it back to `docs/private/PROJECT_PLAN.md` and made it **local-only** (gitignored, stripped from git history before first push).
- Memory files updated with user profile (PHP/Laravel dev, NYC SWE goal) and mentor-mode teaching style.
- **Repo initialized at `~/projects/fridgeai/`.** Docs committed. No remote yet.

## Current state on disk

- Working repo: `~/projects/fridgeai/` (git-initialized, one initial commit; folder renamed from `recipe-app` by the user on 2026-07-05).
- Files in repo:
  - `CLAUDE.md` (root)
  - `HANDOFF.md` (root — this file)
  - `docs/private/PROJECT_PLAN.md` (**private** — its own nested git repo; the outer repo gitignores `docs/private/`, so it is NOT in the public remote or its history)
  - `.gitignore` (Python + Node + env + Docker patterns + `docs/private/`)
- **No remote yet.** User will authenticate via `gh auth login` browser flow (no token) and create a **public** repo — visibility confirmed by user 2026-07-05.
- **Not yet installed on WSL:** `uv`, `nvm`, `node`, `pnpm`, `docker`, `gh`. Only `git 2.34.1` and `python3` exist.
- `~/projects/fridgeai/` has NO `backend/`, NO `frontend/`, NO `docker-compose.yml` — those come from the scaffold stage.

## Next action for the fresh session

Follow the install walkthrough in order. Full expected-output details are in the previous chat transcript, but the terse plan:

1. **Stage 0 — diagnostic:** `for t in git curl wget python3 node npm docker gh uv pnpm; do command -v "$t" >/dev/null && echo "OK  $t $($t --version 2>&1 | head -1)" || echo "MISS $t"; done` — establish what's present.
2. **Stage 1 — apt base:** `sudo apt update && sudo apt upgrade -y && sudo apt install -y build-essential ca-certificates curl wget`.
3. **Stage 2 — uv:** `curl -LsSf https://astral.sh/uv/install.sh | sh` → `source $HOME/.local/bin/env` → `uv --version`.
4. **Stage 3 — Node stack:** install nvm (v0.39.7) → `nvm install --lts` → `corepack enable` → `corepack prepare pnpm@latest --activate`.
5. **Stage 4 — Docker:** install Docker Desktop for Windows on host, enable WSL 2 integration for Ubuntu, verify `docker run --rm hello-world` from WSL.
6. **Stage 5 — gh CLI + push setup:** install `gh` (official apt repo), then `gh auth login` (browser flow). Then in the repo: `gh repo create fridgeai --public --source=. --remote=origin --push` to create the remote and push the docs. **Public** confirmed by user 2026-07-05.
7. **Stage 6 — scaffold:**
   - `backend/` — `uv init --python 3.12`, add fastapi/uvicorn/pydantic/pydantic-settings + dev deps (pytest, httpx, ruff, mypy), `app/main.py` with `/health`, `tests/test_health.py`.
   - `frontend/` — `pnpm create vite frontend --template react-ts`, then `pnpm install`.
   - `docker-compose.yml` — `postgres:16-alpine`, port 5432, POSTGRES_DB=fridgeai, healthcheck, named volume `postgres-data`.
   - `.env.example` — DATABASE_URL, ANTHROPIC_API_KEY, JWT_SECRET_KEY, JWT_ALGORITHM, ACCESS_TOKEN_EXPIRE_MINUTES=15, REFRESH_TOKEN_EXPIRE_DAYS=7, VITE_API_URL.
   - `README.md` — pitch + run instructions.
8. **Verify:** `docker compose up -d && docker compose ps` (Postgres healthy) → `uv run uvicorn app.main:app --reload` in `backend/` (200 on `/health`) → `pnpm dev` in `frontend/` (Vite splash on :5173).
9. **After each stage,** commit: `git add -A && git commit -m "Stage N: <what>"`. After scaffolding, push: `git push`.

## Blockers

None. Waiting on:
- User to run the `gh` install + auth + repo-create commands herself (she runs all state-changing commands — see Notes below). No token needed; browser login.
- (Resolved) Fable-5 model exists and is running these sessions.

## Notes for future sessions

- **Mentor mode:** explain the *why*, use Laravel analogies (Laravel : FastAPI :: Eloquent : SQLAlchemy :: Form Request : Pydantic :: `composer require` : `uv add` / `pnpm add` :: `php artisan migrate` : `alembic upgrade head`), frame in NYC SWE interview terms. Tight over verbose.
- User prefers to **run commands themselves** — give the command + expected output. Only execute for them on explicit request or for read-only diagnostics.
- **HANDOFF.md update policy is behavior-based**, not hook-driven. Update at: user says bye, milestone reached, `/compact` imminent, or explicit request. **Overwrite (not append)** — always current state, never a log. Durable history → PROJECT_PLAN.md §6.
- **Commit regularly** — user requested `git commit` after each meaningful step. Push once remote exists.
- **Do NOT default to LangChain** — see PROJECT_PLAN.md §3.4. Direct `anthropic` SDK only.
- **Do NOT swap the SQL recommendation logic for an LLM** — SQL-first is a portfolio talking point. Push back and cite PROJECT_PLAN.md §3.4 if the user is tempted.
- If a session ends mid-stage, record the exact stage number + last successful command in "Current state on disk" so the next session picks up cleanly.

## Session-exit checkpoint metadata

- **Exit reason:** user restarting immediately with fresh context, wants to try Fable-5 model.
- **Model this session:** Opus 4.7 (1M context).
- **Context used at exit:** ~15% (149.9k / 1M tokens) — well below any pressure.
- **Last successful action:** git repo init + initial commit of the three docs.
- **Nothing in progress; safe to hand off.**

---

## How this file is maintained

Updated at natural session-exit points (user says bye, milestone, `/compact` imminent, explicit request). Overwritten each time. Committed to git so it survives across machines and model switches.
