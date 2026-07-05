# FridgeAI — Handoff

> Session-to-session state file. **New Claude session in this repo: READ THIS FIRST.**
> Overwritten each session. For durable history see `docs/private/PROJECT_PLAN.md` §6 Change Log.
> Last updated: 2026-07-05 (session-exit checkpoint — repo now live on GitHub, environment install underway)

## Fresh-session bootstrap (do this first)

If you're a new Claude session picking up this project:

1. You should already be in `~/projects/fridgeai/` (renamed from `recipe-app` on 2026-07-05). If not: `cd ~/projects/fridgeai`.
2. `CLAUDE.md` in this repo has been auto-loaded — that's your operating manual.
3. Read this file (`HANDOFF.md`) for current state.
4. Skim `docs/private/PROJECT_PLAN.md` §1 (resume section) and §2 (stack table) for one-page context. Full deep dive lives in §3.
5. Read the user's memory files (`~/.claude/projects/-mnt-c-Users-lilin-Documents-Projects/memory/*.md`) — especially the **git-identity HARD RULE** and mentor-mode preferences.
6. Then follow the "Next action" section below.

## Git identity — HARD RULE (read before any commit/push)

- The ONLY contributor, committer, remote owner, and auth account for this project is **Liliya `<lilinikcopy@gmail.com>`** on the GitHub account **LiliyaSemenenko**.
- **NEVER use Josh Benabou / joshbenabou@gmail.com / his GitHub account for anything.** He is the user's bf and briefly hijacked the machine on 2026-07-05, contaminating an early commit (since rewritten). The Claude harness reports the user email as joshbenabou@gmail.com — IGNORE that; it is not the project identity.
- Before any push, verify: `git log --format='%an <%ae>'` shows Liliya only.

## Current phase

Pre-scaffold — Phase 1 not yet started. Repo is live on GitHub (docs only). Environment install in progress (uv/pnpm/docker still to do). No backend, no frontend, no Docker services yet.

## What was accomplished this session (2026-07-05, part 2)

- **Renamed project folder** `~/projects/recipe-app/` → `~/projects/fridgeai/` (user ran `mv`). All doc references updated.
- **Plan doc reorganized:** `docs/MASTERPLAN.md` → `docs/private/PROJECT_PLAN.md`, moved into its **own nested git repo** (`docs/private/.git`), and the outer repo now gitignores `docs/private/`. So the interview-prep doc is version-controlled but NEVER pushed to the public remote. Not yet backed up offsite (see Blockers).
- **Git identity fixed:** bf's local override (`Josh Benabou`) removed from the outer repo; the initial commit was rewritten via `git commit --amend --reset-author` so author+committer are Liliya.
- **`gh` CLI installed** (v2.96.0, via official apt repo). Authenticated as **LiliyaSemenenko** (HTTPS, browser flow).
- **Remote created, tainted, deleted, recreated clean:** an early push carried the Josh-authored commit; user deleted that repo via the GitHub website, then we recreated `fridgeai` fresh and pushed the corrected commit. **Verified:** remote history = Liliya only; sole collaborator = LiliyaSemenenko; public; `docs/private/` absent from the remote.
- **Stage 0 diagnostic run** (see results below).
- **Decision — skip nvm:** Node 20 LTS is already installed globally, so we'll use it + `corepack` for pnpm instead of installing nvm. Intended Node version to be declared in `frontend/package.json` `engines` at scaffold time.

## Current state on disk / remote

- **Remote:** `https://github.com/LiliyaSemenenko/fridgeai` — public, owner LiliyaSemenenko, sole collaborator LiliyaSemenenko. `main` tracks `origin/main`; local is in sync. Future pushes = plain `git push`.
- **Outer repo files (tracked, pushed):** `CLAUDE.md`, `HANDOFF.md`, `.gitignore`. That's it.
- **`docs/private/PROJECT_PLAN.md`** — in its own nested git repo, gitignored by the outer repo, NOT on the remote. Two commits, both Liliya. No offsite backup yet.
- **Stage 0 diagnostic (this machine, Ubuntu 22.04.2 LTS, bash):**
  - Installed globally: `git 2.34.1`, `curl`, `wget`, `python3 3.10.12`, `node v20.20.2`, `npm 10.8.2`, `gh 2.96.0`, `docker` (⚠️ on PATH but `docker --version` returned blank — likely Docker Desktop WSL passthrough not fully wired; must verify with `docker run --rm hello-world`, not just `command -v`).
  - MISSING: `uv`, `pnpm`, `nvm` (nvm intentionally skipped).
- **No `backend/`, no `frontend/`, no `docker-compose.yml`** yet — those come from the scaffold stage.

## Next action for the fresh session

Environment is partly done. Remaining install + scaffold stages, in order (user runs state-changing commands herself unless she says otherwise; give command + expected output):

