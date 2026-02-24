# Implementation Plan — Insights Log

> **Phase:** 3 — Solutioning
> **Agent:** The Architect
> **Parent Artifact:** [`specs/implementation-plan.md`](../implementation-plan.md)
> **Created:** 2026-02-24
> **Last Updated:** 2026-02-24

---

## About This Document

Living insights for the Developer agent capturing architectural concerns, watch items, and rationale for implementation ordering decisions.

---

## Insight Entries

---

### INS-001 · ⚠️ Risk · 2026-02-24T00:00:00Z

**M0-T03 is the highest-risk task in the entire plan**

Replacing ormar query patterns across 15+ router files simultaneously creates a wide regression surface. The recommended safe execution strategy:
1. Migrate ONE router file at a time (start with `classquiz/routers/quiz.py` — the most exercised)
2. Run `coverage run -m pytest classquiz/tests/test_server.py` after each file
3. Do not move to the next router file until all tests pass
4. Gate on test green — never batch multiple router migrations

The `classquiz/socket_server/__init__.py` file should be migrated LAST — it's the most complex and has the most indirect database access paths. Migrate it only after all router tests are stable.

---

### INS-002 · 💡 Decision · 2026-02-24T00:00:00Z

**M0 milestone added by Architecture — not in PRD**

The PRD task breakdown does not include the ormar → SQLModel migration because it was an Architect-phase decision. All M0 tasks are `[R]` refactoring tasks that enable the subsequent milestones. The Developer agent must not skip M0 or merge it into other milestones — all existing backend tests must pass before M1 begins.

---

### INS-003 · ⚠️ Risk · 2026-02-24T00:00:00Z

**M4-T04 (LLM circuit-breaker) is the top approver concern**

The project approver identified "LLM timeout / game loop resilience" as the top concern in the Round 2 architecture review. The `try/except LLMUnavailableError` pattern in `classquiz/socket_server/__init__.py` must be tested explicitly with a mocked `LLMUnavailableError` injection — not just with a slow LLM. Both integration tests (verdict path AND fallback path) are mandatory, not optional.

---

### INS-004 · 🔍 Observation · 2026-02-24T00:00:00Z

**Ollama warm-up latency may affect E8 first-round performance**

Ollama models loaded cold (first request after container start) can take 5–30 seconds depending on model size and host hardware. This exceeds the 5s p95 LLM verdict NFR for the first question of the first game after a restart. Two mitigations:
1. `LITELLM_API_BASE` supports a health check; the `/api/v1/llm/health` endpoint (M4-T05) should be called on game start to pre-warm the model
2. Document in the operator README: run `docker compose exec ollama ollama run llama3.1 "warmup"` after stack start
The circuit-breaker (M1-T04) handles the cold-start failure case gracefully — players get manual review rather than a hang.

---

### INS-005 · 💡 Decision · 2026-02-24T00:00:00Z

**M6 (animations) is Should Have — developer should not block M5 on M6 approval**

The implementation milestone ordering means M6 depends on M2 but not on M5. Dashboard (M5) can be delivered without animations (M6). This allows the approver to ship M0–M5 as a complete "High-Voltage Arcade" release and defer M6 animations to a subsequent sprint if time constraints arise. The design system (M1-T05) already handles `prefers-reduced-motion` globally — M6 tasks layer on top cleanly.

---

### INS-006 · 🔍 Observation · 2026-02-24T00:00:00Z

**SQLModel UUID primary key pattern difference from ormar**

ormar uses `uuid.uuid4` evaluated at class definition time (a known bug risk). SQLModel uses `default_factory=uuid.uuid4` (correct per-instance generation). When migrating `classquiz/db/models.py`, ensure ALL PKs use `Field(default_factory=uuid.uuid4, primary_key=True)` — not `Field(default=uuid.uuid4(), ...)`. The `alembic check` step in M0-T02 will catch schema drift but not the Python-level default evaluation bug.

---

### INS-007 · 💡 Decision · 2026-02-24T00:00:00Z

**`open_ended` type requires no Alembic schema migration (JSON column)**

After verifying the actual `classquiz/db/models.py` schema: `Quiz.questions` is a JSON column. The `QuizQuestion` Pydantic model adding `llm_rubric: str | None = None` and `"open_ended"` to the type enum requires **no schema change** — the JSON blob accepts new optional fields transparently. The Alembic migration file in M4-T01 is a no-op schema migration (empty `upgrade()`/`downgrade()` with a comment). This simplifies E8 backend work but the migration file should still exist as a record for auditing purposes.

---
