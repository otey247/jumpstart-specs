# ADR-001: Replace ormar ORM with SQLModel

## Status
Accepted

## Context

PartyQuiz uses `ormar` as its async ORM — the library underpinning every database operation across `classquiz/db/models.py` (569 lines) and 15+ router files. `ormar` was deprecated by its author and is no longer actively maintained. The library integrates Pydantic v2, so a future Pydantic v3 release or asyncpg breaking change could create an unmaintainable upgrade path with no upstream fix available.

This phase (PRD redesign) touches the database layer via E8 (new question type) and is an appropriate window to resolve the top dependency risk while the codebase is being actively modified.

Migration risk appetite from elicitation: **Moderate** — fix the top risks; leave further refactoring for a future sprint.

## Decision

Replace `ormar` with **SQLModel** (`^0.0.24`) as the async ORM for all database models and queries.

SQLModel is chosen over plain SQLAlchemy 2.x async because:
- It preserves the Pydantic-first model definition style already used throughout PartyQuiz
- Its API surface is similar to ormar's, minimising rewrite scope
- It is actively maintained by Sebastián Ramírez (FastAPI creator)
- It uses SQLAlchemy 2.x async under the hood — maximum ecosystem compatibility

`Alembic` remains the migration tool. SQLModel exposes `SQLModel.metadata` compatible with Alembic's `autogenerate`, so no migration history is lost.

**Migration scope:**
- `classquiz/db/models.py` — all model classes rewritten with `SQLModel` base and `table=True`
- All router files importing ormar models — query patterns updated to SQLAlchemy `select()` / `session.exec()`
- `classquiz/__init__.py` — replace ormar engine init with `create_async_engine` + `async_sessionmaker`
- Remove `ormar` from `Pipfile`; add `sqlmodel`

## Consequences

### Positive
- Eliminates the top dependency risk: `ormar` (deprecated) is removed from the production dependency tree
- Pydantic v2 compatibility is guaranteed going forward
- Model definitions remain readable and Pydantic-native (minimal learning curve)
- Alembic migration history is fully preserved

### Negative
- Significant rewrite effort: all model classes + all query patterns across 15+ router files
- Test fixtures that build model instances directly must be updated
- Risk of regression if any ormar-specific behaviour is missed in the migration
- SQLModel async support is still maturing; some advanced patterns may require dropping to raw SQLAlchemy

### Neutral
- Alembic `autogenerate` works identically with SQLModel metadata
- The `webauthn==1.*` pin and all other non-ORM dependencies are unaffected

## Alternatives Considered

### Keep ormar for this phase
- Pros: Zero migration risk; no effort
- Cons: The deprecated dependency remains; compounds with each new feature touching the data layer; becomes harder to remove with time
- Reason rejected: The PRD touches the ORM layer (E8 new question type), making this an ideal migration window. Moderate risk appetite allows addressing this.

### SQLAlchemy 2.x async directly (without SQLModel)
- Pros: Maximum flexibility; no ORM abstraction layer
- Cons: Loses Pydantic-native model definitions; significantly more boilerplate; higher rewrite effort; departs from the PartyQuiz coding style
- Reason rejected: SQLModel satisfies the requirement with less effort and preserves the existing code style.
