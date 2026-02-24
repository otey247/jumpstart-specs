---
id: prd
phase: 2
agent: PM
status: Approved
created: "2026-02-24"
updated: "2026-02-24"
version: "1.0.0"
approved_by: "Jo Otey"
approval_date: "2026-02-24"
upstream_refs:
  - specs/challenger-brief.md
  - specs/product-brief.md
dependencies:
  - challenger-brief
  - product-brief
risk_level: high
owners:
  - Jo Otey
sha256: null
---

# Product Requirements Document (PRD)

> **Phase:** 2 — Planning
> **Agent:** The Product Manager
> **Status:** Approved
> **Created:** 2026-02-24
> **Approval date:** 2026-02-24
> **Approved by:** Jo Otey
> **Upstream References:**
> - [specs/challenger-brief.md](challenger-brief.md)
> - [specs/product-brief.md](product-brief.md)

---

## Product Overview

PartyQuiz is a brownfield open-source quiz platform forked from ClassQuiz (MPL-2.0) whose current visual identity does not match its social and party positioning. Users dismiss the platform before experiencing its functionality, blocking adoption and commercial credibility. This PRD specifies the requirements for a comprehensive "High-Voltage Arcade" visual and structural redesign of the SvelteKit frontend, plus the addition of LLM-judged open-ended questions as a new game mechanic.

The redesign is built on a library-first TailwindCSS 4 design token system, a mobile-first player experience designed for Sam (guests on smartphones), and a TV-optimised host experience designed for Alex (trivia night organisers). A full quiz editor overhaul serves Morgan (quiz creators), and a new E8 epic adds real-time LLM judging via LiteLLM for open-ended question types — a scope expansion over the original Product Brief that was explicitly confirmed during planning.

The five critical existing user paths (create, start, join, play, view results) must not regress. Every Svelte component touched is migrated from Svelte 4 stores to Svelte 5 Runes in the same pass.

---

## Scope Expansion Note

> **⚠️ SCOPE DELTA vs. Product Brief:** Epic E8 (LLM-Judged Open-Ended Questions) introduces backend changes that the Product Brief placed in Won't Have for this phase. This addition was explicitly confirmed by the approver during Phase 2 planning (Round 3 elicitation). The Won't Have boundary in the Product Brief is superseded for this specific capability.

---

## Epics

### Epic E1: Design System Foundation

**Description:** Establish the library-first CSS design token layer, typography system, and base component primitives that all subsequent epics depend on. This delivers the "High-Voltage Arcade" design language as reusable, tokenised building blocks.

**Primary Persona:** All personas (foundational infrastructure)
**Scope Tier:** Must Have
**Validation Criterion Served:** Criterion 4 — all new features can be built using design tokens without requiring custom per-page CSS

---

#### Story E1-S1: WCAG AA colour contrast audit

**User Story:**

> As a developer beginning the redesign,
> I want a verified WCAG AA contrast audit of the proposed colour palette,
> so that no accessibility failures are introduced into the design token system.

**Acceptance Criteria:**

```gherkin
Given the proposed palette (#FFD02F, #00E055, #2D8CFF, #FF4545 on background #0f1012),
When an automated WCAG AA contrast audit is run using axe-core, Colour Contrast Analyser, or equivalent,
Then each colour achieves a minimum 4.5:1 ratio for normal text usage.

Given a colour is intended for large headings (≥18pt or ≥14pt bold),
When evaluated against the background,
Then it achieves a minimum 3:1 ratio for large text.

Given any colour fails the AA threshold,
When the audit result is documented,
Then an adjusted hex value that passes is recorded in the audit report before any token work begins.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | XS |
| **Dependencies** | None — this is the first task (Stage 1 gate) |

**Notes:** This story is a blocking gate. No E1-S2 or any component story may begin until this audit is complete and all tokens are confirmed as accessible.

---

#### Story E1-S2: CSS design token system

**User Story:**

> As a frontend developer,
> I want a centralised CSS custom-property token file (`@theme` in TailwindCSS 4),
> so that the full High-Voltage Arcade palette, typography scale, and shadow system are available as typed variables across every component.

**Acceptance Criteria:**

```gherkin
Given the SvelteKit app loads in a browser,
When any page renders,
Then the following CSS custom properties are present on :root: --color-bg, --color-surface, --color-accent-yellow, --color-accent-green, --color-accent-blue, --color-accent-red, --color-text-primary, --color-text-muted, --shadow-hard, --shadow-hard-sm, --radius-base, --font-display.

Given a developer needs the yellow accent colour in a new component,
When they write CSS or a Tailwind utility class,
Then they use the token (e.g., bg-[--color-accent-yellow] or var(--color-accent-yellow)) rather than the hardcoded hex value.

Given the token file is updated (e.g., a colour is adjusted after the WCAG audit),
When the app reloads,
Then all components using those tokens automatically reflect the change without requiring individual file edits.

Given a developer looks at the token file,
When tokens are organized by category,
Then colour tokens, typography tokens, spacing tokens, and shadow tokens each have a clear grouping comment.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | M |
| **Dependencies** | E1-S1 (audit must pass before tokens are locked) |

---

#### Story E1-S3: Outfit font self-hosting

**User Story:**

> As a developer configuring the typography system,
> I want the Outfit font loaded from a self-hosted npm package (`@fontsource/outfit`),
> so that no user data is sent to Google's servers and the font renders everywhere including offline dev.

**Acceptance Criteria:**

```gherkin
Given the app loads in a browser with external network requests blocked,
When the page renders,
Then text renders in the Outfit typeface (not a fallback system font).

Given the app is running,
When the browser's DevTools Network tab is inspected,
Then no request is made to fonts.googleapis.com or fonts.gstatic.com.

Given Outfit is configured,
When font weight 400, 700, and 900 are applied to text elements,
Then each weight renders visually distinctly and correctly.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | XS |
| **Dependencies** | E1-S2 (token system must reference the font family via --font-display) |

---

#### Story E1-S4: Button component variants

**User Story:**

> As a developer building any interactive screen,
> I want a reusable Button component with primary, secondary, ghost, and danger variants,
> so that all call-to-action elements in the product share consistent arcade-style styling.

**Acceptance Criteria:**

```gherkin
Given a developer uses <Button variant="primary">,
When the component renders,
Then it displays with the --color-accent-yellow background, black text, uppercase Outfit font (weight 700), and a hard 4px offset box shadow.

Given Button components with "primary", "secondary", "ghost", and "danger" variants are rendered side by side,
When the page is viewed,
Then each variant is visually distinct from one another.

Given a Button is hovered by a pointer device,
When the cursor enters the element,
Then a visual state change (e.g., shadow offset shift or brightness adjustment) occurs via CSS transition.

Given a Button has the `disabled` attribute set,
When the button renders,
Then it is visually dimmed (reduced opacity) and does not trigger any action on click or tap.

Given a Button is focused via keyboard navigation,
When focus is applied,
Then a visible focus ring is shown that meets WCAG AA focus indicator requirements.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | M |
| **Dependencies** | E1-S2, E1-S3 |

---

#### Story E1-S5: Card / Bento container primitive

**User Story:**

> As a developer laying out any dashboard or feature screen,
> I want a reusable Card/Bento container component,
> so that content blocks share a consistent dark-surface, hard-border, neo-brutalist visual style.

**Acceptance Criteria:**

