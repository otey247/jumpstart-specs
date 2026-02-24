# PRD — Insights Log

> **Phase:** 2 — Planning
> **Agent:** The Product Manager
> **Parent Artifact:** [`specs/prd.md`](../prd.md)
> **Created:** 2026-02-24
> **Last Updated:** 2026-02-24

---

## Insight Entries

---

### INS-001 · ⚠️ Scope · 2026-02-24T00:00:00Z

**E8 overrides the Product Brief's Won't Have boundary — this changes the risk profile significantly**

The Product Brief explicitly placed "new question type backend implementation" in Won't Have for the redesign phase. During Round 3 elicitation, the approver introduced a new requirement: LLM-judged open-ended questions via LiteLLM, real-time during gameplay. This was confirmed as in-scope for this PRD, overriding the prior boundary.

The implications are material:
1. Backend changes are now required (`classquiz/socket_server/`, new `classquiz/llm/` module, one Alembic migration)
2. An external service dependency (LLM provider via LiteLLM) is introduced — with GDPR implications for EU player data
3. A latency NFR (< 5s p95 verdict) is added to the live game loop, which now has an external I/O step
4. The Architecture phase must select a provider and evaluate DPA availability before implementation begins

The Architect should seriously evaluate Ollama (local model) as a GDPR-safe alternative to cloud providers.

→ See Epic E8 and Risk Register items 4 and 5 in `specs/prd.md`

---

### INS-002 · 💡 Decision · 2026-02-24T00:00:00Z

**E6-S4 (prefers-reduced-motion) was classified Must Have despite living in a Should Have epic**

E6 (Gamification & Animations) is a Should Have epic. However, E6-S4 — implementing `@media (prefers-reduced-motion: reduce)` compliance across all animated components — is a WCAG 2.1 AA requirement, specifically Success Criterion 2.3.3 (Animation from Interactions). Since the PRD commits to WCAG 2.1 AA across all pages, any animated story that doesn't implement this is technically non-compliant.

The story was kept in E6 for logical grouping but flagged Must Have to ensure it is in the token/foundation scope of Milestone 1, not deferred to Milestone 6 with the other E6 animations. The Notes field on E6-S4 makes this explicit.

→ See Story E6-S4, Milestone 1 (includes E6-S4), NFR Accessibility section

---

### INS-003 · 🔍 Observation · 2026-02-24T00:00:00Z

**The E1-S1 WCAG audit was elevated to a Stage 1 blocking gate, not just a story**

The PM instructions allow requirements to appear as stories. However, the WCAG audit is uniquely a prerequisite that blocks every downstream token value — if #FFD02F fails contrast and must be adjusted to #FFDB2F, every task in Stage 2 that references the colour is affected. Treating it as an ordinary story (which could be done in parallel with other work) would create rework risk.

The decision was made to create a discrete Stage 1 in the task breakdown (before Stage 2 Foundational) that is a single hard gate. No developer task in Stage 2 references specific colour hex values — they reference token names — but the token values themselves must be locked before the `@theme` file is written. Stage 1 ensures this happens first.

→ See Stage 1 in Task Breakdown, Story E1-S1

---

### INS-004 · 💡 Decision · 2026-02-24T00:00:00Z

**Epic boundary: E2 (player) vs E3 (host) separation was confirmed correct by the approver**

An alternative grouping considered during epic definition was merging E2 and E3 into a single "Live Game Screens" epic, since host and player views share underlying game state and socket events. This was presented in Round 1 as an option ("Merge E2 and E3") and rejected by the approver.

The rationale for keeping them separate:
- The layout and design constraints are fundamentally different (375px touch vs. 1280px display)
- The approver identified Alex and Sam as separate primary personas for each epic
- Developer stories can be assigned independently per surface, reducing context switching
- Milestones can ship player-only or host-only reskins independently if needed

→ See Epics E2, E3

---

### INS-005 · 🔍 Observation · 2026-02-24T00:00:00Z

**Gamification animations were consciously kept Should Have despite being central to the "party feel" goal**

In Round 2, the PM presented the question: "Should E6 animations be promoted to Must Have?" The approver chose to keep them as Should Have. The reasoning accepted: the five critical game paths all function correctly without animations — the animations are an enhancement, not a requirement for functionality.

This is consistent with the Product Brief's "gamification = animation level only" boundary. Milestone 2 (Core Game Loop Reskinned) delivers a fully functional, visually redesigned game experience before any animation layer is added. Milestone 6 layers animations on top.

If animations are deprioritised due to delivery constraints, the core redesign goal (Criterion 1: no UI complaints) is still achievable without them.

→ See Epic E6, Round 2 decisions, Implementation Milestones (M6)

---

### INS-006 · ⚠️ Risk · 2026-02-24T00:00:00Z

**LiteLLM GDPR implications were identified but not resolved — Architecture gate required**

Open-ended player answers submitted through the game loop will contain user-generated text. If the LiteLLM service routes these to a cloud LLM provider (OpenAI, Anthropic), the player answer text is transmitted to a third-party service. For EU-based PartyQuiz instances, this triggers GDPR data processing requirements — specifically, the provider must offer a Data Processing Agreement (DPA) and the user should be informed.

The PM captured this as Risk #4 in the PRD risk register and as a compliance note in the NFR Security section. The resolution is an Architect decision:
- **Recommended path A (privacy-safe):** Use Ollama with a locally-hosted model (e.g., llama3.1 or mistral) — no data leaves the host's infrastructure
- **Recommended path B (cloud with DPA):** OpenAI has a DPA; Anthropic has a DPA. Choose one and add a consent/notice mechanism.

→ See Risk #4, NFR Security Compliance section, E8-S2 Notes

---

### INS-007 · 💡 Decision · 2026-02-24T00:00:00Z

**Svelte 4→5 Runes migration was scoped as a per-component interleave strategy**

The approver confirmed in elicitation: "Interleave — migrate each component as it's redesigned." This means no component will be redesigned on Svelte 4 syntax and then migrated separately. Every story task in Stages 3–10 includes an explicit "migrate to Runes" step in the task breakdown.

Rationale: one-pass per component avoids touching files twice (once for redesign, once for Runes), reducing total change volume and regression risk. However it does increase per-component complexity — the developer is making two changes simultaneously instead of one. Risk #1 in the risk register addresses this with a per-component checklist gate.

The task breakdown pattern is: "Locate existing route/component → migrate to Runes → apply new design → add tests → verify."

→ See Risk #1, all Story stages in Task Breakdown, NFR Svelte migration requirement

---

### INS-008 · 🔍 Observation · 2026-02-24T00:00:00Z

**29 stories across 8 epics is large for a single PRD — milestone gating is essential**

The PRD contains 35 stories (E1:5, E2:6, E3:5, E4:3, E5:4, E6:4, E7:3, E8:5). For a solo builder, this is substantial. The milestone structure (M1–M6) was designed to create natural checkpoints and delivery opportunities:

- **M1** (design system) is a fast, high-value first milestone that unblocks everything
- **M2** (core game loop) is the most impactful milestone for the Criterion 3 demo goal
- **M3** (shell/homepage) is orthogonal to M2 and can be worked in parallel by a developer
- **M4** (LLM questions) gates on M2 stability and is riskiest due to backend changes
- **M5** (editor + dashboard) is lowest-risk and can be deferred without affecting core game validation
- **M6** (animations) is entirely additive and can be last or shipped incrementally

If time pressure emerges, M5 and M6 are the natural deferral candidates without compromising the primary validation criteria.

→ See Implementation Milestones, Success Metrics
