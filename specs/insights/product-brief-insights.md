# Product Brief — Insights Log

> **Phase:** 1 — Analysis
> **Agent:** The Analyst
> **Parent Artifact:** [`specs/product-brief.md`](../product-brief.md)
> **Created:** 2026-02-24
> **Last Updated:** 2026-02-24

---

## Insight Entries

---

### INS-001 · 🔍 Observation · 2026-02-24T00:00:00Z

**Scope expanded again during elicitation — this is a platform overhaul, not a restyle**

Phase 0 already expanded the scope from CSS-only to "CSS + structure." During Phase 1 elicitation, the human confirmed that gamification mechanics, new question types, and a full editor redesign are all in scope. Combined with the device split (mobile players + desktop/TV hosts), this is now a substantial multi-surface redesign with feature additions. The Must Have / Should Have split in the scope recommendation is deliberately conservative to create a viable sequencing path — but the human's intent is comprehensive.

→ See [Scope Recommendation](../product-brief.md#scope-recommendation)

---

### INS-002 · 🎯 Assumption · 2026-02-24T00:00:00Z

**The player-on-mobile / host-on-desktop split is a first-class design architecture decision**

When the human confirmed "Players on mobile, host on desktop/TV," this established that PartyQuiz has two distinct primary surfaces that need to be designed independently and intentionally. This is the same model Jackbox uses (TV + "joystick" phone). It means the Architect needs to decide how the SvelteKit routing and layout system handles these two contexts — a single responsive layout may not be sufficient for the TV host screen, which needs very different information density and visual hierarchy.

→ See [User Journeys](../product-brief.md#user-journeys) and [Constraints and Boundaries](../product-brief.md#constraints-and-boundaries)

---

### INS-003 · 💡 Decision · 2026-02-24T00:00:00Z

**Gamification scope bounded to animation-only for this phase**

The human confirmed gamification is "in scope" but did not specify mechanics. The Scout's codebase context showed no existing gamification data model (no XP, no badges, no persistent user scoring beyond game results). Adding persistent gamification mechanics would require database schema changes and backend work — which the human explicitly wants to avoid in this phase. The Product Brief therefore bounds gamification to animation-level effects (answer feedback, score reveal, leaderboard animation) to preserve the no-backend-changes boundary. This is a scope decision made by the Analyst that should be confirmed with the human during PRD.

→ See [Should Have #2](../product-brief.md#should-have) and [Won't Have #4](../product-brief.md#wont-have-this-redesign-phase)

---

### INS-004 · 💡 Decision · 2026-02-24T00:00:00Z

**New question types bounded to design-only in this phase**

Similarly, the human said new question types are "in scope" but the Scout confirmed that adding them would require socket server changes (the answer validation logic is type-specific) and database schema evolution. Adding backend-coupled new question types while also redesigning the UI is compound risk. The Product Brief recommends building type-agnostic component shells (so the design system is ready) while deferring the backend question type implementation to a distinct feature phase. This decision needs explicit confirmation in the PRD.

→ See [Could Have #1](../product-brief.md#could-have) and [Won't Have #3](../product-brief.md#wont-have-this-redesign-phase)

---

### INS-005 · ⚠️ Risk · 2026-02-24T00:00:00Z

**WCAG AA audit is a blocking prerequisite for implementation**

The Human's Phase 0 assumption on WCAG compliance was "Believed." The specific palette (#FFD02F on #0f1012, #00E055 on #0f1012) needs a formal audit before the Architect encodes these as design tokens. If any colour fails AA for normal text, the tokens will be established incorrectly and all components built against them will later need rework. The Product Brief explicitly marks this as a `[NEEDS CLARIFICATION]` item for the Architecture phase.

Rough estimates (not verified): #FFD02F on #0f1012 ≈ ~14:1 contrast (likely passes); #00E055 on #0f1012 ≈ ~8:1 (likely passes); #FF4545 on #0f1012 needs checking. But estimates are insufficient — a tool-verified audit is required.

→ See [Open Questions](../product-brief.md#open-questions) and [Risks](../product-brief.md#risks-to-the-product-concept)

---

### INS-006 · 🔍 Observation · 2026-02-24T00:00:00Z

**The competitive gap PartyQuiz can uniquely own is "open + party-grade"**

Analysis of the competitive landscape shows a clear market gap. Kahoot owns accessible + well-designed but is closed/educational. Jackbox owns party energy but is closed/paid/non-customizable. No open-source, self-hostable platform operates at party-grade visual quality. This is PartyQuiz's defensible differentiator, and it maps directly to the commercial goal from Phase 0: a platform that looks premium, is open, and does what Jackbox and Kahoot can't simultaneously offer.

→ See [Competitive Landscape](../product-brief.md#competitive-landscape) and [Value Proposition](../product-brief.md#value-proposition)

---

### INS-007 · ⚠️ Risk · 2026-02-24T00:00:00Z

**Svelte 4→5 migration sequencing is an architectural decision that must be made before work begins**

The Scout identified that both `stores.ts` (Svelte 4) and `stores.svelte.ts` (Svelte 5 Runes) coexist. The Analyst recommends a per-component sequencing strategy: complete the Runes migration for a component at the same time as its visual redesign, rather than redesigning on Svelte 4 and then migrating again. However, this recommendation must be validated by the Architect — if the migration is more complex than component-level, it may change the order of operations significantly.

→ See [Open Questions](../product-brief.md#open-questions) and [Won't Have #5](../product-brief.md#wont-have-this-redesign-phase)

---

### INS-008 · 🔍 Observation · 2026-02-24T00:00:00Z

**Outfit font GDPR consideration is a small but real compliance question**

The Challenger Brief specified the 'Outfit' font. Loading from Google Fonts sends user IP addresses to Google's servers, which is a GDPR concern for EU users (who are likely given the .de domain history). Self-hosting the font files avoids this. This is a low-complexity architectural decision but should be flagged to the Architect, who can route through the `@fontsource/outfit` npm package or a similar approach.

→ See [Open Questions](../product-brief.md#open-questions)