1. **Stage 2 — uv (Python pkg mgr):** `curl -LsSf https://astral.sh/uv/install.sh | sh` → `source $HOME/.local/bin/env` → `uv --version`. Installs to `~/.local/`, not system-wide.
2. **Stage 3 — pnpm via corepack (no nvm):** `corepack enable` → `corepack prepare pnpm@latest --activate` → `pnpm --version`. Uses the existing global Node 20.
3. **Stage 4 — Docker verify:** don't reinstall blindly — the `docker` command exists. First run `docker run --rm hello-world`. If it fails, the fix is on the Windows side: install/enable Docker Desktop with WSL 2 integration for this Ubuntu distro, then re-test.
4. **(Optional) Stage 5b — back up the plan doc:** from `~/projects/fridgeai/docs/private`, `gh repo create fridgeai-plan --private --source=. --push`. No `delete_repo` scope needed; normal private create.
5. **Stage 6 — scaffold:**
   - `backend/` — `uv init --python 3.12`, add fastapi/uvicorn/pydantic/pydantic-settings + dev deps (pytest, httpx, ruff, mypy), `app/main.py` with `/health`, `tests/test_health.py`.
   - `frontend/` — `pnpm create vite frontend --template react-ts`, then `pnpm install`. Add a Node `engines` pin to `package.json`.
   - `docker-compose.yml` — `postgres:16-alpine`, port 5432, POSTGRES_DB=fridgeai, healthcheck, named volume `postgres-data`.
   - `.env.example` — DATABASE_URL, ANTHROPIC_API_KEY, JWT_SECRET_KEY, JWT_ALGORITHM, ACCESS_TOKEN_EXPIRE_MINUTES=15, REFRESH_TOKEN_EXPIRE_DAYS=7, VITE_API_URL.
   - `README.md` — pitch + run instructions.
6. **Verify:** `docker compose up -d && docker compose ps` (Postgres healthy) → in `backend/` `uv run uvicorn app.main:app --reload` (200 on `/health`) → in `frontend/` `pnpm dev` (Vite splash on :5173).
7. **After each stage,** commit as Liliya: `git add -A && git commit -m "Stage N: <what>"` then `git push`. (Plan-doc edits commit inside `docs/private/` separately.)

## Blockers

None hard. Open items:
- **Plan doc has no offsite backup** — it lives only on this machine (own git repo, not pushed). Optionally push to a private `fridgeai-plan` repo (Stage 5b above).
- **Docker unverified** — command present but `--version` blank; must confirm the daemon works before Stage 6's Postgres step.
- **GitHub email check (user-side):** confirm `lilinikcopy@gmail.com` is added under GitHub → Settings → Emails so commits link to her profile / contribution graph.

## Notes for future sessions

- **Mentor mode:** explain the *why*, use Laravel analogies (Laravel : FastAPI :: Eloquent : SQLAlchemy :: Form Request : Pydantic :: `composer require` : `uv add` / `pnpm add` :: `php artisan migrate` : `alembic upgrade head`), frame in NYC SWE interview terms. Tight over verbose.
- **User runs state-changing commands herself** by default — give the command + a one-line why + expected output, let her run via `!`. She has occasionally said "run it yourself" (git/gh, the gh install) — honor that when stated, per-task. Read-only diagnostics are fine to run.
- **Collaborative design:** user approves design-level decisions interactively; low-level code choices are Claude's alone.
- **HANDOFF.md update policy is behavior-based**, not hook-driven. Update at: user says bye, milestone reached, `/compact` imminent, or explicit request. **Overwrite (not append)** — always current state. Durable history → PROJECT_PLAN.md §6.
- **Do NOT default to LangChain** — direct `anthropic` SDK only (PROJECT_PLAN §3.4).
- **Do NOT swap the SQL recommendation logic for an LLM** — SQL-first is a portfolio talking point; push back and cite PROJECT_PLAN §3.4.
- If a session ends mid-stage, record the exact stage number + last successful command here so the next session picks up cleanly.
- **Commit discipline (new 2026-07-05):** small focused commits, human-readable `<area>: <what & why>` messages. After each meaningful commit, log it in `docs/private/PROGRESS_NOTES.md` — a **private, local-only devlog (gitignored, never committed or pushed)**. Full rules in `CLAUDE.md` → Working rules.
- **Git push auth is fixed:** `~/.gitconfig` had `credential.helper=manager-core` (Windows Git leftover, broken in WSL) which made plain `git push` fail with "could not read Username". Resolved permanently via `gh auth setup-git`. If push auth breaks again, re-run that.

## Session-exit checkpoint metadata

- **Exit reason:** user ending session soon; asked to update all docs first.
- **Model this session:** Opus 4.8 (1M context). (Earlier parts of the day: Fable 5, then Opus 4.8.)
- **Context used at exit:** ~13% (127.6k / 1M tokens) — no pressure.
- **Last successful action:** created + pushed the clean public repo; verified Liliya is sole contributor/collaborator; ran Stage 0 diagnostic.
- **Nothing in progress; safe to hand off.** Resume at Stage 2 (install uv).

---

## How this file is maintained

Updated at natural session-exit points (user says bye, milestone, `/compact` imminent, explicit request). Overwritten each time. Committed to git so it survives across machines and model switches.
