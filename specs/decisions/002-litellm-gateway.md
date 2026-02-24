# ADR-002: LiteLLM Gateway for E8 LLM Evaluation

## Status
Accepted

## Context

Epic E8 requires an LLM to judge free-text player answers during live games. The PRD mandates LiteLLM as the abstraction layer. Four provider approaches were evaluated during Phase 3 elicitation:
1. Ollama (self-hosted, GDPR-safe)
2. OpenAI `gpt-4o-mini` (cloud, DPA available)
3. Anthropic Claude (cloud, DPA available)
4. LiteLLM gateway (operator-configurable)

**GDPR consideration:** Player answer text constitutes user-generated personal data. If transmitted to a cloud LLM, the operator must have a Data Processing Agreement (DPA) with the provider. Self-hosted Ollama eliminates this concern entirely.

**Latency NFR:** LLM verdict must arrive within 5s p95. If the LLM is unavailable or times out, the game must not stall — a manual review fallback is required.

The project approver selected: **LiteLLM gateway (operator-configurable)**. The architecture stays provider-neutral; each deployment sets `LITELLM_MODEL` to choose its own provider.

## Decision

Implement the LLM evaluation integration as a **standalone `classquiz/llm/` module** (Library-First, Article I) wrapping LiteLLM's `acompletion` async API:

```python
# classquiz/llm/__init__.py — public API
async def evaluate_answer(
    question: str,
    answer: str,
    rubric: str | None = None
) -> Verdict: ...
```

**Configuration (environment variables):**
- `LITELLM_MODEL` — model identifier (e.g., `ollama/llama3.1`, `gpt-4o-mini`)
- `LITELLM_API_BASE` — base URL (e.g., `http://ollama:11434` for self-hosted Ollama)
- `LITELLM_API_KEY` — provider API key (not required for Ollama)
- `LLM_TIMEOUT_SECONDS` — max wait in seconds (default: 4)

**Circuit-breaker pattern:** `evaluate_answer()` raises `LLMUnavailableError` on timeout or provider error. The Game Engine catches `LLMUnavailableError` and emits a `manual_review_required` socket event to the host, triggering manual override mode. The game loop never hangs.

**API key never hardcoded.** All credentials are injected via environment variables.

## Consequences

### Positive
- Provider-neutral: operators can use Ollama (GDPR-safe, no cost), OpenAI, Anthropic, or any LiteLLM-supported model without code changes
- Isolated module: `classquiz/llm/` has a clean public API; game engine does not depend on LiteLLM directly
- Circuit-breaker guarantees game loop resilience — LLM unavailability is a graceful degradation, not a crash
- `acompletion` is fully async — no blocking I/O in the socket server event loop

### Negative
- Operators must configure `LITELLM_MODEL` and choose an LLM provider before E8 features work; the fallback mode (manual review) activates if unconfigured
- Self-hosted Ollama requires a capable host (GPU recommended for < 4s p95); not all deployment environments have this
- Cloud providers incur per-request cost

### Neutral
- GDPR compliance responsibility is shifted to the operator/deployer via provider selection
- LiteLLM adds a new production dependency; versioned at `^1.81`

## Alternatives Considered

### Ollama only (force self-hosted)
- Pros: GDPR-safe by default; no API key required
- Cons: Requires GPU/capable hardware; not viable for resource-constrained hosters; locks out cloud providers
- Reason rejected: The operator-configurable gateway approach is a superset — operators can still choose Ollama

### OpenAI gpt-4o-mini only
- Pros: Fast, high quality, established DPA
- Cons: External dependency, cost per request, data leaves infrastructure by default
- Reason rejected: Operator-configurable gateway is strictly more flexible
