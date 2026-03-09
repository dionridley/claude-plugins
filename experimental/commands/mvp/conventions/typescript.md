# TypeScript/React Conventions Reference

This file is read by the `/mvp build` orchestrator at the start of each build session when the stack is JavaScript (Vite + TypeScript + React 19 + TailwindCSS 4).

<!-- PLACEHOLDER: This file is intentionally minimal in v0.1.0. It will be populated with TypeScript/React-specific standing rules, anti-patterns, and code patterns as real builds surface common issues — mirroring the approach taken in conventions/elixir.md. -->

---

## Mandatory Rules for Every Agent

Include this block verbatim in all TypeScript/React agent prompts:

```
## TypeScript/React Conventions — MANDATORY

**TypeScript:**
- NEVER use `any` type — use `unknown` and narrow, or define a proper type/interface
- ALWAYS read an existing file with the Read tool before modifying it
- Only use Write directly for files that do not yet exist

**Components:**
- Use functional components only — no class components
- Keep component files focused: one primary export per file
- Co-locate types with the component that owns them

**Navigation:**
- Use React Router `<Link to="...">` and `useNavigate()` — never `window.location`
- Use the `to` prop with relative paths where possible

**Forms:**
- Use React 19 Form Actions and `useActionState` for form handling — not manual `useState` + handlers
- See the react-19 engineering skill for full API reference

**Styling:**
- Use TailwindCSS 4 utility classes — no inline styles unless dynamically computed
- `@import "tailwindcss"` is the entry point in `src/index.css` — no config file needed in v4

**Database (Drizzle ORM + better-sqlite3):**
- Define schemas in `src/db/schema.ts`
- Use `drizzle-kit push` to apply schema changes during development (no migration files needed)
- Keep all DB access in `src/lib/db/` — never query directly from components
```

---

## Stack Reference

### Project Structure
```
src/
  components/     # Shared UI components
  pages/          # Route-level page components
  lib/
    db/           # Database access layer (Drizzle + SQLite)
  types/          # Shared TypeScript types
  index.css       # TailwindCSS 4 entry point (@import "tailwindcss")
  main.tsx        # React entry point
  App.tsx         # Router setup
```

### Vitest Test Patterns
```typescript
// src/lib/db/things.test.ts
import { describe, it, expect, beforeEach } from 'vitest'
import { createThing, listThings } from './things'

describe('things', () => {
  it('creates a thing with valid data', () => {
    const result = createThing({ name: 'Test' })
    expect(result.name).toBe('Test')
  })
})
```

---

<!-- More patterns will be added here as real builds surface common issues. -->
