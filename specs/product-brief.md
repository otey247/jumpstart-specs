---
id: product-brief
phase: 1
agent: Analyst
status: Approved
created: "2026-02-24"
updated: "2026-02-24"
version: "1.0.0"
approved_by: "Jo Otey"
approval_date: "2026-02-24"
upstream_refs:
  - specs/challenger-brief.md
  - specs/codebase-context.md
dependencies:
  - challenger-brief
risk_level: medium
owners:
  - Jo Otey
sha256: null
---

# Product Brief

> **Phase:** 1 — Analysis
> **Agent:** The Analyst
> **Status:** Approved
> **Created:** 2026-02-24
> **Approval date:** 2026-02-24
> **Approved by:** Jo Otey
> **Upstream Reference:** [specs/challenger-brief.md](challenger-brief.md)

---

## Problem Reference

**Reframed Problem Statement (from Phase 0):**

> PartyQuiz's visual identity doesn't match its social/party positioning — users dismiss it before experiencing its functionality, blocking adoption and commercial credibility.

**Validation Criteria (from Phase 0):**

1. Users stop complaining about the design in feedback and word-of-mouth
2. New user retention improves (people return after their first session)
3. A presentation or live demo generates a positive first-impression reaction without prompting
4. All new features can be built using design tokens without requiring custom per-page CSS

---

## Vision Statement

> PartyQuiz becomes the go-to open platform for social quiz experiences — one that hosts reach for when they want the energy of Jackbox and the accessibility of Kahoot, built on a visual language so bold and alive that players expect to have fun before a single question appears.

---

## User Personas

### Persona 1: Alex, the Trivia Night Host

| Attribute | Detail |
|-----------|--------|
| **Role** | Pub quiz organiser, house party host, bar trivia facilitator |
| **Goals** | Run a smooth, impressive game session; look competent in front of friends/guests; manage the flow without IT hassle |
| **Frustrations** | Current UI looks dated on the pub TV; the leaderboard feels anticlimactic; setting up a game on a laptop while people are watching is stressful |
| **Technical Proficiency** | Medium — comfortable with web apps, not a developer |
| **Relevant Context** | Hosts game on a laptop or TV; players are around them in person; first impression is public and social |
| **Current Workaround** | Uses Kahoot despite its classroom feel, or runs a Jackbox game (paid, no custom questions) |
| **Representative Quote** | "I just want to pull it up on the big screen and have everyone go 'whoa, what is that?' — not 'oh, is this for schools?'" |

### Persona 2: Sam, the Social Player

| Attribute | Detail |
|-----------|--------|
| **Role** | Guest at a bar, party, or event; joins via game PIN on their phone |
| **Goals** | Jump in instantly with zero confusion; see their score and ranking in a way that feels fun; not stare at an ugly screen for 30 minutes |
| **Frustrations** | Joining feels clunky; the answer UI doesn't feel like a game; the leaderboard disappears too fast or is hard to read on mobile |
| **Technical Proficiency** | Low to Medium — will abandon if unclear; expects the intuitiveness of a mobile game |
| **Relevant Context** | On a personal smartphone, in a noisy social setting, likely has a drink in hand |
| **Current Workaround** | Tolerates whatever the host picks; will disengage or mock a bad-looking UI openly |
| **Representative Quote** | "It took me three taps to figure out where to type my answer — at a party that's an eternity." |

### Persona 3: Morgan, the Quiz Creator

| Attribute | Detail |
|-----------|--------|
| **Role** | Teacher, trivia enthusiast, event organiser, or power user who builds quizzes in advance |
| **Goals** | Create polished, varied quizzes efficiently; use different question types to keep things interesting; trust that what they build will look good when played |
| **Frustrations** | The current editor UI is functional but visually inconsistent; no confidence that a newly-created quiz will look great at runtime; limited question type variety |
| **Technical Proficiency** | Medium to High — comfortable with structured forms, typing, drag-and-drop |
| **Relevant Context** | Works asynchronously, often on desktop; quality and expressiveness of the editor directly affects how much effort they invest |
| **Current Workaround** | Builds in Kahoot or Google Forms for polished results; tolerates ClassQuiz editor for self-hosting needs |
| **Representative Quote** | "I spent an hour making a great quiz and then felt embarrassed when it looked like a government form on game night." |

> **Persona Evolution:** When new user behaviours or feedback emerge post-approval, create a Persona Change Proposal using `.jumpstart/templates/persona-change.md` and present it for approval before modifying personas.

