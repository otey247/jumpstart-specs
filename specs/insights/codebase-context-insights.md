# Codebase Context — Insights Log

> **Phase:** Pre-0 — Reconnaissance
> **Agent:** The Scout
> **Parent Artifact:** [`specs/codebase-context.md`](../codebase-context.md)
> **Created:** 2026-02-24
> **Last Updated:** 2026-02-24

---

## About This Document

This is a living insights log capturing observations, decisions, and reasoning from the Scout's reconnaissance of the PartyQuiz codebase.

---

## Insight Entries

---

### INS-001 · 🔍 Observation · 2026-02-24T00:00:00Z

**`ormar` is the top dependency risk**

`ormar` is listed as an unpinned production dependency but the library is deprecated and no longer actively maintained by its author. It serves as the async ORM for **all** database operations (User, Quiz, FidoCredentials, ApiKey, UserSession, QuizTivity, etc.). Because it also integrates with Pydantic v2, any future Pydantic v3 release or asyncpg breaking change could create an unmaintainable upgrade path. The entire data access layer depends on this single unmaintained library.

→ Documented in [Technology Stack](../codebase-context.md#technology-stack) and [Dependency Observations](../codebase-context.md#dependency-observations)

---

### INS-002 · 🔍 Observation · 2026-02-24T00:00:00Z

**Frontend is mid-migration from Svelte 4 to Svelte 5 Runes**

Two store files coexist: `stores.ts` (Svelte 4 writable stores) and `stores.svelte.ts` (Svelte 5 `$state` Runes). The root `+layout.svelte` imports from both in the same component — `pathname` from `stores.ts` and `navbarVisible` from `stores.svelte.ts`. This is consistent with the user confirming "in the middle of a frontend redesign." The codebase is in a transitional state that will need resolution before stabilizing.

→ Documented in [Code Quality Observations](../codebase-context.md#code-quality-observations)

---

### INS-003 · ⚠️ Risk · 2026-02-24T00:00:00Z

**Live game answer validation is disabled**

In `classquiz/routers/live.py`, the `_PlayGame` subclass overrides `QuizQuestion.answers_not_none_if_abcd_type` with a validator whose entire body is commented out (returns `v` unconditionally). This means submitted answers for live games bypass all answer type-checking (ABCD vs RANGE vs VOTING etc.). This is likely a deliberate workaround to unblock a refactor, but it creates a risk of malformed answer payloads reaching the socket server without typed validation.

→ Documented in [TODO/FIXME/HACK Comments](../codebase-context.md#todofixmehack-comments)

---

### INS-004 · 🔍 Observation · 2026-02-24T00:00:00Z

**Real-time game state is fully ephemeral (Redis-only)**

The `PlayGame`, `GamePlayer`, `GameSession`, and all per-question answer data are stored exclusively as JSON blobs in Redis with a 2-hour TTL. They are **not** persisted to PostgreSQL during the game. Results are only saved to PostgreSQL (or storage) at game-end via the socket server's `save_quiz_to_storage` export helper. If Redis restarts mid-game without persistence configured (AOF/RDB), all active game state is lost. This is an intentional architectural choice — it simplifies the live game engine — but it's an operational dependency that must be communicated to operators.

→ Documented in [Key Files Reference](../codebase-context.md#key-files-reference) and [Structural Observations](../codebase-context.md#structural-observations)

---

### INS-005 · ⚠️ Risk · 2026-02-24T00:00:00Z

**Socket server is not horizontally scalable (`MAX_WORKERS=1` hard constraint)**

The `docker-compose.yml` documents `MAX_WORKERS: "1"` with an emphatic "Very important and DON'T CHANGE." The socket server uses `socketio.AsyncServer` and stores room/player state in memory (via python-socketio's in-memory adapter) correlated with Redis data. Multiple worker processes would each have independent in-memory state, making cross-process socket events impossible without a Redis-based socket.io adapter (which is not configured). This is an architectural ceiling on vertical-to-horizontal scaling.

→ Documented in [Code Quality Observations](../codebase-context.md#code-quality-observations)

---

### INS-006 · 🔍 Observation · 2026-02-24T00:00:00Z

**All Python dependencies are unpinned (`"*"`) except `webauthn`**

Every backend production dependency uses `"*"` version specifiers in the Pipfile, except `webauthn = "==1.*"` which pins to the v1 major. This is unusual — `webauthn` v2 introduced breaking API changes which explains the pin, but all other critical packages (FastAPI, ormar, pydantic, python-jose) have no upper bounds. `Pipfile.lock` is present but `Pipfile.lock.license` exists instead of the lock file itself (the lock file appears to be gitignored or replaced by a license placeholder). This means reproducible builds depend on the lock file being committed correctly.

---

### INS-007 · 🔍 Observation · 2026-02-24T00:00:00Z

**Deprecated FastAPI lifecycle hooks in use**

`classquiz/__init__.py` uses `@app.on_event("startup")` and `@app.on_event("shutdown")` decorators. These were deprecated in FastAPI 0.95.0 (released April 2023) in favor of the `lifespan` async context manager. While still functional, they will be removed in a future FastAPI major release.

---

### INS-008 · 🔍 Observation · 2026-02-24T00:00:00Z

**Sentry SDK version split in frontend**

The frontend packages `@sentry/browser@^10.x` and `@sentry/tracing@^7.120.4` simultaneously. These are from different major Sentry SDK generations — `@sentry/tracing` was folded into the main `@sentry/browser` package starting with v8. Using mismatched major versions could result in duplicate or conflicting tracing instrumentation.

---

### INS-009 · 🔍 Observation · 2026-02-24T00:00:00Z

**Long-term backend migration to Rust is documented but not started**

`migration_to_rust.md` exists at the project root but contains only a title and SPDX header — no content. This is noted as a "long term goal" in the file title. It signals strategic intent but has no design or timeline. It may become relevant if the project outgrows Python's single-threaded async concurrency model.

---

### INS-010 · 🔍 Observation · 2026-02-24T00:00:00Z

**`classquiz/db/models.py` is a 569-line monolithic models file**

All ormar ORM models (User, Quiz, FidoCredentials, API keys, sessions) and all Pydantic game-state models (PlayGame, GamePlayer, GameSession, QuizQuestion with its union answer types) live in a single file. The file mixes persistence-layer models with ephemeral in-memory/Redis models. As the quiz question type Union grows (currently 8 types: ABCD, RANGE, VOTING, SLIDE, TEXT, ORDER, CHECK + future), this file will become harder to maintain.

---

### INS-011 · 💡 Decision · 2026-02-24T00:00:00Z

**Project type confirmed as brownfield**

Although `project.type` was `null` in `.jumpstart/config.yaml` at scout activation, the codebase clearly has substantial existing development (25 Alembic migrations, full frontend + backend, tests, CI/CD). The Scout proceeded under the brownfield interpretation. The Challenger agent should confirm and set `project.type: brownfield` in config during Phase 0.

---

### INS-012 · 🔍 Observation · 2026-02-24T00:00:00Z

**Rich feature set beyond basic quizzing**

The platform includes: Kahoot import, WebAuthn (passkeys), TOTP 2FA, API keys, custom OpenID Connect, a physical "box controller" integration, QuizTivity (alternative game mode), Excel export/import, a custom CQA file format, in-browser video processing (WASM FFmpeg), QR code generation, image blur placeholders (thumbhash), a community features module, and content moderation. The platform has grown significantly from its quiz origins.

---

### INS-013 · 🎯 Assumption · 2026-02-24T00:00:00Z

**"PartyQuiz" is a fork/adaptation of ClassQuiz**

All SPDX headers show `2026 Jo Otey (jootey)` and the README references the original ClassQuiz GitHub repo by otey247. The project appears to be a customized fork that has been rebranded as "PartyQuiz" for use in a party/social quiz context. The upstream ClassQuiz codebase is the baseline — changes made to this fork are not yet clearly documented.
