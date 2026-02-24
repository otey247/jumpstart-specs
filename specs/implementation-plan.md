---
title: "PartyQuiz Implementation Plan"
phase: 3
agent: Architect
status: Approved
version: 1.0.0
created: 2026-02-24
approved_by: Jo Otey
approval_date: 2026-02-24
upstream:
  - specs/architecture.md
  - specs/prd.md
---

# PartyQuiz — Implementation Plan

## Extended Table of Contents

| Section | Lines | Summary |
|---------|-------|---------|
| Plan Summary | ~30 | Goals, milestones overview, execution key |
| M0: Backend Preparation | ~110 | ORM migration, lifecycle modernisation, observability wiring |
| M1: Design System Foundation | ~120 | WCAG audit gate, @theme tokens, font, base components |
| M2: Core Game Loop Reskinned | ~160 | Full E2 player + E3 host reskin across 11 screens |
| M3: Shell & Homepage | ~50 | Navigation component + homepage route |
| M4: LLM Open-Ended Questions | ~130 | E8 end-to-end — backend, module, UI, fallback |
| M5: Creator Workflow & Dashboard | ~100 | E5 editor + E7 dashboard/settings reskin |
| M6: Gamification & Polish | ~80 | E6 animations + CI accessibility/Lighthouse gates |
| Traceability Matrix | ~30 | PRD story → implementation task mapping |

---

## Plan Summary

**Project:** PartyQuiz — High-Voltage Arcade redesign + LLM open-ended questions
**Architecture document:** [specs/architecture.md](architecture.md)
**PRD:** [specs/prd.md](prd.md)

### Milestone Map

| Milestone | Goal | Stories | Dep |
|-----------|------|---------|-----|
| M0 | Backend preparation (ORM + infra) | *Architectural prerequisite* | None |
| M1 | Design system live | E1 + E4-S1 + E6-S4 | M0 |
| M2 | Full game loop reskinned | E2 + E3 | M1 |
| M3 | Shell & homepage | E4-S2, E4-S3 | M1 |
| M4 | LLM open-ended questions | E8 | M2 |
| M5 | Creator workflow & dashboard | E5 + E7 | M1, M3 |
| M6 | Gamification & polish | E6-S1→S3 + CI gates | M2 |

### Execution Key

| Symbol | Meaning |
|--------|---------|
| `[S]` | Sequential — must complete before the next task |
| `[P]` | Parallelisable — can run concurrently with other `[P]` tasks in the same milestone |
| `[R]` | Refactoring — modifying existing code without changing external behaviour |
| `[M]` | Migration — data or schema migration |

---

## Milestone 0: Backend Preparation

**Goal:** Replace deprecated `ormar` ORM with `SQLModel`; modernise FastAPI lifecycle; add infrastructure services (Prometheus, Grafana, Ollama) to Docker Compose. All backend code in subsequent milestones uses `SQLModel` patterns.
**PRD Stories:** Architectural prerequisite (not in PRD — added by Architecture phase)
**Depends On:** None

---

### Task M0-T01: Snapshot current Alembic migration state `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | Database |
| **Story Reference** | Architectural prerequisite |
| **Files** | `migrations/versions/` |
| **Dependencies** | None |
| **Order** | `[S]` |

Run `alembic history` and confirm all migrations are at `head`. Create a git tag `pre-sqlmodel-migration`. This provides a rollback point. No code changes.

**Done When:** Git tag `pre-sqlmodel-migration` exists; `alembic current` shows `head` ✓

---

### Task M0-T02: Replace ormar models with SQLModel in models.py `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | API Layer — Database |
| **Story Reference** | Architectural prerequisite |
| **Files** | `classquiz/db/models.py` |
| **Dependencies** | M0-T01 |
| **Order** | `[S][R]` |

Rewrite `classquiz/db/models.py` using `SQLModel`. All `ormar.Model` subclasses become `SQLModel` with `table=True`. Pydantic-only models (game state: `PlayGame`, `GamePlayer`) remain pure `BaseModel` — they are not DB-mapped. Use `Optional[uuid.UUID] = Field(default_factory=uuid.uuid4, primary_key=True)` for UUID PKs. Add `sqlmodel` to `Pipfile`; remove `ormar`. Run `alembic check` — if schema drift detected, generate a corrective migration.

**Tests Required:**
- [ ] `test_models.py`: each SQLModel table class instantiates without error
- [ ] `alembic check` exits 0 — no unresolved schema drift

**Done When:** `classquiz/db/models.py` has no `ormar` imports; `alembic check` passes; all model tests pass ✓

---

### Task M0-T03: Update all router query patterns to SQLModel async sessions `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | API Layer |
| **Story Reference** | Architectural prerequisite |
| **Files** | `classquiz/routers/*.py`, `classquiz/routers/users/*.py`, `classquiz/socket_server/__init__.py`, `classquiz/worker/*.py` |
| **Dependencies** | M0-T02 |
| **Order** | `[S][R]` |

