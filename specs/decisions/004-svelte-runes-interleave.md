# ADR-004: Svelte 4→5 Runes Migration: Interleave Per-Component Strategy

## Status
Accepted

## Context

The frontend is mid-migration from Svelte 4 stores (`stores.ts`) to Svelte 5 Runes (`stores.svelte.ts`). Both patterns currently coexist in the codebase. The PRD mandates completing the migration as part of this redesign phase.

Two strategies were evaluated during Phase 2 PM elicitation:
1. **Big-bang migration first** — Migrate all components to Runes before any redesign work begins
2. **Interleave per-component** — Migrate each component to Runes at the same time it receives its redesign treatment (one pass per component)

The project approver confirmed **interleave per-component** in Phase 2 Round 1.

## Decision

Migrate each Svelte component from Svelte 4 stores to Svelte 5 Runes syntax (`$state`, `$derived`, `$props`, `$effect`) **in the same task as its visual redesign**. No component is redesigned on Svelte 4 syntax and later migrated separately.

**Task pattern for every story stage:**
1. Locate existing route/component
2. Read current Svelte 4 store usage in that file
3. Migrate to Svelte 5 Runes: replace `writable`/`derived` imports with `$state`/`$derived`; replace `$store` reactive references with Rune equivalents
4. Apply new "High-Voltage Arcade" design (tokens, layout, accessibility)
5. Verify `stores.ts` no longer imported in the migrated file
6. Write/update tests

When all PRD stories are complete, `stores.ts` (legacy Svelte 4) should have zero imports and can be deleted.

## Consequences

### Positive
- One pass per component — minimises total files touched vs. migrating separately then redesigning
- Each component exits the task in a completed, modern state — no half-migrated files
- Reduces the period during which both Svelte 4 and Svelte 5 patterns coexist (eliminated progressively)

### Negative
- Each story task is more complex: developer makes two simultaneous changes (Runes migration + visual redesign)
- Risk of introducing Runes bugs while also changing visual logic — tests must cover both
- Some Runes patterns require care (e.g., `$effect` vs. reactive statements)

### Neutral
- Legacy `stores.ts` remains until all stories are complete; the coexistence period is bounded by the implementation plan milestone gates

## Alternatives Considered

### Big-bang Runes migration first
- Pros: Clears the board before redesign; simpler per-story tasks
- Cons: Requires touching every component twice; high upfront risk; delays the first visible redesign milestone
- Reason rejected: Approver selected interleave strategy; double-touching files increases total change volume unnecessarily