```gherkin
Given a Card component is rendered,
When displayed at any viewport width,
Then it has a dark background (--color-surface token), a solid 2px border, and a hard box shadow using --shadow-hard.

Given content is placed inside a Card,
When the viewport is resized from 375px to 1440px,
Then the card responsively adjusts its padding and layout without content overflow.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Should Have |
| **Size** | S |
| **Dependencies** | E1-S2, E1-S4 |

---

### Epic E2: Player Journey (Mobile)

**Description:** Redesign every screen in the player-facing game flow to be mobile-first, thumb-optimised, and visually energetic on a 375px smartphone. This is the most-seen UI in every game session and the primary driver of player satisfaction and word-of-mouth.

**Primary Persona:** Sam, the Social Player
**Scope Tier:** Must Have
**Validation Criterion Served:** Criterion 1 (no more UI complaints), Criterion 2 (player retention)

---

#### Story E2-S1: PIN entry screen

**User Story:**

> As Sam, a player joining a live game,
> I want a clear and fast PIN entry screen on mobile,
> so that I can join the game in two taps without confusion.

**Acceptance Criteria:**

```gherkin
Given a player navigates to /play or the game join URL,
When the screen loads at 375px viewport width,
Then a large PIN input field, a name input field, and a "Join Game" button are all visible above the fold without scrolling.

Given a player enters a valid game PIN and display name and presses Join,
When the server responds with a successful join event,
Then the player's screen transitions to the lobby wait screen (E2-S2) without any additional user input.

Given a player enters an invalid or expired PIN and presses Join,
When the server responds with an error,
Then an inline error message is displayed below the PIN input (e.g., "Game not found") and the PIN input is cleared for re-entry.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | S |
| **Dependencies** | E1-S2, E1-S4 (token layer and button component) |

---

#### Story E2-S2: Lobby wait screen

**User Story:**

> As Sam, waiting for the host to start the game,
> I want a lobby screen that confirms I've joined and shows the game starting,
> so that I feel engaged rather than staring at a blank waiting page.

**Acceptance Criteria:**

```gherkin
Given a player has successfully joined,
When the lobby screen renders at 375px,
Then their chosen display name is prominently shown and a "Waiting for host…" animated indicator is visible.

Given the host has not started the game,
When the player has been on the lobby screen for more than 30 seconds,
Then the waiting animation remains active and no error or timeout message is shown.

Given the host starts the game,
When the game-start socket event is received,
Then the player's screen transitions to the question screen (E2-S3) immediately without requiring any user interaction.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | S |
| **Dependencies** | E2-S1 |

---

#### Story E2-S3: Question and answer screen

**User Story:**

> As Sam, answering a question during the game,
> I want large, clearly-coloured answer buttons on my phone screen,
> so that I can tap my answer quickly and confidently in a noisy, social setting.

**Acceptance Criteria:**

```gherkin
Given a multiple-choice question is active,
When the question screen renders at 375px,
Then the question text and all answer option buttons are visible without horizontal scrolling or vertical overflow beyond one scroll.

Given a multiple-choice question is displayed,
When the answer option buttons render,
Then each option is a tap target of at least 44px in height and spans the full available width.

Given a player taps one of the answer options,
When the tap registers,
Then the selected option is immediately visually highlighted (e.g., border glow or colour fill) and the player cannot select a different option afterwards.

Given the question timer expires before the player has answered,
When the timer reaches zero,
Then the player's screen shows a "Time's up" state and any partial selection (or no answer) is automatically submitted.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | M |
| **Dependencies** | E2-S2, E1-S4 |

---

#### Story E2-S4: Player leaderboard screen

**User Story:**

> As Sam, between questions,
> I want to see my current rank and score on my phone,
> so that I know how I'm doing and feel motivated to keep playing.

**Acceptance Criteria:**

```gherkin
Given a round ends and the leaderboard phase begins,
When the player leaderboard screen loads at 375px,
Then the player's current rank (e.g., "3rd place") and total score are prominently displayed.

Given a player's rank improved from the previous round,
When the leaderboard is shown,
Then a rank improvement indicator (e.g., upward arrow and delta value) is displayed next to their position.

Given a player is ranked 1st,
When the leaderboard shows their position,
Then their row is visually highlighted differently from other positions (e.g., special colour treatment or icon).
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | S |
| **Dependencies** | E2-S3 |

---

#### Story E2-S5: Player results screen

**User Story:**

> As Sam, at the end of the game,
> I want to see my final score and rank on my phone,
> so that I can celebrate (or commiserate) and share the result.

**Acceptance Criteria:**

```gherkin
Given the game ends,
When the player results screen loads at 375px,
Then the player's final rank, total score, and number of correct answers are all visible.

Given the results screen is displayed,
When rendered at 375px width,
Then all three data points (rank, score, correct answers) are visible above the fold without scrolling.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | S |
| **Dependencies** | E2-S4 |

---

#### Story E2-S6: Player error states (invalid PIN, disconnect)

**User Story:**

> As Sam, experiencing a network issue or entering the wrong PIN,
> I want clear error messages and automatic reconnection,
> so that I don't have to start over or bother the host when something goes wrong.

**Acceptance Criteria:**

```gherkin
Given a player enters a PIN that doesn't match any active game,
When the error response is received,
Then an error message reading "Game not found — check your code" appears inline below the PIN input without a page redirect.

Given a player loses network connectivity during gameplay,
When the WebSocket connection drops,
Then a non-blocking reconnection indicator is shown on the player's screen and reconnection is attempted automatically.

Given a reconnection attempt succeeds,
When the socket reconnects within the allowed window,
Then the player is returned to their current game state without requiring them to re-enter their PIN.

Given reconnection has failed after 5 attempts,
When the connection cannot be restored,
Then a full-screen error state is shown with a "Return to join screen" CTA.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | S |
| **Dependencies** | E2-S1 |

---

### Epic E3: Host & Live Game Screens (TV)

**Description:** Redesign every screen in the host-facing live game flow to be TV-optimised, visually commanding, and dramatically satisfying for a room full of players watching together. The host controls the game from a laptop/desktop; the display is often projected or mirrored to a large screen.

**Primary Persona:** Alex, the Trivia Night Host
**Scope Tier:** Must Have
**Validation Criterion Served:** Criterion 3 (positive first-impression reaction without prompting)

---

#### Story E3-S1: TV PIN/lobby screen

**User Story:**

> As Alex, starting a game with a room full of guests,
> I want a lobby screen that displays the game PIN in huge text and shows players joining in real time,
> so that guests can join by glancing at the screen and I can see everyone is in before starting.

**Acceptance Criteria:**

```gherkin
Given a host has created and started a game session,
When the lobby screen renders at 1280px width,
Then the game PIN is displayed in a font size of at least 96px and is clearly legible from 3 metres.

Given a new player joins the game via their phone,
When the join event is received,
Then the player's display name appears in the player list on the host screen within 1 second of joining.

Given 20 players have joined,
When the player list renders,
Then all 20 player names are visible without clipping (scrollable grid or wrapping layout as needed).
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | M |
| **Dependencies** | E1-S2, E1-S4 |

---

#### Story E3-S2: Live question display with countdown

**User Story:**

> As Alex, running a live question,
> I want the question displayed on the TV screen with a dramatic countdown timer,
> so that the room feels the tension of answering under pressure.

**Acceptance Criteria:**

```gherkin
Given a question is broadcast to the live game,
When the host question screen updates,
Then the question text and answer option labels (without correct-answer indicators) are visible at 1280px width.

Given a question has a time limit configured,
When the countdown starts,
Then a visual timer animates in real time and the remaining time is legible from 2 metres at 1280px.

Given the timer reaches zero,
When time expires,
Then the countdown visually signals expiry (e.g., colour change or flash) and the host screen automatically transitions to the answer reveal state.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | M |
| **Dependencies** | E3-S1 |

---

#### Story E3-S3: Live answer tally and reveal

**User Story:**

> As Alex, watching answers come in,
> I want to see a live count of responses and a dramatic correct-answer reveal,
> so that I can build tension in the room and celebrate the reveal moment.

**Acceptance Criteria:**

```gherkin
Given a question is active and players are submitting answers,
When answer events arrive at the host screen,
Then an incrementing response count is displayed in real time (e.g., "14 / 20 answered").

Given all players have answered or the timer has expired,
When the answer reveal is triggered,
Then the correct answer option is highlighted (e.g., green fill) and incorrect options are visually dimmed.

