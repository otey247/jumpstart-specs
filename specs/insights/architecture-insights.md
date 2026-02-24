# Architecture — Insights Log

> **Phase:** 3 — Solutioning
> **Agent:** The Architect
> **Parent Artifact:** [`specs/architecture.md`](../architecture.md)
> **Created:** 2026-02-24
> **Last Updated:** 2026-02-24

---

## About This Document

Living insights log capturing architectural decisions, trade-offs, risk assessments, and close-call reasoning during Phase 3 solutioning for the PartyQuiz redesign + E8 LLM feature.

---

## Insight Entries

<!-- Entries appended chronologically. Format: ### INS-NNN · [type] · [ISO timestamp] -->

---

### INS-001 · 🔍 Observation · 2026-02-24T00:00:00Z

**Phase 3 activated with 5 open architectural decisions from upstream phases**

The following decisions were explicitly deferred to the Architect by the PM (per prd-insights.md and the PRD risk register):

1. **ormar ORM strategy** — deprecated library underpins all database access; no decision made yet
2. **LLM provider selection** — cloud (OpenAI/Anthropic + DPA) vs. self-hosted (Ollama — GDPR-safe)
3. **Design token delivery** — separate `$lib/tokens` module vs. plain CSS in `src/app.css`
4. **Font self-hosting** — `@fontsource/outfit` npm package (health verification required)
5. **Svelte migration sequencing** — interleave confirmed by PM but exact component ordering is an Architecture concern

All five require elicitation input before resolution. Tracking opened.

---

### INS-002 · ⚠️ Risk · 2026-02-24T00:00:00Z

**ormar deprecation is the single highest-stakes decision — affects 100% of data layer operations**

`ormar` is used in every model definition in `classquiz/db/models.py` (569 lines). Replacing it affects:
- All model classes (User, Quiz, FidoCredentials, ApiKey, UserSession, QuizTivity, etc.)
- All query patterns across 15+ router files
- Alembic migrations (though Alembic is ORM-independent — migrations survive a switch)
- Test fixtures that build ORM model instances

Three viable paths:
- **A: Keep ormar for now** — zero migration risk during this redesign phase; technical debt carried forward
- **B: Replace with SQLModel** — Pydantic v2 native, FastAPI-aligned, actively maintained; API surface is similar to ormar, migration is non-trivial but bounded
- **C: Replace with SQLAlchemy 2.x async directly** — maximum flexibility, most work, highest migration effort

Path B (SQLModel) is the preliminary recommendation: it maintains the Pydantic-first model definition style PartyQuiz already uses, aligns with FastAPI's creator (Sebastián Ramírez), and is actively maintained. Decision pending human input.

---

### INS-003 · ⚠️ Risk · 2026-02-24T00:00:00Z

**E8 LLM latency NFR (< 5s p95) puts the live game loop on the critical path of an external I/O call**

The socket server's `submit_answer` event handler for `open_ended` question types will need to call the LLM provider and return a verdict before the question timer expires (default: configurable, typically 15-30s). At p95 < 5s, this is achievable with a local Ollama model or fast cloud APIs, but:

1. A cold Ollama model load can take 5–15s on first request (warm-up required)
2. Cloud APIs have variable latency under load (OpenAI GPT-4o-mini typically 1–3s)
3. Circuit breaker / timeout logic is mandatory: if the LLM is unavailable, the game must not stall

The `classquiz/llm/` module must be designed with:
- Async client (LiteLLM is async-capable)
- Configurable timeout (env: `LLM_TIMEOUT_SECONDS`, default: 4s)
- Fallback verdict: if timeout exceeded, mark answer as "pending manual review" rather than blocking
- Health-check endpoint for operator monitoring

This follows the Library-First gate: `classquiz/llm/` is a standalone module with its own public API before being wired into socket_server.

---

### INS-004 · 🔍 Observation · 2026-02-24T00:00:00Z

**Socket server MAX_WORKERS=1 constraint is an architectural ceiling that must be explicitly documented**

The socket server uses an in-memory python-socketio adapter — rooms, SIDs, and connection state live in the server process RAM. Multiple workers would each maintain independent state, making cross-process socket events impossible without a Redis-based socket.io adapter.

The Architecture Document must explicitly acknowledge this constraint and note that E8 (LLM judging) does not worsen the situation — all LLM calls happen in the same single-worker process, in the same async event loop, without any new in-memory state that would block future multi-process migration.

If a future milestone targets horizontal scaling, the path would be: install `python-socketio[asyncio_client]` + configure `socketio.AsyncRedisManager` as the message_queue parameter. That is out of scope for this PRD and should be documented as a future ADR placeholder.

---

### INS-005 · 💡 Decision · 2026-02-24T00:00:00Z

**Design token delivery: TailwindCSS 4 `@theme` in `app.css` is the correct choice (Library-First satisfied)**

TailwindCSS 4 uses CSS custom properties in `@theme {}` blocks directly in CSS files — there is no JavaScript configuration file. A separate `$lib/tokens.ts` module would be a wrapper around CSS variables, which violates the Anti-Abstraction Gate (Article VII): no wrapper modules around framework primitives.

The correct architecture is:
- `frontend/src/app.css`: Single source of truth for all `@theme` tokens (colour, typography, spacing, radius, shadow)
- Token names follow a consistent pattern: `--color-[role]-[variant]` (e.g., `--color-accent-electric`, `--color-bg-void`)
- Svelte components reference tokens via Tailwind utility classes (`bg-accent-electric`) or CSS variables directly
- After E1-S1 WCAG audit locks hex values, the `@theme` block is written once and frozen for M1 gate

This approach has zero abstraction overhead, is fully aligned with TailwindCSS 4 conventions, and makes the WCAG audit gate clean (one file, one audit, one commit).

---

### INS-006 · 🔍 Observation · 2026-02-24T00:00:00Z

**Alembic migration strategy for E8: one new migration for `question_type` discriminator + `llm_rubric` field**

The existing `questions` JSON blob structure in `Quiz.questions` stores question data as a JSON array in a single `JSON` column. To support `open_ended` type questions with an LLM rubric, two approaches exist:

- **A: Add new fields to the existing JSON schema** — no migration needed; ormar/SQLModel just handles the new optional fields; the `type` discriminator already exists as a string field in `QuizQuestion`
- **B: Add a nullable `llm_rubric` TEXT column to the `quizzes` table (or a separate table)** — adds a migration but keeps rubric data queryable

Since `QuizQuestion` is serialized as JSON within the `Quiz.questions` JSON column, the rubric can be stored inline as a new optional field on the `QuizQuestion` Pydantic model — requiring **no Alembic migration**. The PRD states "1 new Alembic migration" but this may be avoidable if the JSON approach is confirmed.

This needs verification against the actual `classquiz/db/models.py` schema before committing to the migration count.

---