---

## User Journeys

### Current-State Journey (Primary Persona: Alex, the Trivia Night Host)

| Step | Action | Thinking | Feeling | Pain Point (Severity) |
|------|--------|----------|---------|----------------------|
| 1 | Navigates to PartyQuiz on laptop, opens it in front of guests | "I hope this doesn't look weird on the TV" | Anxious | Homepage looks dated; green/brown palette feels educational, not fun (Critical) |
| 2 | Selects a quiz and clicks Start Game | "OK, let's see the PIN screen" | Tentative | PIN/lobby screen is bare-bones; lacks visual presence on a TV (Moderate) |
| 3 | Guests join on mobile via PIN | "Is everyone in?" | Functional | Join flow works but mobile UI is plain; no energy communicated to players (Moderate) |
| 4 | Starts the first question | "Here we go" | Neutral | Question display is readable but lacks countdown drama or visual energy (Moderate) |
| 5 | Players answer; Alex watches answer stats | "I can't really tell how engaged people are" | Disconnected | No celebratory feedback; answer reveal is quiet and flat (Critical) |
| 6 | Leaderboard shown between questions | "OK, who's winning?" | Neutral | Leaderboard is a simple list; no drama, no animation, nothing to cheer at (Critical) |
| 7 | Game ends; results displayed | "Was it fun? People seemed OK with it" | Uncertain | Results screen feels like a spreadsheet; no shareable moment (Moderate) |

### Future-State Journey (Primary Persona: Alex, the Trivia Night Host)

| Step | Action | Thinking | Feeling | Improvement |
|------|--------|----------|---------|-------------|
| 1 | Opens PartyQuiz; dark arcade UI fills the screen | "Yes — this looks like a game" | Confident | Bold #0f1012 base with electric accents signals 'game night' immediately |
| 2 | Starts game; PIN screen appears with large game code on dark background | "That looks great on the TV" | Proud | PIN screen is designed for TV display — large, high contrast, visually intentional |
| 3 | Guests join; player names appear with entrance animations | "They can see their name pop up!" | Excited | Mobile join flow is clean and fast; each join triggers visible acknowledgement |
| 4 | First question launches with countdown animation | "3… 2… 1… boom" | Engaged | Countdown timer is animated; question slide-in has kinetic energy |
| 5 | Players answer; response counters animate in real time | "Watch the answers come in!" | Delighted | Live answer tally animates; correct answer reveal has a satisfying flash effect |
| 6 | Leaderboard flies in with ranked positions and score changes | "YES! Look at who moved up!" | Thrilled | Animated leaderboard reveal shows position changes; crowd-pleasing |
| 7 | Final results displayed with winner highlight | "This is the moment" | Triumphant | Results are cinematic; winner is celebrated; moment feels shareable |

### Current-State Journey (Secondary Persona: Sam, the Social Player)

| Step | Action | Thinking | Feeling | Pain Point (Severity) |
|------|--------|----------|---------|----------------------|
| 1 | Pulls out phone, navigates to game URL given by host | "What is this? Looks like a school thing" | Sceptical | First impression on mobile is unflattering (Critical) |
| 2 | Enters PIN and name | "OK, I guess I just type here?" | Uncertain | PIN entry UI is functional but lacks game-feel; no feedback on successful join (Moderate) |
| 3 | Waits in lobby | "Why does it look like a spreadsheet?" | Bored | Lobby is a static list; no countdown energy (Moderate) |
| 4 | Sees question and options on mobile | "Which button do I press?" | Confused briefly | Answer buttons are small for thumbs; no visual hierarchy between options (Moderate) |
| 5 | Answers; waits for next question | "Did it register? Am I right?" | Uncertain | Minimal feedback on answer submission; no immediate indication of score impact (Critical) |
| 6 | Leaderboard shown | "Cool, I'm 4th" | Flat | Leaderboard is unexciting; no animation or celebration of movement (Moderate) |

### Future-State Journey (Secondary Persona: Sam, the Social Player)