Given the reveal has fired,
When the answer statistics are displayed,
Then the percentage or count of players who selected each option is visible alongside each option.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | M |
| **Dependencies** | E3-S2 |

---

#### Story E3-S4: Host leaderboard

**User Story:**

> As Alex, between questions,
> I want a leaderboard showing the top players on the TV screen,
> so that the crowd can cheer for leaders and feel dramatic tension between rounds.

**Acceptance Criteria:**

```gherkin
Given a round ends and the leaderboard phase begins,
When the host leaderboard screen renders at 1280px,
Then the top 5 players (or all players if fewer than 5) are displayed with rank number, display name, and current score.

Given a player's rank changed from the previous round,
When the leaderboard is displayed,
Then an indicator (arrow up / arrow down) shows their rank movement alongside their row.

Given the leaderboard is rendered on a TV display,
When text sizes are evaluated at 1280px,
Then player names and scores use a minimum 32px font size, ensuring legibility at 2 metres.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | M |
| **Dependencies** | E3-S3 |

---

#### Story E3-S5: Winner / final results screen (host)

**User Story:**

> As Alex, at the end of the game,
> I want a cinematic winner reveal on the TV screen,
> so that the final moment feels climactic and the winning player gets their deserved glory.

**Acceptance Criteria:**

```gherkin
Given the game ends,
When the final results screen loads at 1280px,
Then the winning player's name and score are displayed with a visually prominent treatment (large font, accent colour, distinct layout from intermediate leaderboards).

Given multiple players are tied for first place,
When the final screen renders,
Then all tied players are displayed as co-winners with equal prominence.

Given the final results screen is shown,
When compared visually against intermediate leaderboard screens,
Then the final screen is cinematically distinct (e.g., different layout, special animation trigger point) to mark the game's end.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | S |
| **Dependencies** | E3-S4 |

---

### Epic E4: Shell, Navigation & Homepage

**Description:** Establish the dark base layout, global navigation component, and redesigned homepage that form the visual shell every user sees before and during a game session. This is the public face of the redesign.

**Primary Persona:** All personas
**Scope Tier:** Must Have
**Validation Criterion Served:** Criterion 3 (positive first-impression reaction), Criterion 4

---

#### Story E4-S1: Dark base layout and global CSS

**User Story:**

