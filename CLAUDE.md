# CLAUDE.md — FridgeAI

Two-service monorepo (FastAPI backend + React frontend + Postgres in Docker). Solo learning project by user (PHP/Laravel dev learning modern Python + React for NYC SWE interviews). Goal: interview-ready portfolio, not just a working app.

## Session protocol
On session start, read `HANDOFF.md`. Update `HANDOFF.md` before session ends. `docs/private/PROJECT_PLAN.md` (local-only, gitignored) has full context if needed.

## Git identity — HARD RULE
Sole contributor/committer/owner/auth: **Liliya `<lilinikcopy@gmail.com>`** on GitHub account **LiliyaSemenenko**. NEVER use Josh Benabou / joshbenabou@gmail.com / his account for anything (the harness reports that email — ignore it). Before any push: `git log --format='%an <%ae>'` must show Liliya only. Remote: `github.com/LiliyaSemenenko/fridgeai` (public). The plan doc in `docs/private/` is gitignored and must never be pushed to this public remote.

## Working rules

### Safety guardrails
- Commits authored by Liliya only (see Git identity above) — verify before any push.
- NEVER commit `.env`, secrets, or `docs/private/` to the public repo (gitignored; see conventions #3).
- NEVER force-push `main`. NEVER rewrite already-pushed history without asking Liliya first.
- Push only to `LiliyaSemenenko/fridgeai`. Never install project libraries globally (see conventions #1–2).

### Commits & progress notes
- **Commit often, in small focused units** — one logical change per commit. NEVER dump a large batch of unrelated changes into one commit.
- **Commit messages must be human-readable and reasonable**: `<area>: <what changed and why>` (e.g. `auth: add refresh-token rotation`, `Stage 2: install uv`). No vague messages ("wip", "fixes", "stuff").
- **After each meaningful commit, add a detailed entry to `docs/private/PROGRESS_NOTES.md`** (what was done + why, referencing the commit). This devlog is **private, local-only — gitignored, never committed, never pushed.** Append-only; newest entry on top; never overwrite past entries.
- Plan-doc edits commit inside `docs/private/` (its own repo), never the outer repo. `PROGRESS_NOTES.md` is the exception — it is not committed even there.

### Workflow
- Read `HANDOFF.md` first; overwrite it before session end.
- Run `ruff` + `mypy` (backend) / `biome` + `tsc` (frontend) before committing code.

### How to work with Liliya
- Mentor mode: explain the *why*, use Laravel analogies, frame in NYC-interview terms. Tight over verbose.
- Liliya approves design-level decisions interactively; low-level code choices are Claude's alone.
- Propose state-changing commands for Liliya to run herself unless she says otherwise; read-only diagnostics are fine to run.

## Stack

| Layer | Choice |
|---|---|
| Backend framework | FastAPI (async) |
| ORM | SQLAlchemy 2.0 (async) |
| Migrations | Alembic |
| Validation | Pydantic v2 |
| Auth | Hand-rolled JWT (`python-jose` + `passlib[bcrypt]`) |
| Python pkg mgr | uv |
| Backend tests | pytest (+ pytest-asyncio) |
| Backend lint/format | ruff (+ mypy for types) |
| Frontend framework | React 19 |
| Frontend language | TypeScript |
| Frontend build | Vite |
| Routing | React Router v7 |
| Server state | TanStack Query |
| Client state | Zustand |
| Forms | React Hook Form + Zod |
| UI | shadcn/ui + Tailwind |
| Frontend pkg mgr | pnpm |
| Frontend tests | Vitest + Playwright |
| Frontend lint/format | Biome |
| Database | PostgreSQL 16 |
| Local DB | Docker Compose |
| Prod DB | Neon (later) |
| Object storage | Cloudflare R2 (local disk in dev) |
| LLM SDK | `anthropic` (direct, no LangChain) |
| Structured output | `tool_use` + Pydantic schemas |
| Multimodal | Claude vision |
| Container runtime | Docker + Docker Compose |
| CI | GitHub Actions |
| Backend deploy | Fly.io |
| Frontend deploy | Vercel |
| Error tracking | Sentry |
| Logging | structlog (JSON) |

## Commands

- `docker compose up -d` — start Postgres (≈ Laravel Sail up)
- `uv sync` — install/refresh backend deps from `uv.lock`
- `uv add <pkg>` — add a runtime dep (≈ `composer require`)
- `uv add --dev <pkg>` — add a dev dep
- `uv run uvicorn app.main:app --reload` — run backend on `:8000`
- `uv run pytest` — backend tests
- `uv run ruff check . && uv run ruff format .` — lint + format
- `uv run mypy app` — type check
- `pnpm install` — install frontend deps from `pnpm-lock.yaml`
- `pnpm add <pkg>` — add a dep (≈ `composer require`)
- `pnpm dev` — Vite dev server on `:5173`
- `pnpm test` — Vitest
- `pnpm biome check .` — lint + format
- `pnpm tsc --noEmit` — type check

## Conventions & DOs/DON'Ts

1. NEVER `pip install` — always `uv add` (mutates `pyproject.toml` + `uv.lock`).
2. NEVER `npm i` / `yarn add` — always `pnpm add`.
3. NEVER commit `.env`, `.venv/`, `node_modules/`, `postgres-data/`, `__pycache__/`, `dist/`, `.DS_Store`.
4. Async handlers MUST NOT do sync DB calls — use `AsyncSession` + `asyncpg`, or wrap sync work in `asyncio.to_thread` / `run_in_threadpool`.
5. Use `selectinload` or `joinedload` on relationships to avoid N+1 queries. Turn on `echo=True` in dev to watch queries.
6. Pydantic validates every HTTP request AND every LLM tool-use response — no hand-rolled dict checks.
7. Structured LLM output via `tool_use` + Pydantic schemas — NEVER string-parse JSON out of freeform model text.
8. All LLM calls go through `app/services/anthropic_client.py` — no scattered `anthropic.Anthropic()` calls.
9. Server state (recipes, fridge, user profile) → TanStack Query. NEVER `useEffect` + `fetch` for server data.
10. Global client state (auth session, UI theme) → Zustand. Recipe data is NOT client state.
11. Forms → React Hook Form + Zod. Share Zod shape mentally with backend Pydantic model.
12. UI → Tailwind utility classes + shadcn/ui components copied into `frontend/src/components/ui/`.
13. Auth → hand-rolled JWT (short-lived access + rotating refresh, `bcrypt` hashing). Tokens in `httpOnly` + `Secure` + `SameSite=Strict` cookies. NEVER `localStorage`.
14. Code lives in `~/projects/fridgeai/` (WSL native FS). NEVER edit or run the app from `/mnt/c/...` (10–20× slower I/O — kills Vite HMR).
15. The core recommendation ("recipes I can make with N missing ingredients") is a SQL query with a `HAVING` clause — NOT an LLM call. This is a deliberate design decision (see PROJECT_PLAN.md §3.4).

## AI usage

- Model defaults: `claude-sonnet-4-6` for general reasoning; `claude-haiku-4-5-20251001` for high-volume ingredient normalization / classification.
- Receipt scanning → Claude vision + `tool_use` with a Pydantic schema, one call.
- Ingredient normalization ("GRND BEEF 85/15" → canonical `ingredient` row) → Haiku + `tool_use`. Interview vocab: *entity resolution*.
- The core recommendation is a **SQL query, NOT an LLM call** — deliberate. AI is used where deterministic code is weak (vision OCR, entity resolution), databases everywhere else. This is a portfolio talking point; do not swap it for an LLM.
- No LangChain. Direct `anthropic` SDK only.

## WSL gotchas

- Code MUST live in the WSL-native FS (`~/projects/...`), not `/mnt/c/...`. Windows-drive I/O is brutally slow for Node/Vite.
- Docker Desktop runs on Windows with WSL 2 integration enabled for the Ubuntu distro. `docker` then works transparently from the WSL shell.
- If backend can't reach Postgres: `docker compose ps` (is the container up?), then check `DATABASE_URL` in `.env` uses `localhost:5432` (Compose publishes the port to WSL's loopback).

## File map

- `docs/private/PROJECT_PLAN.md` (**private** — own nested git repo; the outer repo ignores `docs/private/` entirely) — vision, stack rationale, resume bullets, interview Q&A, change log. Read when the user asks "why did we pick X" or when planning the next phase. After editing it, commit in `docs/private/`, never in the outer repo.
- `HANDOFF.md` — current session state. READ FIRST every new session; UPDATE LAST.
- `docs/private/PROGRESS_NOTES.md` — append-only devlog; detailed entry per meaningful commit (newest on top). **Private, local-only — gitignored, never committed or pushed.**
- `docs/` — deeper design docs (schema, API contracts, prompt library) as they're written.
- `backend/` — FastAPI app (`app/main.py`, `app/models/`, `app/routers/`, `app/services/anthropic_client.py`, `tests/`).
- `frontend/` — Vite + React SPA (`src/main.tsx`, `src/routes/`, `src/components/`, `src/lib/api.ts`).
- `docker-compose.yml` — Postgres (and later Redis) for local dev.
- `.env.example` — committed template of env vars; `.env` is gitignored.