| Step | Action | Thinking | Feeling | Improvement |
|------|--------|----------|---------|-------------|
| 1 | Opens game URL on phone; sees dark arcade splash | "Oh this looks cool!" | Intrigued | Mobile-first design; dark background + electric accents pop immediately |
| 2 | Enters PIN and name; confetti-style entrance animation plays | "I'm in!" | Fired up | Join confirmation is celebratory; name appears on nearby TV screen |
| 3 | Waits in lobby with animated countdown | "Hurry up, I want to play" | Impatient (good) | Lobby has kinetic energy; animated player list makes waiting feel dynamic |
| 4 | Question appears with chunky arcade-style option buttons | "Green or yellow — tap!" | Focused | Large thumb-friendly buttons; colour-coded answer options; clear hierarchy |
| 5 | Taps answer; immediate feedback (right: green flash / wrong: red flash) | "Yes! Got it!" or "Argh!" | Emotionally engaged | Instant answer feedback with satisfying animation; score delta shown |
| 6 | Leaderboard animates; their position highlighted | "I moved up three places!" | Invested | Personalised leaderboard position highlighted; movement celebrated |

---

## Value Proposition

**Structured Format:**

- **For** trivia night hosts and social quiz players
- **Who** want the high-energy fun of a party game without paying for closed platforms
- **The** PartyQuiz redesign
- **Is a** visual and interaction overhaul of an open-source quiz platform
- **That** delivers a Jackbox-level party feel with Kahoot-level accessibility — fully self-hostable and customisable
- **Unlike** Kahoot (educational, clinical, no self-host) or Jackbox (paid, closed-source, no custom questions)
- **Our approach** combines a bold "High-Voltage Arcade" design system, mobile-first player UX, cinematic live game flow, and an open platform that anyone can host

**Narrative Version:**

> PartyQuiz sits at the intersection of two experiences people already love: the slick accessibility of Kahoot and the party-room energy of Jackbox. The redesign isn't cosmetic — it establishes a design system capable of carrying every future feature, from new question types to gamification, without ever needing to ask "does this look right?" The result is a platform that hosts reach for because it makes *them* look good, and players enjoy because it makes even waiting for a question feel like part of the game.

---

## Competitive Landscape

| Alternative | Type | Strengths | Weaknesses | Relevance |
|-------------|------|-----------|------------|-----------|
| Kahoot | Direct Competitor | Market-leading; great accessibility; clean mobile UX; known brand | Educational/corporate aesthetic; no self-host; proprietary; expensive at scale | High — the dominant reference point; PartyQuiz must close the polish gap |
| Jackbox Games | Indirect Competitor | Legendary party-game feel; host + audience split; no phone install needed | Paid; closed-source; no custom question creation; requires game console/Steam | High — the gold standard for party energy; the aesthetic PartyQuiz is targeting |
| Quizwow | Direct Competitor | Fresher visual style than Kahoot | Niche audience; limited feature depth; less accessible | Medium |
| Mentimeter | Indirect Substitute | Strong for presentations and live polling | Not a quiz game; clinical UX; no game-show feel | Low |
| Google Forms / Slides | DIY Workaround | Free; familiar | Zero game energy; no real-time scoring | Low |

**Key Insight:** The gap PartyQuiz is uniquely positioned to own is **open + party-grade** — neither Kahoot (closed/educational) nor Jackbox (closed/non-customizable) serves this combination.

---

## Scope Recommendation

### Must Have (MVP)

