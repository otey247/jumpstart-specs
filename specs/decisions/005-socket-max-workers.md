# ADR-005: Socket Server MAX_WORKERS=1 Constraint Acknowledged

## Status
Accepted

## Context

The `docker-compose.yml` documents `MAX_WORKERS: "1"` with comment: *"Very important and DON'T CHANGE"*. The socket server (`classquiz/socket_server/`) uses `socketio.AsyncServer` with the default **in-memory adapter**. Room membership, SID mappings, and player-to-connection state are held in the server process RAM.

Multiple gunicorn/uvicorn workers would each maintain **independent in-memory state**. Cross-process socket delivery (e.g., broadcasting to a room from worker A when the player's socket is in worker B) would silently fail. The E8 LLM evaluation feature adds async I/O to the game loop but does **not** introduce new in-memory state that would further bind the constraint.

## Decision

Accept the `MAX_WORKERS=1` constraint as the architectural operating condition for this project phase. The constraint is **explicitly documented** and **not to be changed** without first implementing a Redis-based socket.io adapter.

**Future migration path (out of scope for this PRD):**
If horizontal scaling becomes a requirement, the migration steps are:
1. Add `python-socketio[asyncio_client]` to `Pipfile`
2. Configure `socketio.AsyncRedisManager(redis_url)` as the `message_queue` parameter in `AsyncServer` instantiation
3. Re-enable `MAX_WORKERS > 1`
4. This change requires a dedicated migration task and regression testing of the full game loop at multi-worker concurrency

This path is documented here so it is not lost; it is a deliberate future ADR when the requirement arises.

## Consequences

### Positive
- Zero risk of silent multi-process socket delivery failures
- No additional Redis infrastructure complexity required now
- E8 LLM integration remains safe: all async I/O happens within the single event loop

### Negative
- The system is limited to vertical scaling — a larger single host instance
- Concurrency ceiling determined by a single Python async event loop processing socket events

### Neutral
- This constraint predates the current PRD and is intentional; it is being formally acknowledged rather than introduced

## Alternatives Considered

### Implement Redis socket adapter now
- Pros: Enables `MAX_WORKERS > 1`; better concurrency headroom
- Cons: Significant infrastructure change mid-redesign phase; increases testing complexity; out of scope for moderate risk appetite
- Reason rejected: Not required by any PRD NFR; defer to a future phase when horizontal scaling is a concrete need