> As any user visiting PartyQuiz,
> I want every page to render with the High-Voltage Arcade dark base (#0f1012 background, Outfit font),
> so that the brand identity is immediately consistent across the entire application.

**Acceptance Criteria:**

```gherkin
Given any page in the application loads,
When the HTML and body render,
Then the background colour resolves to the --color-bg token (#0f1012 or audited equivalent).

Given any page body text renders,
When the font stack is applied,
Then Outfit is the primary display font and --color-text-primary is the default text colour.

Given the app is opened in Chrome and Firefox (latest 2 versions each),
When the global layout renders,
Then no visual regressions are observable in the dark base across both browsers.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | S |
| **Dependencies** | E1-S2, E1-S3 |

---

#### Story E4-S2: Primary navigation component

**User Story:**

> As any logged-in user navigating PartyQuiz,
> I want a consistent navigation bar with the brand wordmark, primary links, and my account controls,
> so that I always know where I am and can reach any main section in one click.

**Acceptance Criteria:**

```gherkin
Given a logged-in user views any page,
When the navigation component renders,
Then the PartyQuiz logo/wordmark, primary navigation links (e.g., Dashboard, Explore), and a user avatar/account menu are present.

Given a user is on a 375px mobile viewport,
When the navigation renders,
Then it collapses to a compact pattern (hamburger menu or bottom navigation bar) with no horizontal overflow.

Given a user is not logged in,
When the navigation renders,
Then "Sign in" and "Create account" CTAs are visible in place of the authenticated user avatar.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | M |
| **Dependencies** | E4-S1, E1-S4 |

---

#### Story E4-S3: Homepage redesign

**User Story:**

> As an unauthenticated visitor evaluating PartyQuiz,
> I want a homepage that immediately communicates the platform's party energy and invites me to try it,
> so that I understand what PartyQuiz is and want to start a game within 30 seconds.

**Acceptance Criteria:**

```gherkin
Given an unauthenticated user visits the root URL (/),
When the page loads at 1280px viewport width,
Then a hero section with a bold headline, sub-headline, and a primary CTA button ("Host a Game" or equivalent) is visible above the fold.

Given the homepage loads at 375px viewport width,
When rendered on mobile,
Then all hero content is readable and the primary CTA is visible above the fold without horizontal scroll.

Given the homepage renders at any viewport width,
When evaluated against the design system,
Then the High-Voltage Arcade palette tokens, Outfit font, hard box shadows, and bento grid content layout are all present.

Given an authenticated user visits the homepage,
When the page loads,
Then it surfaces their recent quizzes, a quick-start action, or redirects to their dashboard.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | L |
| **Dependencies** | E4-S1, E4-S2, E1-S5 |

---

### Epic E5: Quiz Editor Redesign

**Description:** Overhaul the quiz creation and editing experience with the new design system, delivering a professional, visually consistent editor that gives Morgan confidence that quizzes will look great at runtime.

**Primary Persona:** Morgan, the Quiz Creator
**Scope Tier:** Should Have

---

#### Story E5-S1: Editor shell and toolbar

**User Story:**

> As Morgan, building a quiz in the editor,
> I want an editor layout with a clearly organised question list, canvas area, and toolbar,
> so that I can navigate my quiz structure and access all editing tools without hunting for controls.

**Acceptance Criteria:**

```gherkin
Given a user navigates to the quiz editor (create or edit),
When the page loads at 1280px,
Then the layout presents a left panel (question list), a central canvas area, and a top toolbar — each visually delineated.

Given a user is in the editor,
When the toolbar renders,
Then Add Question, Save Quiz, Preview, and Settings actions are accessible as labelled controls.

Given a user clicks "Add Question",
When the action is triggered,
Then a new empty question card appears in the canvas area and the question list panel updates to include the new entry.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Should Have |
| **Size** | L |
| **Dependencies** | E1-S2, E1-S4, E1-S5 |

---

#### Story E5-S2: Multiple choice question card

**User Story:**

> As Morgan, editing a multiple-choice question,
> I want a visually clear question card with editable text and answer inputs,
> so that I can author questions efficiently and clearly mark the correct answer.

**Acceptance Criteria:**

```gherkin
Given a multiple-choice question card is in edit mode,
When the card renders in the canvas,
Then a question text input and four answer option inputs are present and editable.

Given an author clicks the correct-answer indicator on one of the four options,
When the option is toggled,
Then it is visually distinguished from the other options (e.g., green border or checkmark icon).

Given an author has filled in question text and all four option fields and navigates away from the card,
When the save/blur event fires,
Then the changes are persisted and the question list panel shows a summary of the updated question.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Should Have |
| **Size** | M |
| **Dependencies** | E5-S1 |

---

#### Story E5-S3: Quiz metadata and settings form

**User Story:**

> As Morgan, configuring a new quiz,
> I want a settings form with title, description, and visibility controls,
> so that I can manage my quiz's identity and sharing settings without navigating away from the editor.

**Acceptance Criteria:**

```gherkin
Given a user opens quiz settings (via toolbar Settings action),
When the form renders,
Then fields for quiz title (required), description (optional), and visibility toggle (public / private) are present.

Given a user enters a valid title and saves the settings form,
When the save completes successfully,
Then the updated title is immediately reflected in the editor toolbar and in the quiz list panel.

Given a user attempts to save the settings form with an empty title,
When validation runs,
Then an inline error message ("Title is required") prevents form submission.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Should Have |
| **Size** | M |
| **Dependencies** | E5-S1 |

---

#### Story E5-S4: Additional question type visual shells

**User Story:**

> As Morgan, exploring new question types in the editor,
> I want to see visual shell cards for upcoming question types,
> so that I can understand what's coming and the design system is ready for them.

**Acceptance Criteria:**

```gherkin
Given a user opens the "Add Question" menu,
When additional question types beyond multiple-choice are listed,
Then each type has a distinct visual shell card (e.g., "Open Text", "True/False", "Slider") that renders in the canvas when selected.

Given a visual shell for an unimplemented question type is rendered,
When a user views it in the editor,
Then it is clearly labelled as "Coming soon" or "Not yet available" and does not allow content to be saved as a functional question.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Could Have |
| **Size** | M |
| **Dependencies** | E5-S1, E5-S2 |

---

### Epic E6: Gamification & Feedback Animations

**Description:** Layer animation-level gamification — answer feedback flashes, score counter movements, and leaderboard rank animations — on top of the redesigned screens to elevate the emotional energy of answering and competing.

**Primary Persona:** Sam (player feedback), Alex (crowd reaction)
**Scope Tier:** Should Have

---

#### Story E6-S1: Answer correct/wrong feedback flash

**User Story:**

> As Sam, seeing whether my answer was right,
> I want an immediate, satisfying visual flash when the correct answer is revealed,
> so that I feel the emotional hit of being right or wrong in the moment.

**Acceptance Criteria:**

```gherkin
Given a player has submitted an answer and the host triggers the reveal,
When the answer is correct,
Then the player's screen displays a green flash animation and a positive indicator within 200ms of the reveal socket event.

Given a player has submitted an answer and the host triggers the reveal,
When the answer is incorrect,
Then the player's screen displays a red or dark desaturated flash and shows the correct answer highlighted.

Given the answer feedback animation plays,
When the OS or browser has prefers-reduced-motion: reduce active,
Then no flashing animation occurs — only an immediate static state change to the correct/incorrect visual.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Should Have |
| **Size** | S |
| **Dependencies** | E2-S3, E1-S2 |

---

#### Story E6-S2: Score counter animation

**User Story:**

> As Sam, watching my score after a correct answer,
> I want my score to count up visually,
> so that each point gained feels earned and satisfying.

**Acceptance Criteria:**

```gherkin
Given a player's score increases after a correct answer,
When the score counter is displayed post-reveal,
Then the score number increments visually from the previous value to the new value over no more than 500ms.

Given the score animation plays,
When the OS has prefers-reduced-motion: reduce active,
Then the score updates to the new value immediately without an animated increment.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Should Have |
| **Size** | S |
| **Dependencies** | E6-S1 |

---

#### Story E6-S3: Leaderboard rank change animation

**User Story:**

> As a player and the watching crowd,
> I want to see leaderboard positions animate into place,
> so that rank movements create a dramatic visual moment between rounds.

**Acceptance Criteria:**

```gherkin
Given the leaderboard is displayed after a round,
When a player's rank has changed from the previous round,
Then their row animates into its new vertical position (slide or cross-fade transition) rather than jumping instantaneously.

Given the leaderboard rank animation plays,
When the OS has prefers-reduced-motion: reduce active,
Then positions update to their new values instantly without movement animation.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Should Have |
| **Size** | M |
| **Dependencies** | E3-S4, E2-S4, E1-S2 |

---

#### Story E6-S4: `prefers-reduced-motion` compliance

**User Story:**

> As a developer building animated components,
> I want a consistent pattern for respecting `prefers-reduced-motion: reduce`,
> so that all animations in the system are accessible by default.

**Acceptance Criteria:**

```gherkin
Given any animated component in the codebase,
When the OS or browser has prefers-reduced-motion: reduce active,
Then CSS animations and transitions in that component are disabled or replaced with instant state changes.

Given a developer creates a new animated component,
When they write the animation CSS,
Then they use the @media (prefers-reduced-motion: reduce) query to override motion with static transitions.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | XS |
| **Dependencies** | E1-S2 |

**Notes:** E6-S4 is Must Have because it ties back to the WCAG AA accessibility standard. It is listed in E6 for logical grouping but should be implemented as part of E1 foundational work (token-level CSS variable or mixin that all animated components consume).

---

### Epic E7: Dashboard & Supporting Screens

**Description:** Redesign the quiz library dashboard, post-game results summary, and user profile/settings pages using the bento grid layout and new component library, completing the full-platform design consistency.

**Primary Persona:** Alex (results), Morgan (dashboard), Sam (results)
**Scope Tier:** Should Have

---

#### Story E7-S1: Quiz dashboard / library page

**User Story:**

> As Morgan, managing my quiz collection,
> I want a dashboard that shows my quizzes as a bento grid of cards,
> so that I can easily scan, launch, or edit any quiz at a glance.

**Acceptance Criteria:**

```gherkin
Given a logged-in user navigates to /dashboard,
When the page loads,
Then their quiz library is displayed as a responsive bento grid of quiz cards.

Given a user has no quizzes yet,
When the dashboard loads,
Then an empty state with a "Create your first quiz" CTA is displayed instead of an empty grid.

Given a quiz card is rendered in the grid,
When displayed,
Then it shows the quiz title, question count, and last-modified date.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Should Have |
| **Size** | M |
| **Dependencies** | E4-S1, E1-S5 |

---

#### Story E7-S2: Post-game results summary page

**User Story:**

> As Alex, reviewing game performance after it ends,
> I want a results summary page with all player scores,
> so that I can see how the game went and share highlights.

**Acceptance Criteria:**

```gherkin
Given a game has ended,
When the host navigates to the results summary page,
Then a ranked list of all players with their final scores and correct-answer counts is displayed.

Given the results summary page loads at 1280px,
When the layout is rendered,
Then it uses the bento-grid layout consistent with the dashboard (E7-S1).

Given the results summary loads at 375px,
When the player list renders on mobile,
Then it is displayed as a readable single-column stacked list without overflow.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Should Have |
| **Size** | M |
| **Dependencies** | E4-S1, E1-S5, E7-S1 |

---

#### Story E7-S3: User profile / account settings

**User Story:**

> As any authenticated user,
> I want my profile and settings pages to match the new dark design system,
> so that the visual experience is consistent across the entire app.

**Acceptance Criteria:**

```gherkin
Given an authenticated user navigates to their profile or settings page,
When the page loads,
Then the global dark layout (E4-S1) and new component styles are applied consistently.

Given a user updates their display name and saves,
When the form submits successfully,
Then the navigation component (E4-S2) immediately reflects the updated name without a full page reload.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Should Have |
| **Size** | S |
| **Dependencies** | E4-S1, E4-S2 |

---

### Epic E8: LLM-Judged Open-Ended Questions

**Description:** Add a new "open-ended" question type where players type free-text answers, and the LiteLLM service judges correctness in real time during gameplay. This requires both backend changes (socket server, new question type, LiteLLM integration) and frontend components for the host and player experiences.

**Primary Persona:** Morgan (creates open-ended questions), Alex (hosts them), Sam (answers them)
**Scope Tier:** Must Have *(scope confirmed by approver during Phase 2 Round 3, overriding Product Brief Won't Have)*
**Validation Criterion Served:** Criterion 1 (broader question variety eliminates limitations complaints)

> **⚠️ Backend Changes:** This epic introduces backend modifications to `classquiz/socket_server/`, a new `classquiz/llm/` module, and database schema changes for the open-ended question type. These were explicitly approved in Phase 2 scope review.

---

#### Story E8-S1: Open-ended question type (backend)

**User Story:**

> As Morgan, creating a quiz with open-ended questions,
> I want an "open text" question type stored and transmitted through the live game flow,
> so that the game engine can present a text-input question to players.

**Acceptance Criteria:**

```gherkin
Given a quiz contains a question with type "open_ended",
When a live game reaches that question,
Then the socket server broadcasts a question event with type "open_ended" and no pre-defined answer options.

Given players submit free-text answers to an open-ended question,
When answer events arrive at the socket server,
Then each answer is stored and queued for LLM evaluation (E8-S2) rather than immediately scored.

Given the open-ended question type is selected in the editor,
When the question is saved,
Then the database stores it with the correct type discriminator distinguishing it from multiple-choice questions.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | L |
| **Dependencies** | E1-S2 (token layer for UI), existing socket_server architecture |

---

#### Story E8-S2: LiteLLM evaluation service

**User Story:**

> As the game system processing open-ended answers,
> I want a LiteLLM-backed evaluation service that judges free-text answers in real time,
> so that players receive a correct/incorrect verdict without manual host intervention.

**Acceptance Criteria:**

```gherkin
Given a player submits a text answer to an open-ended question,
When the LiteLLM evaluation service receives the answer text and question text,
Then it calls the configured LLM provider via LiteLLM and returns a verdict (correct / incorrect / partial) within 5 seconds.

Given the LLM returns a verdict,
When the evaluation completes,
Then the verdict is emitted as a socket event to the answering player (their result) and to the host screen (aggregate count).

Given the LLM provider is misconfigured, rate-limited, or unavailable,
When the evaluation call fails,
Then the system emits a fallback event triggering manual host review mode (E8-S5) rather than crashing or hanging the live game.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | L |
| **Dependencies** | E8-S1, LiteLLM Python package, configured LLM provider API key |

**Notes:** The LLM provider (OpenAI, Anthropic, local via Ollama, etc.) is an Architect-phase decision. The PRD requires only that LiteLLM is the abstraction layer. API key must be stored as an environment variable — never hardcoded.

---

#### Story E8-S3: Open-ended question — player view

**User Story:**

> As Sam, answering an open-ended question on my phone,
> I want a text input field where I can type my answer and submit it,
> so that I can participate in open-ended rounds with the same ease as multiple-choice.

**Acceptance Criteria:**

```gherkin
Given an open-ended question is active,
When the player's screen renders at 375px,
Then a text input field (with soft keyboard support on mobile) and a "Submit" button are visible above the fold.

Given a player types an answer and presses Submit,
When the submission is confirmed by the socket,
Then the text input is locked against further editing and a "Waiting for result…" animated state is shown.

Given the LLM verdict arrives,
When the socket delivers the result to the player,
Then the player's screen transitions to the correct/incorrect feedback state (reusing E6-S1 visuals).
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | M |
| **Dependencies** | E8-S1, E2-S3, E6-S1 |

---

#### Story E8-S4: Open-ended answer verdict display (host)

**User Story:**

> As Alex, hosting a round with open-ended questions,
> I want to see judgement verdicts arrive live on my screen,
> so that I know how the round is progressing and can advance when all players have results.

**Acceptance Criteria:**

```gherkin
Given players are submitting answers to an open-ended question,
When the host screen is active,
Then a running count of "Evaluated: N / Total: M" is displayed, updating as each verdict arrives.

Given all players have submitted and been evaluated (or the timer has expired),
When the verdict phase completes,
Then the host screen advances to the leaderboard / next-round flow with scores updated to reflect open-ended question results.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | M |
| **Dependencies** | E8-S2, E3-S3 |

---

#### Story E8-S5: LLM failure fallback (manual override)

**User Story:**

> As Alex, when the LLM service is unavailable during a live game,
> I want to be able to manually judge players' text answers,
> so that the game can continue even if the AI evaluation fails.

**Acceptance Criteria:**

```gherkin
Given the LiteLLM service fails to return a verdict,
When the fallback event is received by the host screen,
Then a manual review panel appears on the host screen listing each player's submitted text answer alongside "Correct" and "Incorrect" buttons.

Given the host manually marks an answer as correct,
When the manual verdict is submitted,
Then the player's score is updated as if the LLM had returned a "correct" verdict and the player receives the appropriate feedback.
```

| Attribute | Value |
|-----------|-------|
| **Priority** | Must Have |
| **Size** | M |
| **Dependencies** | E8-S2, E8-S4 |

---

## Non-Functional Requirements

### Performance

| NFR ID | Requirement | Threshold | Percentile | Verification Method | SLA Tier |
|--------|-------------|-----------|-----------|-------------------|----------|
| NFR-P01 | Player flow page transitions (mobile) | < 2s render on 3G | p95 | Lighthouse mobile simulation | Tier 1 |
| NFR-P02 | End-to-end open-ended verdict (submit → display) | < 5s | p95 | Socket event timing log in test game | Tier 1 |
| NFR-P03 | Design token propagation (CSS custom property changes) | Uniform across all components at reload | — | Visual regression test | Tier 2 |
| NFR-P04 | Animation frame rate for leaderboard and feedback effects | ≥ 60fps on mid-range mobile (2020 or newer) | — | DevTools Performance panel | Tier 2 |

### Throughput and Scalability

| NFR ID | Requirement | Target | Sustained Duration | Verification Method |
|--------|-------------|--------|-------------------|-------------------|
| NFR-T01 | Concurrent players per live game session | 50 players | Full game duration | Load test via simulate_players.py (existing) |
| NFR-T02 | LiteLLM concurrent evaluation queue | 50 sequential calls within timer window | Per question | Timing test in dev environment |

### Availability and Reliability

| NFR ID | Requirement | Target | Measurement Period | Verification Method |
|--------|-------------|--------|-------------------|-------------------|
| NFR-A01 | Five critical user paths pass CI after every PR | 100% pass rate | Per PR merge | Existing test suite + new regression tests |
| NFR-A02 | LLM service failure must not crash the live game | 100% — game continues in fallback mode | Per failure event | Integration test with mocked LLM failure |
| NFR-A03 | All error states return user-visible feedback | 100% of error paths have associated UI state | Per feature | Manual QA checklist |

### Security

| Requirement | Detail | Verification Method |
|-------------|--------|-------------------|
| Session and JWT cookie integrity | Redesign changes must not modify auth cookie handling or session management | Auth regression tests (test_auth.py) |
| LLM API key storage | API key for LiteLLM provider must be stored as environment variable in `.env` / Docker secrets | Code review — grep for hardcoded keys |
| No new public unauthenticated endpoints | E8 evaluation endpoint must require an active game session token | Integration test — call without session → expect 401/403 |

**Compliance requirements:** GDPR — player answers sent to LLM provider may contain personal data. Architecture phase must assess whether player display names or answer text are transmitted and which provider is selected. Prefer providers with data processing agreements (DPAs).

### Accessibility

| Requirement | Target | Verification Method |
|-------------|--------|-------------------|
| WCAG 2.1 AA colour contrast | All text/background combinations pass 4.5:1 (normal) or 3:1 (large) | axe-core automated scan + E1-S1 audit |
| Keyboard navigability | All interactive elements reachable and operable via keyboard | Manual keyboard-only test walkthrough |
| Focus indicators | All focusable elements have visible focus rings meeting WCAG 2.4.11 | Automated axe scan |
| Motion accessibility | All animations respect prefers-reduced-motion: reduce | E6-S4 implementation; CSS media query audit |
| Alternative text | All decorative and informational images have appropriate alt attributes | axe-core scan |
| Form labels | All form inputs have associated visible or screen-reader-accessible labels | axe-core scan |

**Definition of Done:** A story is "done" when: code is merged to main, all five critical paths pass CI, the screen passes a Lighthouse accessibility audit (score ≥ 90), and visual output has been manually verified against design intent.

### Observability

| Requirement | Detail |
|-------------|--------|
| Error monitoring | Sentry integration must remain connected through all file changes |
| LLM evaluation logging | Each LiteLLM call must log: question_id, player_id, latency_ms, verdict, error (if any) at INFO level |
| Socket event logging | Existing socket_server event logging must not be broken |

### Other

| Category | Requirement | Detail |
|----------|-------------|--------|
| Browser support | Chrome, Firefox, Safari (latest 2 versions each); Mobile Safari iOS 16+; Chrome Android latest | Player flow tested on Safari iOS (common party scenario) |
| SPDX compliance | All new and modified files carry SPDX MPL-2.0 header | Code review checklist |
| Svelte migration | Every component touched during redesign is simultaneously migrated from Svelte 4 stores to Svelte 5 Runes | Per-component migration checklist |

### Backward Compatibility (Brownfield)

| NFR ID | Requirement | Detail | Verification |
|--------|-------------|--------|-------------|
| NFR-BC01 | Create game path must not regress | POST /api/v1/quiz and editor save flow | test_server.py + manual walkthrough |
| NFR-BC02 | Start game path must not regress | Game session creation and lobby socket join | test_server.py |
| NFR-BC03 | Join game path must not regress | Player PIN entry → socket join → lobby | test_server.py |
| NFR-BC04 | Play game path must not regress | Full question/answer loop via socket | test_server.py |
| NFR-BC05 | View results path must not regress | Post-game results retrieval and display | test_server.py |
| NFR-BC06 | Authentication must not regress | Login, OAuth (Google/GitHub), WebAuthn, TOTP | test_auth.py |
| NFR-BC07 | Admin and moderation routes must not regress | Admin dashboard access control | Manual verification post-deploy |
| NFR-BC08 | Quiz creation and editing must not regress | CRUD operations on quizzes | test_server.py + manual editor walkthrough |

---

## Dependencies and Risks

### External Dependencies

| # | Dependency | Type | Impact if Unavailable | Mitigation |
|---|-----------|------|----------------------|------------|
| 1 | `@fontsource/outfit` npm package | Package | Outfit font unavailable; fallback to system font | Audit package health before starting E1-S3; pin to specific version |
| 2 | LiteLLM Python package | Package | E8 cannot be implemented | Pin to specific version; evaluate at Phase 3; confirm Python 3.13 compatibility |
| 3 | LLM provider API (OpenAI / Anthropic / other) | External Service | E8-S2 judgment calls fail | Architecture phase selects provider; E8-S5 fallback handles runtime failures |
| 4 | TailwindCSS 4 `@theme` config-free model | Framework | Token system architecture changes | Confirmed compatible with SvelteKit 2; test in isolation before E1-S2 |

### Risk Register

| # | Risk Description | Type | Impact | Probability | Mitigation | Owner |
|---|-----------------|------|--------|------------|------------|-------|
| 1 | Svelte 4→5 Runes interleaving introduces component regressions — migrating and redesigning simultaneously doubles the change surface per component | Technical | High | Medium | Per-component checklist: migrate Runes → apply design → test independently before merging. Never leave a component half-migrated. | Developer |
| 2 | TailwindCSS 4 `@theme` behaviour is new; some CSS-in-JS patterns or class utilities may behave unexpectedly without a config file | Technical | Medium | Medium | Prototype the token layer (E1-S2) early; validate all utility class patterns before using them in story stages | Developer |
| 3 | WCAG AA audit fails on one or more palette colours, requiring a colour adjustment that ripples through design assets | Technical | Medium | Low | Audit is Stage 1 gate (E1-S1); run before any token work. Document adjusted values early. | Developer |
| 4 | LiteLLM or chosen LLM provider has GDPR data processing implications for EU users submitting open-ended answers | Compliance | High | Medium | Architecture phase selects provider and evaluates DPA availability. Consider self-hosted model (Ollama) as GDPR-safe alternative. | Architect |
| 5 | E8 LLM call latency exceeds 5-second NFR threshold on the chosen provider at peak load | Technical | High | Medium | Architecture phase selects provider with sub-3s p95 latency. Implement 5-second timeout with E8-S5 fallback trigger. | Architect |
| 6 | Editor redesign (E5-S1, L-size) causes scope creep into adjacent backend quiz CRUD APIs | Technical | Medium | Low | E5 stories explicitly scope to visual layer only; any discovered backend changes must be flagged as a PRD deviation before implementation | Developer |
| 7 | Gamification animation scope grows beyond animation-only into new scoring mechanics | Scope | Medium | Low | E6 stories are explicitly bounded to CSS/JS animations. Any new scoring mechanic requires a PRD amendment and approval. | Jo Otey |

---

## Success Metrics

| Metric | Phase 0 Criterion | Target | Measurement Method | Frequency | Baseline |
|--------|-------------------|--------|-------------------|-----------|----------|
| Design system token coverage | Criterion 4 | 100% of new/modified components reference design tokens (zero hardcoded hex values) | Automated CSS grep / lint rule | Per PR | 0% — no token layer exists today |
| User-reported UI design complaints | Criterion 1 | Zero design-related complaints in post-event feedback for 3 consecutive hosted games | Manual feedback collection by host | Per event | Qualitative — "looks like a school thing" is current verbal feedback pattern |
| Demo first-impression reaction | Criterion 3 | Unprompted positive verbal or written reaction within 30 seconds of seeing the new UI | Live demo observation note | Per demo | No current baseline — this is new |
| Host return rate (≥2 games in 7 days) | Criterion 2 | ≥ 25% of new hosts run a second game within 7 days of first game | Server session log analysis | Weekly | Unknown — not currently tracked |

---

## Implementation Milestones

### Milestone 1: Design System Foundation

**Goal:** The High-Voltage Arcade design token layer, Outfit font, WCAG-verified palette, and base components (Button, Card) exist and are usable by all other epics.
**Stories Included:** E1-S1, E1-S2, E1-S3, E1-S4, E1-S5, E4-S1, E6-S4
**Depends On:** None

---

### Milestone 2: Core Game Loop Reskinned

**Goal:** A complete live game session — from PIN entry to final results — works end-to-end with the new design on both mobile (player) and desktop/TV (host).
**Stories Included:** E2-S1, E2-S2, E2-S3, E2-S4, E2-S5, E2-S6, E3-S1, E3-S2, E3-S3, E3-S4, E3-S5
**Depends On:** Milestone 1

---

### Milestone 3: Shell & Homepage

**Goal:** The public-facing homepage and navigation shell carry the new visual identity; any visitor sees the party brand immediately.
**Stories Included:** E4-S2, E4-S3
**Depends On:** Milestone 1

---

### Milestone 4: LLM Open-Ended Questions

**Goal:** A host can add open-ended questions to a quiz and run a live game where player answers are judged in real time by the LiteLLM evaluation service, with a manual fallback if the LLM is unavailable.
**Stories Included:** E8-S1, E8-S2, E8-S3, E8-S4, E8-S5
**Depends On:** Milestone 2 (game loop must be stable before adding a new question type to it)

---

### Milestone 5: Creator Workflow & Dashboard

**Goal:** The quiz editor is fully reskinned and the dashboard and supporting screens complete the full-platform design consistency.
**Stories Included:** E5-S1, E5-S2, E5-S3, E5-S4, E7-S1, E7-S2, E7-S3
**Depends On:** Milestone 1, Milestone 3

---

### Milestone 6: Gamification & Polish

**Goal:** Answer feedback animations, score counters, and leaderboard movement animations are layered on top of the core game loop screens.
**Stories Included:** E6-S1, E6-S2, E6-S3
**Depends On:** Milestone 2

---

## Task Breakdown

> **Path Conventions:**
> - Frontend: `frontend/src/`
> - Frontend components: `frontend/src/lib/components/`
> - Frontend routes: `frontend/src/routes/`
> - Backend: `classquiz/`
> - Socket server: `classquiz/socket_server/`
> - New LLM module: `classquiz/llm/`
> - Tests: `classquiz/tests/` (backend), `frontend/` (Vitest for frontend)

---

### Stage 1: WCAG Audit Gate (Blocking Prerequisite)

**Purpose:** Run and document the colour contrast audit before any token work begins.

- [ ] T001 [E1-S1] Run WCAG AA contrast audit on all five accent colours (#FFD02F, #00E055, #2D8CFF, #FF4545) against #0f1012 using axe-core CLI or Colour Contrast Analyser; document results in `specs/decisions/wcag-audit.md`
- [ ] T002 [E1-S1] If any colour fails, record the adjusted hex value that passes in `specs/decisions/wcag-audit.md`

**Checkpoint:** ☐ Audit complete and all token values confirmed — proceed to Stage 2

---

### Stage 2: Design System Foundation (Blocking Prerequisites)

**Purpose:** Establish the token layer, font, and base components that all story stages depend on.

- [ ] T003 [E1-S2] Add `@fontsource/outfit` to `frontend/package.json` and import weights 400, 700, 900 in `frontend/src/app.css`
- [ ] T004 [E1-S2, E1-S3] Define `@theme` CSS custom properties block in `frontend/src/app.css`: all colour tokens, --font-display, --shadow-hard, --shadow-hard-sm, --radius-base
- [ ] T005 [P] [E1-S2] Verify no requests to fonts.googleapis.com in DevTools; document in insights
- [ ] T006 [E4-S1] Apply --color-bg to `html, body` in `frontend/src/app.css`; apply --font-display as default font stack
- [ ] T007 [E1-S4] Create `frontend/src/lib/components/ui/Button.svelte` — primary, secondary, ghost, danger variants; Svelte 5 Runes; SPDX header
- [ ] T008 [P] [E1-S4] Write Vitest unit test for Button component variants in `frontend/src/lib/components/ui/Button.test.ts`
- [ ] T009 [E1-S5] Create `frontend/src/lib/components/ui/Card.svelte` — dark surface, hard border, shadow; Svelte 5 Runes; SPDX header
- [ ] T010 [E6-S4] Add `@media (prefers-reduced-motion: reduce)` utility block to `frontend/src/app.css` that stubs all CSS transition and animation durations to 0ms

**Checkpoint:** ☐ Token layer live, Button and Card components pass unit tests, dark base applying to all pages — proceed to story stages

---

### Stage 3: E2-S1 — PIN Entry Screen (Must Have)

**Goal:** Player can enter a PIN and name on mobile and be routed to the lobby.
**Independent Test:** Navigate to /play at 375px; enter a valid game PIN; confirm lobby screen loads.

- [ ] T011 [E2-S1] Locate existing PIN entry route in `frontend/src/routes/`; migrate to Svelte 5 Runes
- [ ] T012 [E2-S1] Apply token-driven styles: dark background, Outfit font, Button component for "Join" CTA
- [ ] T013 [E2-S1] Verify inputs are visible above fold at 375px; tap targets ≥ 44px height
- [ ] T014 [E2-S6] Add inline error state for invalid PIN (no redirect; input reset)
- [ ] T015 [P] [E2-S1] Write Playwright or Vitest integration test: valid PIN → lobby; invalid PIN → error message

**Checkpoint:** ☐ PIN entry screen fully functional and independently testable at 375px

---

### Stage 4: E2-S2, E2-S3, E2-S4, E2-S5, E2-S6 — Player Mobile Flow (Must Have)

**Goal:** Full player journey — lobby → question → leaderboard → results — works on mobile with new design.
**Independent Test:** Run a complete game loop as a player on a 375px viewport; verify each transition.

- [ ] T016 [E2-S2] Redesign lobby wait screen; migrate to Runes; animated "waiting" indicator using CSS; apply tokens
- [ ] T017 [E2-S3] Redesign question/answer screen; large colour-coded answer buttons (≥ 44px); selected-state highlight; timer display; migrate to Runes
- [ ] T018 [E2-S4] Redesign player leaderboard screen; rank and score display; rank improvement indicator; migrate to Runes
- [ ] T019 [E2-S5] Redesign player results screen; rank, score, correct-count above fold at 375px; migrate to Runes
- [ ] T020 [E2-S6] Add disconnect/reconnect handler: reconnection indicator, 5-attempt retry, error state with CTA
- [ ] T021 [P] [E2-S3] Write integration test: question renders, tap answer, confirm lock; timer expiry submission
- [ ] T022 [P] [E2-S6] Write integration test: mock socket disconnect → reconnect indicator shown → reconnect success → state restored

**Checkpoint:** ☐ All five player screens functional and tested at 375px

---

### Stage 5: E3-S1 through E3-S5 — Host & TV Screens (Must Have)

**Goal:** Full host journey — lobby (TV PIN screen) → question display → reveal → leaderboard → winner — reskinned for 1280px TV display.
**Independent Test:** Run a complete host-side flow at 1280px; verify each screen renders correctly.

- [ ] T023 [E3-S1] Redesign host lobby screen; ≥ 96px game PIN; real-time player join list; migrate to Runes; apply tokens
- [ ] T024 [E3-S2] Redesign question display; question text + answer options; animated countdown timer; migrate to Runes
- [ ] T025 [E3-S3] Redesign answer tally and reveal; live response counter; correct answer highlight; per-option stats; migrate to Runes
- [ ] T026 [E3-S4] Redesign host leaderboard; top-N ranking; rank movement indicators; ≥ 32px font size; migrate to Runes
- [ ] T027 [E3-S5] Redesign winner/final results screen; winner prominence; co-winner support; cinematically distinct from intermediate leaderboard; migrate to Runes
- [ ] T028 [P] [E3-S1] Write test: player join event → name appears in list within 1s
- [ ] T029 [P] [E3-S4] Write test: leaderboard renders top-5; rank movement indicator present for players who moved

**Checkpoint:** ☐ All five host screens functional and tested at 1280px

---

### Stage 6: E4-S2, E4-S3 — Navigation & Homepage (Must Have)

**Goal:** Public homepage and global navigation carry the new brand identity.
**Independent Test:** Visit root URL unauthenticated at 1280px and 375px; verify hero, navigation, and CTA.

- [ ] T030 [E4-S2] Create/redesign `frontend/src/lib/components/layout/Nav.svelte`; logo, links, auth CTA states; mobile collapse; Runes; SPDX
- [ ] T031 [E4-S3] Redesign `/` homepage route; hero section, headline, sub-headline, primary CTA; bento grid layout; token classes; Runes; SPDX
- [ ] T032 [E4-S3] Add authenticated-user homepage variant (recent quizzes or redirect)
- [ ] T033 [P] [E4-S3] Write test: unauthenticated root → hero + CTA visible; authenticated root → redirect or recent quizzes shown

**Checkpoint:** ☐ Navigation and homepage pass visual verfication at both 375px and 1280px

---

### Stage 7: E8 — LLM Open-Ended Questions (Must Have)

**Goal:** Open-ended question type functions end-to-end: editor → live game → LiteLLM evaluation → verdict display → fallback.
**Independent Test:** Create a quiz with an open-ended question; run a game; submit a text answer; verify verdict or fallback activates.

- [ ] T034 [E8-S1] Add `open_ended` to question type enum in `classquiz/db/models.py`; create Alembic migration
- [ ] T035 [E8-S1] Update socket server in `classquiz/socket_server/` to broadcast `open_ended` question events and queue text answer submissions
- [ ] T036 [E8-S2] Create `classquiz/llm/__init__.py` module; implement `evaluate_answer(question: str, answer: str) -> Verdict` using LiteLLM; SPDX header
- [ ] T037 [E8-S2] Configure LiteLLM provider via environment variable `LITELLM_MODEL` (e.g., `gpt-4o-mini`); never hardcode API key
- [ ] T038 [E8-S2] Wire LLM evaluation into socket server answer handler; emit verdict events to player and host; implement 5-second timeout triggering E8-S5 fallback
- [ ] T039 [E8-S3] Create open-ended question player view component at `frontend/src/lib/components/game/OpenEndedPlayer.svelte`; text input + submit; locked state; Runes; SPDX
- [ ] T040 [E8-S4] Add verdict count display to host question screen (E3-S3 update): "Evaluated: N / M"
- [ ] T041 [E8-S5] Create manual review panel component at `frontend/src/lib/components/game/ManualReviewPanel.svelte`; list of player answers with Correct/Incorrect buttons; Runes; SPDX
- [ ] T042 [P] [E8-S1] Write backend test: quiz with open_ended type created, saved, and retrieved
- [ ] T043 [P] [E8-S2] Write backend test: `evaluate_answer` called with mocked LiteLLM; correct verdict returned; timeout triggers fallback
- [ ] T044 [P] [E8-S5] Write test: mock LLM failure → manual review panel appears → host marks answer → score updates

**Checkpoint:** ☐ Full open-ended question loop functional; LLM failure fallback verified

---

### Stage 8: E5 — Quiz Editor Redesign (Should Have)

**Goal:** Editor shell, question cards, and settings form are reskinned using the new component library.
**Independent Test:** Create a new quiz with 3 MC questions and 1 open-ended question via editor; all functionality works with new design.

- [ ] T045 [E5-S1] Redesign editor route layout; left panel (question list), canvas area, top toolbar; migrate to Runes; apply tokens
- [ ] T046 [E5-S1] Toolbar actions: Add Question, Save, Preview, Settings
- [ ] T047 [E5-S2] Redesign multiple-choice question card: question text, 4 option inputs, correct-answer indicator; Runes
- [ ] T048 [E5-S3] Redesign quiz settings form: title (required), description, visibility toggle; inline validation; Runes
- [ ] T049 [E5-S4] Add additional question type shells (open text, true/false) to Add Question menu with "Coming soon" badges; Runes

**Checkpoint:** ☐ Editor functional with new design; quiz CRUD regression tests pass (NFR-BC08)

---

### Stage 9: E6-S1, E6-S2, E6-S3 — Gamification Animations (Should Have)

**Goal:** Answer feedback flash, score counter, and leaderboard rank animations are live in the game flow.
**Independent Test:** Play a full game round; verify animations play (and are suppressed with prefers-reduced-motion).

- [ ] T050 [E6-S1] Add correct/incorrect flash animation to player answer feedback in `OpenEndedPlayer.svelte` and MC answer screen; CSS keyframes; prefers-reduced-motion override
- [ ] T051 [E6-S2] Add score counter increment animation to player leaderboard and results screens; CSS counter or JS tween ≤ 500ms; prefers-reduced-motion override
- [ ] T052 [E6-S3] Add leaderboard row reorder animation (slide/cross-fade) to host and player leaderboard screens; CSS transition; prefers-reduced-motion override

**Checkpoint:** ☐ All animations play at ≥ 60fps; all suppressed with prefers-reduced-motion: reduce

---

### Stage 10: E7 — Dashboard & Supporting Screens (Should Have)

**Goal:** Dashboard, results summary, and profile/settings complete the full-platform visual consistency.
**Independent Test:** Log in; view dashboard; verify bento grid; navigate to settings; update display name; verify nav update.

- [ ] T053 [E7-S1] Redesign `/dashboard` route; bento grid of quiz cards; empty state CTA; Runes; apply tokens
- [ ] T054 [E7-S2] Redesign post-game results summary page; ranked player list; bento grid at 1280px; stacked list at 375px; Runes
- [ ] T055 [E7-S3] Redesign profile/settings pages; apply global dark layout (E4-S1); display name update → nav reflects change; Runes

**Checkpoint:** ☐ Dashboard, results, and settings pages match design system; nav display-name update test passes

---

### Stage N: Polish & Cross-Cutting Concerns

- [ ] TXXX [P] Add SPDX MPL-2.0 header audit to CI: verify all `frontend/src/**` files modified in the PR carry the header
- [ ] TXXX [P] Run full axe-core Playwright accessibility scan against all redesigned routes; fix any issues scoring < 90
- [ ] TXXX [P] Run Lighthouse mobile simulation on /play, /dashboard, and / at 375px; verify performance scores
- [ ] TXXX Add `classquiz/llm/__init__.py` to Sentry error monitoring scope
- [ ] TXXX Code review: grep `frontend/src/` for hardcoded hex values — none should remain after E1-S2 completion

---

### Dependencies & Execution Order

#### Stage Dependencies

```
Stage 1: WCAG Audit Gate
    ↓
Stage 2: Design System Foundation (BLOCKS all story stages)
    ↓
Stage 3: E2-S1 PIN Entry      Stage 5: Host/TV Screens      Stage 6: Nav & Homepage
Stage 4: E2 Full Player Flow ─────────────────────────────────────────────────────────┐
    ↓                                                                                  │
Stage 7: E8 LLM Questions (depends on Stage 4 game loop stability)                    │
    ↓                                                                                  │
Stage 8: E5 Editor (depends on Stage 2 tokens; can start parallel to Stage 4/5)       │
Stage 9: E6 Animations (depends on Stages 4 and 5)                                   │
Stage 10: E7 Dashboard (depends on Stage 2 + Stage 6)   ◄──────────────────────────────┘
    ↓
Stage N: Polish & Cross-Cutting
```

---

## Glossary

| Term | Definition |
|------|-----------|
| Design Token | A named CSS custom property (e.g., `--color-accent-yellow`) that encodes a design decision and is referenced by all components rather than using hardcoded values |
| `@theme` | TailwindCSS 4's config-file-free mechanism for defining design tokens as CSS custom properties in a CSS file |
| Bento Grid | A layout pattern inspired by Japanese bento boxes — a grid of variably-sized, hard-bordered content cards |
| Hard Shadow | The `box-shadow: 4px 4px 0px 0px #000` visual pattern characteristic of Tactile Neo-Brutalism style |
| LiteLLM | A Python library providing a unified interface to multiple LLM providers (OpenAI, Anthropic, Ollama, etc.) |
| Verdict | The result of LiteLLM evaluation of an open-ended answer — one of: `correct`, `incorrect`, or `partial` |
| Svelte 5 Runes | The Svelte 5 reactivity model using `$state`, `$derived`, and `$props` syntax, replacing legacy Svelte 4 store patterns |
| INVEST | Criteria for well-formed user stories: Independent, Negotiable, Valuable, Estimable, Small, Testable |
| High-Voltage Arcade | The name of the PartyQuiz design system — dark background, electric accent palette, Outfit font, hard shadows |

---

## Insights Reference

**Companion Document:** [specs/insights/prd-insights.md](insights/prd-insights.md)

Key insights that shaped this document:

1. **LLM scope expansion** — E8 overrides the Product Brief's Won't Have boundary for backend question type changes. This was explicitly confirmed in Round 3 and materially affects the risk profile and architecture scope.
2. **E6-S4 is cross-cutting, not optional** — `prefers-reduced-motion` compliance was classified Must Have because it is an accessibility requirement tied to the WCAG AA standard, not a gamification feature.
3. **E1-S1 WCAG audit is Stage 1 gate** — All token values depend on this audit passing. Making it a blocking Stage 1 task prevents downstream rework if any colour fails.

See the insights document for complete decision rationale, epic boundary logic, scope trade-offs, and dependency discoveries.

---

## Phase Gate Approval

- [x] Human has reviewed this PRD
- [x] Every epic has at least one user story
- [x] Every Must Have story has at least 2 acceptance criteria
- [x] Acceptance criteria are specific and testable (no vague qualifiers)
- [x] Non-functional requirements have measurable thresholds
- [x] At least one implementation milestone is defined
- [x] Task breakdown includes Setup, Foundational, and at least one user story stage
- [x] Dependencies have identified mitigations
- [x] Risks have identified mitigations
- [x] Success metrics map to Phase 0 validation criteria
- [x] Human has explicitly approved this PRD for Phase 3 handoff

**Approved by:** Jo Otey
**Approval date:** 2026-02-24
**Status:** Approved

---

## Linked Data

```json-ld
{
  "@context": { "js": "https://jumpstart.dev/schema/" },
  "@type": "js:SpecArtifact",
  "@id": "js:prd",
  "js:phase": 2,
  "js:agent": "PM",
  "js:status": "Draft",
  "js:version": "1.0.0",
  "js:upstream": [
    { "@id": "js:challenger-brief" },
    { "@id": "js:product-brief" }
  ],
  "js:downstream": [
    { "@id": "js:architecture" },
    { "@id": "js:implementation-plan" }
  ],
  "js:traces": []
}
```