| # | Capability | Validation Criterion Served | Rationale |
|---|-----------|---------------------------|-----------|
| 1 | "High-Voltage Arcade" design system — CSS token layer (`@theme` in TailwindCSS 4): palette (#0f1012, #FFD02F, #00E055, #2D8CFF, #FF4545), typography (Outfit, uppercase, weight-700/900), shadow system (hard offset), border radius | Criterion 4 (all new features use tokens) | Every other capability depends on this foundation existing first |
| 2 | Mobile-first player flow redesign: PIN entry, lobby wait screen, question/answer screen, leaderboard, results — all responsive and thumb-optimised | Criterion 1 (no more UI complaints), Criterion 2 (retention) | Players are on mobile; this is the most-seen UI in every game session |
| 3 | Live game question display redesign: animated countdown, question slide-in, answer option buttons (Bento/arcade style), live answer collection view | Criterion 1, Criterion 3 (positive demo reaction) | The core gameplay loop must feel like a game |
| 4 | Leaderboard redesign: animated position reveal, score delta display, winner highlight | Criterion 1, Criterion 3 | Crowd-facing moment — most emotionally impactful screen |
| 5 | PIN/lobby screen redesign: TV-optimised layout, player join animations, game code display | Criterion 3 (demo reaction) | First visual thing guests see; sets the tone for the entire session |
| 6 | Homepage / landing page redesign: reflects the new visual identity | Criterion 3 | Determines first impression for new users evaluating the platform |
| 7 | Navigation shell, global layout, and dark mode base redesign | Criterion 4 | Shell wraps all other pages — must be consistent before per-page work |

### Should Have

| # | Capability | Rationale |
|---|-----------|-----------|
| 1 | Full quiz creation editor redesign (shell, toolbar, question card layout) | Core workflow for Quiz Creators (Persona 3); affects long-term retention |
| 2 | Gamification animations: answer correct/wrong feedback flash, score counter animation, streak indicator | Significantly increases engagement feel; validates the "gamification" ask |
| 3 | Dashboard redesign (quiz list, stats cards) | Bento grid layout directly inspired by the design system; manageable alongside shell |
| 4 | Results / post-game summary page redesign | Final impression; affects whether the host returns or shares |

### Could Have

| # | Capability | Rationale |
|---|-----------|-----------|
| 1 | New question type UI components (visual specs and component shells for future types) | Design foundation can be laid without full backend implementation |
| 2 | Cinematic winner reveal / confetti / podium screen | High delight but not blocking for core validation |
| 3 | Shareable results card design (e.g., screenshot-formatted summary) | Supports word-of-mouth growth but is secondary |

### Won't Have (This Redesign Phase)

| # | Capability | Reason for Exclusion |
|---|-----------|---------------------|
| 1 | Backend changes (new APIs, database schema changes) | Out of scope — app is functionally sound |
| 2 | Native mobile app (iOS/Android) | Web-responsive satisfies the mobile use case; native is a future platform decision |
| 3 | New question type backend implementation | New question types need backend + socket server changes; separated into a future feature phase |
| 4 | New gamification mechanics (XP, levels, badges, persistent scoring) | Requires backend data model changes; animation-level gamification is sufficient for this phase |
| 5 | Full Svelte 4→5 Runes migration completion | Migration is in-flight but separate; redesign should be Runes-compatible but not gate on migration completing |

### Constraints and Boundaries

- The design system MUST be implemented as TailwindCSS 4 `@theme` CSS custom properties — no `.tailwind.config.js` (v4 is config-file-free)
- All components MUST use SvelteKit patterns; no introduction of third-party UI component libraries (e.g., shadcn, Flowbite) without explicit approval
- The player flow (PIN entry → game → results) MUST be fully functional on a 375px-wide mobile screen at all times
- The host/TV view MUST be designed for minimum 1280px display width
- All new/modified files MUST carry SPDX MPL-2.0 headers
- The redesign MUST NOT break any of the five critical user paths: create game, start game, join game, play game, view results

---

## Ambiguity Coverage Summary

| Category | Status | Resolution |
|----------|--------|------------|
| Functional Scope & Behaviour | Clear | Must Have + Should Have list; 5 critical paths identified |
| Domain & Data Model | Clear | No data model changes; visual layer only |
| Interaction & UX Flow | Partial | Key flows identified; exact component interactions deferred to PRD |
| Non-Functional Quality Attributes | Partial | Mobile 375px + TV 1280px targets set; WCAG AA still needs colour audit `[NEEDS CLARIFICATION: WCAG AA audit of proposed palette]` |
| Integration & External Dependencies | Clear | No new integrations; existing backend unchanged |
| Edge Cases & Failure Handling | Deferred | Low impact at design phase; architecture will address |
| Terminology | Clear | Design system terminology established in challenger brief |

---

## Open Questions

### Resolved (from Phase 0)
- *Is this purely CSS?* → No. UX structure changes are in scope for leaderboard, question/answer screens, and quiz creation.
- *What is the root problem?* → Visual identity mismatch with party/commercial positioning.
- *What motivates the timeline?* → New features are coming and the design foundation must be right before they land.

### New Questions (for Phase 2 / Architect)
- Should the design token system be implemented as a separate internal Svelte library/package, or as plain CSS in `src/app.css`?
- Should the Svelte 4→5 migration be completed in a coordinated sequence with the redesign (i.e., finish Runes migration per component as it's redesigned), or kept fully separate?
- Should the `'Outfit'` font be self-hosted (GDPR compliance for EU users) or loaded from Google Fonts?
- What is the WCAG AA verification result for the full proposed colour palette? `[NEEDS CLARIFICATION: run contrast checks before architecture phase]`

### Deferred
- Exact interaction spec for each gamification animation (timing, easing curves, triggers) — deferred to PRD/Architecture
- Whether the editor redesign should use a different rich-text editing approach (CKEditor5 is current; large dependency)

---

## Risks to the Product Concept

| Risk | Impact | Probability | Mitigation |
|------|--------|------------|------------|
| Svelte 4→5 migration and redesign create compound instability — each component touched twice at different times | High | Medium | Define a per-component sequencing strategy: complete Runes migration for a component before or alongside its visual redesign |
| "Gamification" scope grows during implementation beyond what the design system can cleanly support | Medium | High | Define gamification scope tightly in PRD as animation-only (no new backend mechanics) for this phase |
| WCAG AA failure on proposed colour palette forces late-stage colour changes | Medium | Medium | Run contrast audit immediately; establish approved token values before component work begins |
| New question types become a dependency on the design work | Medium | Medium | Design type-agnostic component shells; defer backend question type implementation to a separate feature phase |
| The Outfit font or specific design tokens conflict with existing Tailwind configuration | Low | Low | Tailwind 4 CSS-variable approach is highly composable; token conflicts are unlikely but should be tested early |

---

## Requirements Coverage Summary

| Section | Relevance | Coverage | Key Gaps |
|---------|-----------|----------|----------|
| 1 — Context, Goals | HIGH | 95% | — |
| 2 — System Inventory | HIGH | 90% | (Scout covered fully) |
| 3 — Pain Points | HIGH | 90% | Animation specifics deferred |
| 4 — Functional Reqs | HIGH | 70% | Per-screen component specs deferred to PRD |
| 5 — NFRs | MEDIUM | 60% | WCAG audit pending; no perf targets set |
| 6 — Data & Integration | LOW | 90% | No new integrations; token approach TBD |
| 7 — Compatibility | MEDIUM | 70% | Mobile 375px + TV 1280px; browser matrix TBD |
| 8 — Users & UX | HIGH | 85% | Detailed interaction specs deferred to PRD |
| 9 — Governance & Risk | MEDIUM | 70% | SPDX compliance required; no new reg. risk |
| 10 — Releases | MEDIUM | 60% | Sequencing TBD by Architect |
| 11 — Tech Architecture | HIGH | 75% | Token strategy decision deferred to Architect |
| 12 — Cost & Budget | LOW | N/A | Open-source, no third-party cost expected |
| 13 — Team & Staffing | LOW | N/A | Solo builder |
| 14 — Documentation | MEDIUM | 50% | Design system docs needed as deliverable |
| 15 — AI Components | LOW | N/A | No AI features in scope |
| 16 — Compliance | MEDIUM | 60% | WCAG AA pending; GDPR font hosting question |
| 17 — Observability | LOW | 70% | Sentry already integrated; no new req |
| 18 — Vendors | LOW | 90% | No new vendors; Google Fonts question pending |

---

## Insights Reference

**Companion Document:** [specs/insights/product-brief-insights.md](insights/product-brief-insights.md)

Key insights that shaped this document:

1. **This is a platform overhaul, not a restyle** — Confirmation of gamification, new question types, and full editor redesign means scope is substantially wider than Phase 0 implied.
2. **Players are on mobile; hosts are on desktop/TV** — This device split is a design architecture decision; two distinct layouts must be first-class, not afterthoughts.
3. **The open + party-grade combination is PartyQuiz's unique market position** — Neither Kahoot nor Jackbox owns this space; the redesign is the product's commercial differentiation.

See the insights document for complete decision rationale, alternatives considered, and questions explored.

---

## Phase Gate Approval

- [x] Human has reviewed this brief
- [x] At least one user persona is defined
- [x] User journeys are mapped
- [x] MVP scope is populated
- [x] Every Must Have capability traces to a Phase 0 validation criterion
- [x] All open questions are resolved or explicitly deferred with rationale
- [x] Human has explicitly approved this brief for Phase 2 handoff

**Approved by:** Jo Otey
**Approval date:** 2026-02-24
**Status:** Approved

---

## Linked Data

```json-ld
{
  "@context": { "js": "https://jumpstart.dev/schema/" },
  "@type": "js:SpecArtifact",
  "@id": "js:product-brief",
  "js:phase": 1,
  "js:agent": "Analyst",
  "js:status": "Draft",
  "js:version": "1.0.0",
  "js:upstream": [
    { "@id": "js:challenger-brief" },
    { "@id": "js:codebase-context" }
  ],
  "js:downstream": [
    { "@id": "js:prd" }
  ],
  "js:traces": []
}
```
