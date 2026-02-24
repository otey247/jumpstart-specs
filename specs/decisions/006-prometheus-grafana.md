# ADR-006: Add Prometheus + Grafana for Operator Metrics

## Status
Accepted

## Context

The current observability stack is Sentry (error tracking) + Plausible (page analytics). Elicitation Round 2 confirmed the operator wants **Sentry + Prometheus/Grafana** for production monitoring. The budget constraint is "minimise cloud spend" — self-hosted Prometheus + Grafana added to the existing Docker Compose stack satisfies this at zero additional cloud cost.

With E8 (LLM integration) introducing an external I/O step in the live game loop, operator visibility into LLM call latency, timeout rates, and verdict distribution becomes important for incident response.

## Decision

Add `prometheus` and `grafana` services to `docker-compose.yml`. Instrument the FastAPI backend using `prometheus-fastapi-instrumentator` to expose a `/metrics` endpoint.

**Key metrics to expose:**
- HTTP request latency (by route) — P50, P90, P99
- HTTP error rate (4xx, 5xx)
- Socket.IO connection count (custom gauge)
- LLM evaluation call count (custom counter)
- LLM evaluation latency (custom histogram: `llm_eval_duration_seconds`)
- LLM timeout/failure count (custom counter: `llm_eval_errors_total`)

A pre-built Grafana dashboard JSON is committed at `grafana/dashboards/partyquiz.json` so operators have a working dashboard on first launch.

**Prometheus scrape config** targets `fastapi:8000/metrics` at 15s interval.

## Consequences

### Positive
- Zero cloud cost — self-hosted in the existing Docker Compose stack
- LLM latency and failure visibility enables rapid incident detection for E8 issues
- Grafana dashboard available immediately on stack start
- Sentry remains for detailed error traces; Prometheus covers aggregate metrics

### Negative
- Two additional Docker containers (Prometheus + Grafana) increase memory footprint on the host
- Grafana dashboard requires initial setup and provisioning; must be maintained alongside feature changes
- `prometheus-fastapi-instrumentator` adds a new backend dependency

### Neutral
- Prometheus and Grafana are industry-standard self-hosted monitoring tools with extensive community support
- Operators who do not need metrics can skip starting these services — the main application works without them

## Alternatives Considered

### Keep Sentry only
- Pros: No additional services; simpler stack
- Cons: Sentry tracks errors, not aggregate latency metrics; no visibility into LLM timeout rates or game loop performance
- Reason rejected: Approver explicitly requested Prometheus + Grafana in elicitation Round 2

### Cloud-hosted metrics (Datadog, New Relic, Grafana Cloud)
- Pros: Reduced operational overhead
- Cons: Cost; contradicts "minimise cloud spend" requirement
- Reason rejected: Budget constraint rules out paid cloud monitoring services