Replace all `ormar` query patterns with SQLModel `AsyncSession` + `select()`. Pattern: `async with get_session() as session: result = await session.exec(select(Model).where(...))`. Create `classquiz/db/session.py` with `create_async_engine` and `async_sessionmaker` using `DB_URL` from config. Inject sessions via FastAPI `Depends(get_session)`. Run existing test suite after each router file — fix any failures before moving to the next file.

**Tests Required:**
- [ ] All existing `classquiz/tests/test_auth.py` tests pass
- [ ] All existing `classquiz/tests/test_server.py` tests pass

**Done When:** No `ormar` imports remain in any file under `classquiz/`; full test suite passes ✓

---

### Task M0-T04: Migrate FastAPI lifecycle to lifespan context manager `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | API Layer |
| **Story Reference** | Architectural prerequisite |
| **Files** | `classquiz/__init__.py` |
| **Dependencies** | M0-T03 |
| **Order** | `[S][R]` |

Replace `@app.on_event("startup")` and `@app.on_event("shutdown")` with the `@asynccontextmanager` lifespan pattern introduced in FastAPI 0.95. Move all startup logic (DB table creation, Redis init, MeiliSearch init) into the lifespan context. This removes deprecation warnings and prepares for future FastAPI major releases.

**Done When:** `classquiz/__init__.py` uses `lifespan=` parameter on `FastAPI()`; no `@app.on_event` decorators remain; app starts and passes smoke test ✓

---

### Task M0-T05: Restore disabled live game answer validator `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | API Layer |
| **Story Reference** | Architectural prerequisite (bug fix) |
| **Files** | `classquiz/routers/live.py` |
| **Dependencies** | M0-T02 |
| **Order** | `[S][R]` |

In `classquiz/routers/live.py`, the `answers_not_none_if_abcd_type` validator body is commented out (returns `v` unconditionally). Restore the validator logic using SQLModel/Pydantic v2 `@field_validator` syntax. Test that ABCD questions reject None answers and that RANGE questions accept numeric values.

**Tests Required:**
- [ ] Test: ABCD question with None answers raises `ValidationError`
- [ ] Test: `open_ended` question with None answers is valid (no answer options required)

**Done When:** Validator is active; tests pass; no regression on existing game loop tests ✓

---

### Task M0-T06: Add Prometheus, Grafana, and Ollama to docker-compose.yml `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | Infrastructure |
| **Story Reference** | ADR-006, ADR-002 |
| **Files** | `docker-compose.yml`, `docker-compose.dev.yml`, `prometheus/prometheus.yml` (new), `grafana/dashboards/partyquiz.json` (new) |
| **Dependencies** | None |
| **Order** | `[P]` |

Add three new services to `docker-compose.yml`:
- `ollama`: image `ollama/ollama:latest`; port 11434; volume `ollama_data`; default env: `LITELLM_MODEL=ollama/llama3.1`, `LITELLM_API_BASE=http://ollama:11434`
- `prometheus`: image `prom/prometheus:latest`; port 9090; mount `./prometheus/prometheus.yml`; scrapes `api:8000/metrics` every 15s
- `grafana`: image `grafana/grafana:latest`; port 3000; provisioned dashboard JSON at `./grafana/dashboards/partyquiz.json`

Document `ollama pull llama3.1` one-time setup step in `README.md`.

**Done When:** `docker compose up` starts all new services without errors; Grafana dashboard accessible at `:3000`; Prometheus targets show `api` as UP ✓

---

### Task M0-T07: Add prometheus-fastapi-instrumentator + LLM metrics `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | API Layer — Observability |
| **Story Reference** | ADR-006 |
| **Files** | `classquiz/__init__.py`, `classquiz/llm/__init__.py`, `Pipfile` |
| **Dependencies** | M0-T04 |
| **Order** | `[S]` |

Add `prometheus-fastapi-instrumentator` to `Pipfile`. Mount `/metrics` endpoint on the FastAPI app. In `classquiz/llm/__init__.py`, register three metrics: `llm_eval_duration_seconds` (histogram), `llm_eval_calls_total` (counter), `llm_eval_errors_total` (counter by `error_type`). Increment counters and observe duration in `evaluate_answer()`.

**Done When:** `GET /metrics` returns valid Prometheus text format; LLM histogram bucket appears after one `evaluate_answer()` call ✓

---

### Milestone 0 Verification

- [ ] No `ormar` imports remain anywhere in `classquiz/`
- [ ] Full Python test suite passes (`coverage run -m pytest`)
- [ ] `docker compose up` starts all services including prometheus, grafana, ollama
- [ ] `/metrics` endpoint accessible and scraping correctly

**Milestone completed on:** PENDING

---

## Milestone 1: Design System Foundation

**Goal:** The High-Voltage Arcade design token layer, Outfit font, WCAG-verified palette, and base components (Button, Card) exist and are usable by all epics.
**PRD Stories:** E1-S1, E1-S2, E1-S3, E1-S4, E1-S5, E4-S1, E6-S4
**Depends On:** M0

