# ADR-003: Design Tokens Delivered via TailwindCSS 4 @theme in app.css

## Status
Accepted

## Context

The PRD (E1-S2) requires a design token layer for the "High-Voltage Arcade" visual system: colour palette, typography, spacing, shadow, and border-radius tokens. TailwindCSS 4 is already in use in the project.

Two approaches were evaluated:
1. **Separate `$lib/tokens.ts` (or similar)** — a TypeScript module exporting token values, which Svelte components import
2. **TailwindCSS 4 `@theme {}` block in `app.css`** — CSS custom properties defined inline in the CSS file, consumed by Tailwind utility classes and raw CSS variable references

## Decision

Define all design tokens as CSS custom properties in a `@theme {}` block in `frontend/src/app.css`. No JavaScript or TypeScript module is created to expose token values.

**Token naming convention:**
```css
@theme {
  --color-bg-void: #0f1012;
  --color-accent-electric: #FFD02F;  /* verified WCAG AA against bg-void */
  --color-accent-green: #00E055;
  --color-accent-blue: #2D8CFF;
  --color-accent-red: #FF4545;
  --font-display: "Outfit", sans-serif;
  --shadow-hard: 4px 4px 0px 0px #000;
  --shadow-hard-sm: 2px 2px 0px 0px #000;
  --radius-base: 0px;  /* Neo-brutalism: zero radius */
}
```

Svelte components reference tokens exclusively via Tailwind utility classes (e.g., `bg-accent-electric`) or CSS variable syntax (`var(--color-bg-void)`). Hardcoded hex values in component files are prohibited (CI lint step enforces this).

## Consequences

### Positive
- Fully aligned with TailwindCSS 4's native design-token mechanism — no framework fight
- Single source of truth: one file (`app.css`) contains all token values
- WCAG audit gate (E1-S1) has a clear, singular target: audit the `@theme` block in `app.css`, lock values, then all downstream components inherit correct colours
- Anti-Abstraction Gate (Article VII) satisfied — no wrapper module around a framework primitive
- Zero runtime JS overhead — tokens are static CSS custom properties

### Negative
- Token values are not accessible to TypeScript (no type-safe token auto-complete in component code)
- Any future token value changes require searching for usages across all component CSS

### Neutral
- TailwindCSS 4 config-file-free approach means the `@theme` block is the configuration — this is by design

## Alternatives Considered

### Separate `$lib/tokens.ts` JavaScript module
- Pros: Type-safe; can be imported in TypeScript; enables programmatic theming
- Cons: Creates a JS abstraction layer over TailwindCSS 4's native CSS custom property mechanism; violates Anti-Abstraction Gate (Article VII); duplicates token values (CSS layer would still need the properties for Tailwind utilities to work)
- Reason rejected: Unnecessary abstraction. TailwindCSS 4's `@theme` is specifically designed for this use case.
