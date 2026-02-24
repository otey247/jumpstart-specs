---
id: challenger-brief
phase: 0
agent: Challenger
status: Draft
created: "2026-02-24"
updated: "2026-02-24"
version: "1.0.0"
approved_by: null
approval_date: null
upstream_refs:
  - specs/codebase-context.md
dependencies: []
risk_level: medium
owners:
  - Jo Otey
sha256: null
---

# Challenger Brief

> **Phase:** 0 — Problem / Challenge Discovery
> **Agent:** The Challenger
> **Status:** Approved
> **Created:** 2026-02-24
> **Approval date:** 2026-02-24
> **Approved by:** Jo Otey

---

## Original Statement

> Changing the style to follow something more similar to the following. Here is a breakdown of the specific style elements:
>
> **1. Color Palette: "High-Voltage Arcade"**
> The palette uses a dark mode base with highly saturated, specific accent colors designed to pop against the background. It avoids gradients in favor of flat, bold tones.
> - Background: `quiz-dark` (#0f1012) — A near-black, deep charcoal
> - Electric Yellow: #FFD02F (emphasis and primary actions)
> - Vibrant Green: #00E055 (success states, positive actions)
> - Strong Blue: #2D8CFF (information, neutral actions)
> - Punchy Red: #FF4545 (danger, errors, or "red team" identifiers)
> - Pure Black (#000000) borders and shadows against White text elements
>
> **2. UI/UX Style: "Tactile Neo-Brutalism"**
> - Hard Shadows: `box-shadow: 4px 4px 0px 0px #000` (no soft/blurred shadows)
> - Interaction: buttons physically move (`translate-y-1`) on click, shadow disappears
> - Typography: 'Outfit' font, uppercase, heavy weights (700/900), wide letter-spacing
> - Container Style (Bento Grids): `border-2`, `rounded-3xl`, distinct compartmentalization
> - Animations: `animate-bounce`, `pulse`, entrance animations (zoom-in, fade-in)
>
> In Summary: looks like a modern, high-energy mobile game or trendy SaaS landing page (similar to Gumroad/Figma's recent marketing). It prioritizes fun and clarity over professionalism or minimalism.

**Follow-up context (gathered during elicitation):**

> The current app is functionally sound. The redesign is motivated by user complaints about the existing green and brown color scheme looking "ugly and outdated." The goal is to establish the right design foundation before adding new features, in preparation for commercializing and scaling the platform.

---

## Assumptions Identified

| # | Assumption | Category | Status | Evidence / Notes |
|---|-----------|----------|--------|-----------------|
| 1 | The current ClassQuiz-inherited visual style is the primary reason PartyQuiz doesn't feel right for party/social contexts | Problem | Believed | Users have complained about aesthetics specifically, but whether style alone is the root barrier is unconfirmed |
| 2 | Target party/social users prefer a high-energy, dark arcade/game-show aesthetic | User | Validated | Human has direct evidence from user feedback or interaction |
| 3 | The specific color palette (#FFD02F, #00E055, #2D8CFF on #0f1012) meets WCAG AA contrast requirements | Feasibility | Believed | Not yet verified against accessibility standards |
| 4 | The ongoing Svelte 4→5 migration won't create additional conflict or risk when combined with a structural UX redesign | Feasibility | Believed | The two parallel changes are both in-flight; interaction risk is unquantified |
| 5 | The scope includes UX structure changes, not just CSS/styling | Scope | Clarified | Human explicitly confirmed: "I would like to change the structure as well to align with the style" |
| 6 | Visual + structural redesign is sufficient to differentiate PartyQuiz from ClassQuiz for commercial purposes | Value | Validated | Human has evidence this creates competitive advantage |
| 7 | A comprehensive system-level design overhaul (tokens, components, typography) is the right approach rather than incremental restyling | Solution | Believed | Motivated by desire to avoid per-page CSS technical debt |

**Summary:** 1 validated directly, 1 validated (value), 1 scope-clarified as broader, 4 believed, 1 untested (accessibility).

---

## Root Cause Analysis (Five Whys)

**Starting point:** PartyQuiz's design doesn't feel right for its party/social quiz positioning.

**Analysis Chain:**

1. **Why?** Users have complained the design is ugly and outdated, specifically disliking the existing green and brown color scheme inherited from ClassQuiz.
2. **Why?** An ugly/outdated aesthetic creates negative word-of-mouth and reduces engagement — people don't want to work with or endorse the platform.
3. **Why?** This matters because the project is moving toward commercialization and needs to be well-accepted at scale; the UI is currently a gap that undermines that goal.
4. **Why?** The app is functionally healthy, so UI is the first and most visible barrier — and since new features are coming, getting the design foundation right now prevents each new feature from launching on a broken aesthetic base.

**Root cause reached at 4 layers (no artificial 5th needed).**

**Logic Check (Working Backwards):**
> The platform lacks a visual identity that matches its social/party purpose and commercial ambitions **therefore** first impressions are poor **therefore** negative word-of-mouth and disengagement **therefore** the design feels wrong and blocks adoption and growth.
>
> *This chain holds without logical leaps.*

**Root Cause Identified:**
> PartyQuiz's visual identity is inherited from a general-purpose educational tool and has never been adapted to match its actual positioning — a social, high-energy party quiz platform aimed at commercial scale. The mismatch creates a first-impression barrier that undermines both user adoption and commercial credibility before any interaction with the platform's features.

**Alternative branches considered:**
- Could the problem be UX flows, not aesthetics? → Rejected. Users specifically complained about visual style, not navigation or usability.
- Could it be feature gaps? → Rejected. Human confirmed the platform functions as expected.

---

## Stakeholder Map

| Stakeholder | Relationship to Problem | Impact Level | Current Workaround |
|-------------|------------------------|--------------|-------------------|
| Party/social quiz players | Experience the visual mismatch directly; sources of negative word-of-mouth | High | Tolerate or abandon the platform |
| Quiz creators / hosts | First contact with the UI; visual quality influences whether they adopt the platform | High | Use competitor platforms (Kahoot, etc.) |
| Jo Otey (product owner) | Building toward commercialization; UI is a blocker to that goal | High | In-progress redesign (this effort) |
| Potential commercial customers | Evaluate the platform on first impression; current aesthetic undermines purchase intent | High | None — they simply don't convert |
| Future contributors / developers | Will build new features and components on top of the design system | Medium | Currently building on inconsistent foundation |

**Missing stakeholders check:** Confirmed by human — no stakeholders are missing.

**Adversely affected parties:** None identified by the human at this stage.

---

## Reframed Problem Statement

**Reframe options presented:**

1. **A (Identity-credibility gap):** PartyQuiz's visual identity doesn't match its social/party positioning — users dismiss it before experiencing its functionality, blocking adoption and commercial credibility.
2. **B (Design foundation debt):** Without a cohesive design system, every new feature ships looking mismatched, creating UI debt that worsens as the platform grows toward commercialization.
3. **C (Competitive differentiation):** PartyQuiz is functionally sound but indistinguishable from the free educational tool it was forked from, making it impossible to position or price as a distinct premium social product.

**Selected problem statement:**

> **PartyQuiz's visual identity doesn't match its social/party positioning — users dismiss it before experiencing its functionality, blocking adoption and commercial credibility.**

---

## Validation Criteria

How will we know the problem has been solved?

| # | Criterion | Type | Measurable? |
|---|-----------|------|------------|
| 1 | Users stop complaining about the design in feedback and word-of-mouth | Behavioral | Yes — qualitative tracking of feedback themes |
| 2 | New user retention improves (people return after their first session) | Metric | Yes — measurable via session/retention analytics |
| 3 | A presentation or live demo generates a positive first-impression reaction without prompting | Qualitative | Yes — observable in demo/sales contexts |
| 4 | All new features can be built using design tokens without requiring custom per-page CSS | Metric | Yes — measurable during development (percentage of new work using tokens) |

---

## Constraints and Boundaries

### Explicitly Out of Scope
- None declared by the human. **This is a wide-scope effort** — UX structural changes are included alongside visual restyling.

### Non-Negotiable Constraints
- None declared. Human is open to best recommendations on tooling and approach.

### Important Contextual Constraints (from Codebase Context)
- The Svelte 4→5 Runes migration is currently **in-flight** — both `stores.ts` and `stores.svelte.ts` coexist. A simultaneous UX restructure increases the risk of compound breaking changes.
- TailwindCSS 4 (PostCSS plugin, config-file-free) is already in use — any design token approach must be compatible with TailwindCSS 4's CSS variable and `@theme` directive model.
- `MAX_WORKERS=1` constraint means the backend is not the bottleneck — frontend work can proceed independently.
- All frontend files must carry SPDX MPL-2.0 headers (REUSE compliance).

### Known Unknowns
- Whether the specific proposed color palette passes WCAG AA contrast — requires audit before implementation.
- How deep the UX structural changes go — not yet quantified; the Analyst phase will scope this.
- What the new page/feature structure looks like — to be defined in Phase 1 (Product Brief) and Phase 2 (PRD).
- How to sequence the Svelte 4→5 migration completion relative to the design system work.

---

## Insights Reference

**Companion Document:** [specs/insights/challenger-brief-insights.md](insights/challenger-brief-insights.md)

Key insights that shaped this document:

1. **Solution-first framing** — The original statement was entirely a solution spec (colors, fonts, shadows). Interrogation revealed the problem underneath: a visual identity mismatch blocking commercial adoption.
2. **Scope is wider than a restyle** — The human explicitly confirmed UX structure changes are included, making this a broader effort than the original statement implied.
3. **Two in-flight migrations** — The Svelte 4→5 migration and the design redesign are both happening simultaneously, which the human believes won't conflict but has not validated.

See the insights document for complete decision rationale, alternatives considered, and questions explored.

---

## Phase Gate Approval

- [x] Human has reviewed this brief
- [x] Problem statement is specific and testable
- [x] At least one validation criterion is defined
- [x] Constraints and boundaries section is populated
- [x] Human has explicitly approved this brief for Phase 1 handoff

**Approved by:** Jo Otey
**Approval date:** 2026-02-24
**Status:** Approved

---

## Linked Data

```json-ld
{
  "@context": { "js": "https://jumpstart.dev/schema/" },
  "@type": "js:SpecArtifact",
  "@id": "js:challenger-brief",
  "js:phase": 0,
  "js:agent": "Challenger",
  "js:status": "Draft",
  "js:version": "1.0.0",
  "js:upstream": [
    { "@id": "js:codebase-context" }
  ],
  "js:downstream": [
    { "@id": "js:product-brief" }
  ],
  "js:traces": []
}
```