---

### Task M1-T01: WCAG AA contrast audit (blocking gate) `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Design System |
| **Story Reference** | E1-S1 |
| **Files** | `specs/decisions/wcag-audit.md` (new) |
| **Dependencies** | None |
| **Order** | `[S]` |

Run axe-core CLI or Colour Contrast Analyser against all five accent colours (#FFD02F, #00E055, #2D8CFF, #FF4545, and white #FFFFFF) on the dark background #0f1012. For each colour, record the contrast ratio and WCAG AA pass/fail (minimum 4.5:1 for normal text, 3:1 for large text). If any colour fails, find the nearest passing hex value and document the adjustment. All subsequent token tasks use the verified hex values from this document.

**Done When:** `specs/decisions/wcag-audit.md` exists with ratio + pass/fail for all 5 colours; final hex values locked ✓

---

### Task M1-T02: Import Outfit font + write @theme tokens `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Design System |
| **Story Reference** | E1-S2, E1-S3 |
| **Files** | `frontend/src/app.css`, `frontend/package.json` |
| **Dependencies** | M1-T01 |
| **Order** | `[S]` |

Add `@fontsource/outfit` to `frontend/package.json`. In `frontend/src/app.css`, import Outfit weights 400, 700, 900 from `@fontsource/outfit`. Define the `@theme {}` block with all tokens: `--color-bg-void`, `--color-accent-*` (5 colours from M1-T01 audit), `--font-display`, `--shadow-hard`, `--shadow-hard-sm`, `--radius-base: 0px`. Apply `--color-bg-void` to `html, body` and `--font-display` as the default font stack. Verify in DevTools: no request to `fonts.googleapis.com`.

**Tests Required:**
- [ ] Vitest: `app.css` contains `@theme` block with all required token names
- [ ] Manual: DevTools network tab shows no external font request

**Done When:** Outfit renders in browser; all token names defined; no Google Fonts request ✓

---

### Task M1-T03: Create Button.svelte base component `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Design System |
| **Story Reference** | E1-S4 |
| **Files** | `frontend/src/lib/components/ui/Button.svelte`, `frontend/src/lib/components/ui/Button.test.ts` |
| **Dependencies** | M1-T02 |
| **Order** | `[P]` |

Create `Button.svelte` using Svelte 5 Runes (`$props()`). Props: `variant: "primary" | "secondary" | "ghost" | "danger"`, `disabled: boolean`. Uses token classes: `bg-accent-electric text-bg-void shadow-hard` (primary), `border border-accent-electric text-accent-electric` (secondary), etc. SPDX MPL-2.0 header required.

**Tests Required:**
- [ ] Vitest: all 4 variants render without error
- [ ] Vitest: `disabled` prop adds `aria-disabled` attribute

**Done When:** Component renders all variants; tests pass; no hardcoded hex values ✓

---

### Task M1-T04: Create Card.svelte base component `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Design System |
| **Story Reference** | E1-S5 |
| **Files** | `frontend/src/lib/components/ui/Card.svelte` |
| **Dependencies** | M1-T02 |
| **Order** | `[P]` |

Create `Card.svelte` using Svelte 5 Runes. Dark surface (`bg-bg-void`), hard border (`border-2 border-white`), hard shadow (`shadow-hard`), zero border-radius. Accepts `class` prop for composition. SPDX header.

**Done When:** Card renders with expected visual properties; Vitest render test passes ✓

---

### Task M1-T05: Add prefers-reduced-motion utility `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Design System |
| **Story Reference** | E6-S4 |
| **Files** | `frontend/src/app.css` |
| **Dependencies** | M1-T02 |
| **Order** | `[P]` |

Append `@media (prefers-reduced-motion: reduce) { *, *::before, *::after { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; } }` to `frontend/src/app.css`. This global override ensures all animations and transitions are suppressed for users with motion sensitivity. All animated components added in M6 automatically respect this — no per-component opt-in required.

**Done When:** CSS block is present; Playwright test with `prefers-reduced-motion: reduce` confirms no transitions fire ✓

---

### Milestone 1 Verification

- [ ] All M1 tasks marked COMPLETE
- [ ] Vitest: Button + Card component tests pass
- [ ] No `fonts.googleapis.com` request in network tab
- [ ] WCAG audit document locked with verified hex values
- [ ] `@theme` block in `app.css` has all required token names

**Milestone completed on:** PENDING

---

## Milestone 2: Core Game Loop Reskinned

**Goal:** A complete live game session — PIN entry through final results — works end-to-end with the new design on both mobile (375px) and desktop/TV (1280px).
**PRD Stories:** E2-S1 through E2-S6, E3-S1 through E3-S5
**Depends On:** M1

> **Regression guard:** Socket game loop + auth + quiz CRUD must pass throughout this milestone (NFR-BC01–BC08). Run `coverage run -m pytest classquiz/tests/test_server.py` between tasks.

---

### Task M2-T01: PIN entry screen `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Player UI |
| **Story Reference** | E2-S1, E2-S6 (error state) |
| **Files** | `frontend/src/routes/play/+page.svelte` (and related `+page.ts`) |
| **Dependencies** | M1 complete |
| **Order** | `[S]` |

Locate existing PIN entry route. Migrate to Svelte 5 Runes. Apply token-driven styles: `--color-bg-void` background, Outfit font, `Button` component for "Join" CTA. Verify tap targets ≥ 44px. Add inline error state for invalid PIN (no redirect; input reset). No game flow logic changes — styling only.

**Tests Required:**
- [ ] Playwright: valid game PIN → lobby screen loaded
- [ ] Playwright: invalid PIN → inline error message shown, no redirect

**Done When:** Renders correctly at 375px; error state works; both tests pass ✓

---

### Task M2-T02: Lobby wait screen + full player flow `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Player UI |
| **Story Reference** | E2-S2, E2-S3, E2-S4, E2-S5 |
| **Files** | `frontend/src/routes/play/game/` (lobby, question, leaderboard, results screens) |
| **Dependencies** | M2-T01 |
| **Order** | `[S]` |

Redesign all four player screens (lobby → question → leaderboard → results) in a single pass per screen, each with Runes migration. Key requirements: answer buttons ≥ 44px height, selected-state highlight using token colours, timer countdown display, rank improvement indicator on leaderboard, rank + score + correct-count above fold at 375px on results screen. CSS "waiting" animation on lobby (respects prefers-reduced-motion).

**Tests Required:**
- [ ] Playwright: complete game loop as player at 375px — all 4 transitions succeed
- [ ] Playwright: answer button tap locks further input, shows waiting state

**Done When:** All 4 player screens render at 375px; game loop test passes ✓

---

### Task M2-T03: Disconnect/reconnect handler `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Player UI |
| **Story Reference** | E2-S6 |
| **Files** | `frontend/src/lib/socket.ts`, active game route components |
| **Dependencies** | M2-T02 |
| **Order** | `[S]` |

Add reconnection indicator overlay, 5-attempt retry logic with exponential backoff, and error state with "Reconnecting…" CTA. On reconnect success, restore current game state from socket event. On exhausted retries, show "Connection lost" with "Try again" button.

**Tests Required:**
- [ ] Playwright: mock socket disconnect → indicator shown → reconnect success → state restored
- [ ] Playwright: 5 failed reconnects → final error state shown

**Done When:** All reconnect scenarios covered; tests pass ✓

---

### Task M2-T04: Host lobby screen `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Host UI |
| **Story Reference** | E3-S1 |
| **Files** | `frontend/src/routes/host/` (lobby screen) |
| **Dependencies** | M1 complete |
| **Order** | `[P]` |

Redesign host lobby: game PIN displayed at ≥ 96px font size, real-time player join list (socket event driven). Migrate to Runes. Apply tokens.

**Tests Required:**
- [ ] Playwright: player join socket event → name appears in list within 1s

**Done When:** Renders at 1280px; PIN prominent; player join test passes ✓

---

### Task M2-T05: Host question display + answer tally `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Host UI |
| **Story Reference** | E3-S2, E3-S3 |
| **Files** | `frontend/src/routes/host/` (question + tally screens) |
| **Dependencies** | M2-T04 |
| **Order** | `[S]` |

Question display: question text + answer options + animated countdown timer. Answer tally + reveal: live response counter, correct answer highlight, per-option stats bars. Both screens migrate to Runes + token styles.

**Done When:** Renders at 1280px; timer animates; tally updates live; regression tests pass ✓

---

### Task M2-T06: Host leaderboard + winner screens `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Host UI |
| **Story Reference** | E3-S4, E3-S5 |
| **Files** | `frontend/src/routes/host/` (leaderboard + winner screens) |
| **Dependencies** | M2-T05 |
| **Order** | `[S]` |

Leaderboard: top-N ranking, rank movement indicators, ≥ 32px font. Winner/final: winner prominence, co-winner support, visually distinct from intermediate leaderboard. Runes + tokens.

**Tests Required:**
- [ ] Playwright: leaderboard renders top-5; rank movement indicator present

**Done When:** Both screens render at 1280px; leaderboard test passes ✓

---

### Milestone 2 Verification

- [ ] All M2 tasks marked COMPLETE
- [ ] Playwright game loop test passes end-to-end (player 375px + host 1280px)
- [ ] Disconnect/reconnect test passes
- [ ] Backend regression tests pass (`test_server.py`, `test_auth.py`)

**Milestone completed on:** PENDING

---

## Milestone 3: Shell & Homepage

**Goal:** Public-facing homepage and global navigation carry the new brand identity.
**PRD Stories:** E4-S2, E4-S3
**Depends On:** M1

---

### Task M3-T01: Nav.svelte component `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Layout |
| **Story Reference** | E4-S2 |
| **Files** | `frontend/src/lib/components/layout/Nav.svelte` |
| **Dependencies** | M1 complete |
| **Order** | `[S]` |

Create `Nav.svelte` (Runes). Logo, navigation links, auth CTA states (signed in / signed out). Mobile collapse at 375px. Uses `Button` component for auth CTA. SPDX header.

**Done When:** Renders at both 375px and 1280px; auth state switches correctly ✓

---

### Task M3-T02: Homepage route `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Routes |
| **Story Reference** | E4-S3, E4-S1 |
| **Files** | `frontend/src/routes/+page.svelte`, `frontend/src/routes/+layout.svelte` |
| **Dependencies** | M3-T01 |
| **Order** | `[S]` |

Redesign `/` homepage: hero section with headline ("Play smarter. Party harder."), sub-headline, primary "Host a Quiz" CTA button, bento grid layout showcasing feature cards. Apply `--color-bg-void` to `+layout.svelte` root. Authenticated-user variant: redirect to dashboard or show "Your recent quizzes" grid.

**Tests Required:**
- [ ] Playwright: unauthenticated → hero + CTA visible
- [ ] Playwright: authenticated → dashboard redirect or recent quizzes

**Done When:** Homepage passes visual verification at 375px + 1280px; both tests pass ✓

---

### Milestone 3 Verification

- [ ] Nav component renders at both breakpoints
- [ ] Homepage passes both aunauthenticated + authenticated tests

**Milestone completed on:** PENDING

---

## Milestone 4: LLM Open-Ended Questions

**Goal:** Open-ended questions work end-to-end — editor through live game judgment — with a fallback if LLM is unavailable.
**PRD Stories:** E8-S1 through E8-S5
**Depends On:** M2 (game loop must be stable before adding new question type)

---

### Task M4-T01: Add open_ended question type to models + Alembic migration `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | API Layer — Database |
| **Story Reference** | E8-S1 |
| **Files** | `classquiz/db/models.py`, `migrations/versions/NNN_add_open_ended_type.py` |
| **Dependencies** | M0 complete |
| **Order** | `[S][M]` |

In `classquiz/db/models.py`, update `QuizQuestion` Pydantic model: add `"open_ended"` to the `type` field enum; add `llm_rubric: str | None = None` optional field. Because `questions` is stored as a JSON column, adding an optional field to the Pydantic model requires no schema change — the migration file validates this and only contains a comment record. Generate migration with `alembic revision -m "add_open_ended_question_type"`.

**Tests Required:**
- [ ] pytest: `Quiz` with `open_ended` question type saves and retrieves correctly
- [ ] pytest: `open_ended` question with `llm_rubric` round-trips through JSON serialisation

**Done When:** `open_ended` type saves/retrieves correctly; migration applies cleanly ✓

---

### Task M4-T02: Socket server E8 event handling `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | Game Engine |
| **Story Reference** | E8-S1 |
| **Files** | `classquiz/socket_server/__init__.py`, `classquiz/socket_server/models.py` |
| **Dependencies** | M4-T01 |
| **Order** | `[S]` |

In the socket server's `submit_answer` handler: detect `question.type == "open_ended"`. For open_ended answers: store answer text in Redis game state (not scored immediately). Queue for LLM evaluation (async dispatch to M4-T03 `evaluate_answer()`). Do not emit score until verdict arrives. Broadcast `open_ended` question events to players with `type: "open_ended"` and no `answers` array.

**Done When:** `open_ended` question events broadcast correctly; answers queued without immediate scoring ✓

---

### Task M4-T03: Create classquiz/llm/ evaluation module `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | LLM Evaluation Module |
| **Story Reference** | E8-S2 |
| **Files** | `classquiz/llm/__init__.py`, `classquiz/llm/models.py`, `classquiz/llm/health.py` |
| **Dependencies** | M0-T07 |
| **Order** | `[S]` |

Create standalone `classquiz/llm/` module. Public API: `async def evaluate_answer(question: str, answer: str, rubric: str | None = None) -> Verdict`. Implementation uses `litellm.acompletion()` with a structured prompt asking the model to respond with JSON `{"verdict": "correct"|"incorrect"|"partial"}`. `Verdict = Literal["correct", "incorrect", "partial"]`. Config from env: `LITELLM_MODEL`, `LITELLM_API_BASE`, `LITELLM_API_KEY`, `LLM_TIMEOUT_SECONDS` (default 4). Timeout: `asyncio.wait_for(..., timeout=LLM_TIMEOUT_SECONDS)`. On `asyncio.TimeoutError` or any exception: raise `LLMUnavailableError`. SPDX header. Add `litellm` to `Pipfile`.

**Tests Required:**
- [ ] pytest: `evaluate_answer()` with mocked `acompletion` returns correct `Verdict`
- [ ] pytest: `asyncio.TimeoutError` → `LLMUnavailableError` is raised
- [ ] pytest: malformed LLM response → `LLMUnavailableError` is raised

**Done When:** All 3 tests pass; `LLMUnavailableError` raises on all failure modes ✓

---

### Task M4-T04: Wire LLM module into socket server answer handler `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | Game Engine — LLM Module |
| **Story Reference** | E8-S2, E8-S5 |
| **Files** | `classquiz/socket_server/__init__.py` |
| **Dependencies** | M4-T02, M4-T03 |
| **Order** | `[S]` |

In the answer handler for `open_ended` type: call `await evaluate_answer(question_text, answer_text, rubric)` wrapped in `try/except LLMUnavailableError`. On success: emit `llm_verdict` event to player SID + update host aggregate count. On `LLMUnavailableError`: emit `manual_review_required` event to host SID with answer text list. Never block the event loop — the `evaluate_answer` call is `await`ed inside the async handler.

**Tests Required:**
- [ ] Integration test: mock LLM success → `llm_verdict` event emitted to correct player SID
- [ ] Integration test: mock `LLMUnavailableError` → `manual_review_required` emitted to host only

**Done When:** Both verdict paths tested; game loop does not hang on LLM failure ✓

---

### Task M4-T05: LLM health check endpoint `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | API Layer — LLM Module |
| **Story Reference** | E8-S2 |
| **Files** | `classquiz/routers/utils.py` |
| **Dependencies** | M4-T03 |
| **Order** | `[P]` |

Add `GET /api/v1/llm/health` route (admin auth required). Calls `classquiz/llm/health.health_check()` which does a minimal `evaluate_answer("test", "test")` call with 2s timeout. Returns `{"status": "healthy", "provider": LITELLM_MODEL, "latency_ms": N}` or `{"status": "unhealthy", "error": "..."}`.

**Done When:** Endpoint returns 200 when LLM reachable, 503 when not; admin-only guard tested ✓

---

### Task M4-T06: OpenEndedPlayer.svelte component (player text input) `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — E8 UI |
| **Story Reference** | E8-S3 |
| **Files** | `frontend/src/lib/components/game/OpenEndedPlayer.svelte` |
| **Dependencies** | M1 complete, M2-T02 |
| **Order** | `[S]` |

Create `OpenEndedPlayer.svelte` (Runes). Text input with soft keyboard support (mobile). "Submit" button using `Button` component. On submission: lock input, show "Waiting for result…" with CSS spinner (respects prefers-reduced-motion). On `llm_verdict` socket event: transition to correct/incorrect feedback state (token-coloured flash). SPDX header.

**Tests Required:**
- [ ] Playwright: text input visible above fold at 375px
- [ ] Playwright: submit → input locked → waiting state shown → verdict → feedback state

**Done When:** Component renders at 375px; all state transitions tested ✓

---

### Task M4-T07: ManualReviewPanel.svelte + host verdict display `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — E8 UI |
| **Story Reference** | E8-S4, E8-S5 |
| **Files** | `frontend/src/lib/components/game/ManualReviewPanel.svelte`, host question screen |
| **Dependencies** | M4-T04, M4-T06 |
| **Order** | `[S]` |

`ManualReviewPanel.svelte`: list of player answers with Correct/Incorrect/Partial buttons. Emits `manual_verdict` socket event on host action. Host question screen update (E3-S3): add "Evaluated: N / M" counter that increments on `llm_verdict` events. Activates ManualReviewPanel when `manual_review_required` event arrives.

**Tests Required:**
- [ ] Playwright: mock `manual_review_required` → panel appears with player answers
- [ ] Playwright: host clicks Correct → `manual_verdict` socket event emitted → player score updates

**Done When:** Manual review panel functional; score update on host verdict tested ✓

---

### Milestone 4 Verification

- [ ] All M4 tasks marked COMPLETE
- [ ] Full open-ended round tested end-to-end (editor → game → LLM verdict)
- [ ] LLM failure fallback verified (mock LLM down → manual review activates)
- [ ] `/api/v1/llm/health` accessible to admin users

**Milestone completed on:** PENDING

---

## Milestone 5: Creator Workflow & Dashboard

**Goal:** Quiz editor fully reskinned; dashboard and supporting screens complete full-platform design consistency.
**PRD Stories:** E5-S1 through E5-S4, E7-S1, E7-S2, E7-S3
**Depends On:** M1, M3

---

### Task M5-T01: Quiz editor layout `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Editor |
| **Story Reference** | E5-S1 |
| **Files** | `frontend/src/routes/edit/` |
| **Dependencies** | M1 complete |
| **Order** | `[S]` |

Redesign editor route: left panel (question list), canvas area, top toolbar (Add Question, Save, Preview, Settings). Migrate to Runes. Apply tokens. Quiz CRUD regression test must pass after this task.

**Done When:** Editor layout renders; `test_server.py` quiz CRUD tests pass ✓

---

### Task M5-T02: Question cards + settings form + type menu `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Editor |
| **Story Reference** | E5-S2, E5-S3, E5-S4 |
| **Files** | `frontend/src/routes/edit/` (question cards, settings panel) |
| **Dependencies** | M5-T01 |
| **Order** | `[S]` |

MC question card: question text, 4 option inputs, correct-answer indicator. Settings form: title (required), description, visibility toggle, inline validation. Add Question menu: MC + open text + true/false options; true/false and additional types show "Coming soon" badges (E8 open-ended is functional). Runes throughout.

**Done When:** Editor functional; open-ended question type selectable; quiz saves + retrieves ✓

---

### Task M5-T03: Dashboard route `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Dashboard |
| **Story Reference** | E7-S1 |
| **Files** | `frontend/src/routes/dashboard/+page.svelte` |
| **Dependencies** | M1, M3 complete |
| **Order** | `[S]` |

Redesign `/dashboard`: bento grid of quiz cards (title, question count, last modified, Play/Edit buttons). Empty state CTA when no quizzes. Runes + tokens.

**Done When:** Dashboard renders quiz grid; empty state CTA shown for new users ✓

---

### Task M5-T04: Results summary + profile/settings pages `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Supporting Screens |
| **Story Reference** | E7-S2, E7-S3 |
| **Files** | `frontend/src/routes/results/+page.svelte`, `frontend/src/routes/settings/+page.svelte` |
| **Dependencies** | M5-T03 |
| **Order** | `[S]` |

Results summary: ranked player list bento grid (1280px) / stacked list (375px). Profile/settings: display name update → nav `Nav.svelte` reflects change on save. Runes + tokens.

**Tests Required:**
- [ ] Playwright: update display name → Nav shows updated name without page reload

**Done When:** Results and settings pages match design system; display-name update test passes ✓

---

### Milestone 5 Verification

- [ ] All M5 tasks marked COMPLETE
- [ ] Editor functional with new design; quiz CRUD tests pass
- [ ] Dashboard, results, and settings match design system

**Milestone completed on:** PENDING

---

## Milestone 6: Gamification & Polish

**Goal:** Answer feedback animations, score counters, and leaderboard movement animations layered on the core screens; CI accessibility + performance gates added.
**PRD Stories:** E6-S1, E6-S2, E6-S3 (Should Have)
**Depends On:** M2

---

### Task M6-T01: Answer feedback flash animation `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Animations |
| **Story Reference** | E6-S1 |
| **Files** | `frontend/src/lib/components/game/OpenEndedPlayer.svelte`, MC answer screen |
| **Dependencies** | M2-T02, M4-T06 |
| **Order** | `[S]` |

Add CSS `@keyframes` correct/incorrect flash (green/red background pulse ≤ 300ms) to player answer feedback components. `prefers-reduced-motion` global override in `app.css` (M1-T05) suppresses automatically.

**Done When:** Flash animation plays at ≥ 60fps; suppressed with `prefers-reduced-motion: reduce` ✓

---

### Task M6-T02: Score counter increment animation `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Animations |
| **Story Reference** | E6-S2 |
| **Files** | Player leaderboard + results screens |
| **Dependencies** | M2-T02 |
| **Order** | `[P]` |

CSS counter or JS number tween animating score from previous to new value over ≤ 500ms. Applied to player leaderboard rank column and results screen final score.

**Done When:** Score increments animate; motion preference test passes ✓

---

### Task M6-T03: Leaderboard row reorder animation `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | SvelteKit Frontend — Animations |
| **Story Reference** | E6-S3 |
| **Files** | Host leaderboard screen, player leaderboard screen |
| **Dependencies** | M2-T06 |
| **Order** | `[P]` |

CSS cross-fade/slide transition when leaderboard rows reorder between rounds using Svelte's `animate:flip` directive (or manual FLIP). ≤ 500ms total.

**Done When:** Rows animate visibly on reorder; suppressed with motion preference ✓

---

### Task M6-T04: CI — SPDX MPL-2.0 header audit `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | CI/CD |
| **Story Reference** | NFR — Cross-cutting |
| **Files** | `.github/workflows/` (extend existing) |
| **Dependencies** | None |
| **Order** | `[P]` |

Add a `reuse lint` step (REUSE tool) to the GitHub Actions CI pipeline. Fail PR if any modified `frontend/src/**` or `classquiz/**` file is missing an SPDX MPL-2.0 header.

**Done When:** CI step fails on a test file missing SPDX header; passes on compliant files ✓

---

### Task M6-T05: CI — axe-core accessibility scan `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | CI/CD |
| **Story Reference** | NFR — Accessibility (WCAG AA) |
| **Files** | `.github/workflows/`, Playwright test file |
| **Dependencies** | M2 complete |
| **Order** | `[P]` |

Add `@axe-core/playwright` to frontend dev dependencies. Playwright test: scan each redesigned route (`/`, `/play`, `/dashboard`, `/edit`, host game screen) with axe. Fail if any rule violation > 0. Run in CI on PR.

**Done When:** All routes pass axe scan with 0 violations; CI fails on introduced violation ✓

---

### Task M6-T06: CI — Lighthouse mobile performance `[PENDING]`

| Attribute | Detail |
|-----------|--------|
| **Component** | CI/CD |
| **Story Reference** | NFR — Performance |
| **Files** | `.github/workflows/`, `lighthouserc.json` (new) |
| **Dependencies** | M2 complete |
| **Order** | `[P]` |

Add `@lhci/cli` (Lighthouse CI) to CI pipeline. Configure mobile simulation runs on `/play`, `/dashboard`, `/`. Fail if Lighthouse performance score < 90 or accessibility score < 90.

**Done When:** CI runs Lighthouse; fails on score below threshold; passes for compliant pages ✓

---

### Milestone 6 Verification

- [ ] All M6 tasks marked COMPLETE
- [ ] Animations play at ≥ 60fps (manual verification)
- [ ] All animations suppressed with `prefers-reduced-motion: reduce`
- [ ] CI SPDX, axe-core, and Lighthouse gates all passing on main branch

**Milestone completed on:** PENDING

---

## Traceability Matrix

| PRD Story | Title | Implementation Tasks | Status |
|-----------|-------|---------------------|--------|
| E1-S1 | WCAG audit | M1-T01 | PENDING |
| E1-S2 | @theme tokens | M1-T02 | PENDING |
| E1-S3 | Outfit typography | M1-T02 | PENDING |
| E1-S4 | Button component | M1-T03 | PENDING |
| E1-S5 | Card component | M1-T04 | PENDING |
| E2-S1 | PIN entry | M2-T01 | PENDING |
| E2-S2 | Lobby screen | M2-T02 | PENDING |
| E2-S3 | Question screen | M2-T02 | PENDING |
| E2-S4 | Player leaderboard | M2-T02 | PENDING |
| E2-S5 | Player results | M2-T02 | PENDING |
| E2-S6 | Disconnect/reconnect | M2-T01, M2-T03 | PENDING |
| E3-S1 | Host lobby | M2-T04 | PENDING |
| E3-S2 | Host question display | M2-T05 | PENDING |
| E3-S3 | Host answer tally | M2-T05, M4-T07 | PENDING |
| E3-S4 | Host leaderboard | M2-T06 | PENDING |
| E3-S5 | Winner screen | M2-T06 | PENDING |
| E4-S1 | Global dark base | M1-T02 | PENDING |
| E4-S2 | Navigation | M3-T01 | PENDING |
| E4-S3 | Homepage | M3-T02 | PENDING |
| E5-S1 | Editor layout | M5-T01 | PENDING |
| E5-S2 | MC question card | M5-T02 | PENDING |
| E5-S3 | Quiz settings form | M5-T02 | PENDING |
| E5-S4 | Question type menu | M5-T02 | PENDING |
| E6-S1 | Answer flash animation | M6-T01 | PENDING |
| E6-S2 | Score animation | M6-T02 | PENDING |
| E6-S3 | Leaderboard animation | M6-T03 | PENDING |
| E6-S4 | prefers-reduced-motion | M1-T05 | PENDING |
| E7-S1 | Dashboard | M5-T03 | PENDING |
| E7-S2 | Results summary | M5-T04 | PENDING |
| E7-S3 | Profile/settings | M5-T04 | PENDING |
| E8-S1 | Open-ended backend | M4-T01, M4-T02 | PENDING |
| E8-S2 | LLM evaluation | M4-T03, M4-T04, M4-T05 | PENDING |
| E8-S3 | Player text input | M4-T06 | PENDING |
| E8-S4 | Host verdict display | M4-T07 | PENDING |
| E8-S5 | Manual review fallback | M4-T04, M4-T07 | PENDING |

---

## Insights Reference

**Companion Document:** [specs/insights/implementation-plan-insights.md](insights/implementation-plan-insights.md)

1. **M0 is additive** — The ORM migration (M0-T02, M0-T03) is the highest-risk task in the plan. Run the full test suite after EACH router file is migrated, not at the end.
2. **M4-T04 circuit-breaker is the top concern** — Verified by approver in Round 2. The `try/except LLMUnavailableError` pattern in the socket handler is the guarantee that the game loop never hangs.
3. **Runes interleave risk** — Each story task migrates + redesigns simultaneously. Per-component checklist gates (migrate → test → style → test) reduce the risk of introducing Runes bugs during visual changes.

---

## Phase Gate Approval

- [x] Every Must Have PRD story maps to at least one implementation task
- [x] Task dependencies form a valid DAG (no circular dependencies)
- [x] M0 (backend prep) dependencies are ordered before M1
- [x] Human has explicitly approved this plan for Phase 4 handoff

**Approved by:** Jo Otey
**Approval date:** 2026-02-24
**Status:** Approved

---

## Linked Data

```json-ld
{
  "@context": { "js": "https://jumpstart.dev/schema/" },
  "@type": "js:SpecArtifact",
  "@id": "js:implementation-plan",
  "js:phase": 3,
  "js:agent": "Architect",
  "js:status": "Draft",
  "js:version": "1.0.0",
  "js:upstream": [
    { "@id": "js:prd" },
    { "@id": "js:architecture" }
  ],
  "js:downstream": [],
  "js:traces": []
}
```
