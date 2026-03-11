# Changelog

All notable changes to the Experimental plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.3.2] - 2026-03-10

### Fixed
- Playwright browser binary check was unconditional in the Elixir scaffold path — now checks for existing Playwright CLI and Chromium binary before installing, matching the TypeScript scaffold behavior

## [0.3.8] - 2026-03-10

### Added
- **Richer `resumePoint` schema** — now includes `nextAction` (one-sentence concrete next step), `lastCompletedTask`, `recentFilesChanged` (up to 5 files), and `openIssues` (deferred decisions, partial failures, blockers); updated after every completed task
- **Structured agent logs** written to `.mvp/agent-logs/[agent-id].json` after each agent completes — captures task, files, quality review verdict, browser test result, and a `decisions` field for non-obvious choices that would otherwise be lost when context compacts
- **Session-start context reconstruction** — Phase 1 of `/mvp build` now reads `state.json`, the brainstorm file, and the 3 most recent agent logs, then outputs a "where we are" summary block to the conversation before doing anything else; provides working context for resumed sessions without relying on conversation history

## [0.3.7] - 2026-03-10

### Added
- **Silent failure check** in quality review agent — reviewer now explicitly reads every event handler, form action, and button in modified files and fails any that return early or catch errors without visible user feedback
- **Empty state + error path steps** added as mandatory items in the Playwright per-screen browser test script — tests that the screen shows meaningful content with no data, and that invalid/failed actions produce visible feedback (not a silent no-op)
- **End-of-phase regression smoke test** after `core` and `polish` phases (when Playwright is enabled) — walks the complete core user flow and navigates every screen, catching regressions introduced by work in that phase before advancing to the next

### Changed
- Phase `polish` browser testing changed from "reserve for significant changes" to per-screen tests for all new screens built in the phase, matching Phase `core` behavior

## [0.3.6] - 2026-03-10

### Added
- `"typecheck": "tsc --noEmit"` script added to `package.json` at scaffold time (JS stack)

### Changed
- Quality review agent now runs `npm run typecheck` (`tsc --noEmit`) for JS tasks instead of the previous vague instruction to "look for TypeScript type errors" — catches type errors that Vite's dev server silently ignores, at the per-task level rather than only at final build time
- Elixir quality review compile check now explicitly states warnings are treated as failures (matching the Elixir conventions rule)

## [0.3.5] - 2026-03-10

### Fixed
- `git init` now runs in Phase 4 of `/mvp start` (alongside `.mvp/` directory creation), before the required restart — ensures worktree isolation is available from the first build agent dispatch even if the session was interrupted between start and build
- `/mvp build` Phase 1 now explicitly checks `git rev-parse --git-dir` before dispatching any agents; if the repo is not initialized, it runs `git init` + baseline commit and logs a warning — worktree isolation can no longer silently fail

## [0.3.4] - 2026-03-10

### Fixed
- `vite.config.ts` is now written from scratch during scaffold (not read-and-edited) with `defineConfig` imported from `'vitest/config'` instead of `'vite'` — prevents a TypeScript error (`'test' does not exist in type 'UserConfigExport'`) that only surfaces during production builds, not `npm run dev`. The `test` block stub with `jsdom` environment is included from the start.
- Tidewave Vite plugin import corrected to `tidewave/vite-plugin` (was `tidewave/vite`) — the old path caused `Missing "./vite" specifier` at build time.

## [0.3.3] - 2026-03-10

### Fixed
- Scaffold now uses a named subdirectory (`[slug]/`) instead of `.` for both Vite and Phoenix — avoids the interactive "directory not empty" TTY prompt that silently exits when `.mvp/` files already exist in the project root. Files are copied up and the subdirectory removed immediately after scaffold completes.

## [0.3.2] - 2026-03-10

### Added
- **Port health check at every `/mvp build` session start** — before starting any processes, verifies that the ports in `state.json` are free; identifies occupying processes, kills known stale MVP PIDs with command verification, and stops with a clear message if an unknown process is holding the port
- **Process kill reference at session end** — when `serverManagement == "agent"`, every build session pause (Step G) and completion (Phase 7) now prints a process cleanup block with exact `kill [pid]` commands for all processes started during the session, even if they were already stopped

### Changed
- **Non-default ports for all MVP projects** to avoid collisions with standard framework defaults:
  - JavaScript: Express API on `3500` (was `3001`), Vite frontend on `3600` (was `5173`)
  - Elixir: Phoenix on `4500` (was `4000`)
- Port conflicts in Phase 3 prerequisites now **stop with a clear error** instead of silently auto-incrementing to the next port — silent port bumps caused hard-to-diagnose API failures
- `conventions/typescript.md` port reference section updated: explains the non-default port strategy, removes the dangerous `lsof -ti:[port] | xargs kill` pattern

## [0.3.0] - 2026-03-09

### Added
- **Auto-configured MCP servers** — `/mvp start` writes `.mcp.json` and `.claude/settings.local.json` into the project directory automatically; no manual Claude Code settings required
  - Tidewave MCP (`mcp__tidewave__*`) configured for both Elixir and JavaScript stacks
  - Playwright MCP (`@playwright/mcp@latest --isolated --caps=vision`) configured when E2E testing is opted in
- **Stack permissions files** — `commands/mvp/settings/elixir.json` and `commands/mvp/settings/typescript.json` define all required tool permissions written to new projects at setup time
- **Server management choice** — `/mvp start` now asks whether the dev server should be agent-managed (fully autonomous) or user-managed (user runs server in a separate terminal; agent instructs when to restart). Stored in `state.json` and respected throughout the build.
- **Tidewave in TypeScript scaffold** — `@tidewave/tidewave` installed, Express middleware wired up, MCP server available at `/tidewave`
- **Prescriptive TypeScript server structure** — scaffold now creates `server/index.ts`, `server/db/schema.ts`, `server/db/client.ts`, `server/db/seed.ts`, and `server/lib/routes.ts` (shared API route constants used by both Express and the fetch client to prevent URL mismatches)
- **Playwright browser binaries installed at setup** — `npx playwright install chromium` runs during scaffold if Playwright is opted in; restart-and-resume instructions shown at completion
- **Swoosh hackney fix as a concrete scaffold step** — `config/dev.exs` and `config/test.exs` patched at Elixir scaffold time; previously convention-only
- **Tidewave added to Elixir scaffold** — `{:tidewave, "~> 0.1", only: :dev}` and router setup added as scaffold steps
- **Process safety warnings** — agent-managed builds show a per-stack warning at session start about same-name process collision risk (`mix phx.server`, `vite`, `tsx`)
- **Command verification before all process kills** — every kill operation now checks `ps -p [pid] -o args=` against the stored command fragment; PIDs recycled to unrelated processes are marked `"recycled"` and skipped rather than killed
- **Subagent PID lifecycle tracking** — `state.processes.subagentPids` tracks every background process started by a subagent with full start/stop metadata; swept at session start, phase completion, and build completion
- **`--resume` flag instructions** — completion message now tells users to copy the resume command before restarting Claude Code so they can continue the conversation with full context

### Changed
- Projects now scaffold into the current directory (using `.`) instead of creating a subdirectory — matches expected workflow of `mkdir my-app && cd my-app && claude`
- Playwright question updated to reflect automatic MCP configuration; "requires manual setup" caveat removed
- `lsof -ti:[port] | xargs kill` pattern removed entirely from agent-managed server startup; replaced with PID-based kill with command verification
- `typescript-conventions.md` fully fleshed out — `verbatimModuleSyntax`/`import type` rule (blank-page prevention), DB-in-callbacks prohibition, `npx tsc --noEmit` quality gate, idempotent seed pattern, API URL contract rule, port handling, Tidewave setup, Playwright smoke test guidance
- `elixir-conventions.md` expanded — Swoosh config rule, `flash_group` Layouts import rule, compiler warnings as failures, `signed_in_path` test update rule, Tidewave usage rule
- Convention files reorganised into `commands/mvp/conventions/` folder; filenames simplified to `elixir.md` and `typescript.md`

## [0.2.0] - 2026-03-08

### Added
- Playwright E2E testing support — opt-in during `/mvp start`, enabled per-screen in Phase 5 (core feature) if chosen
- Mandatory Phase 3: Test Scaffolding — unit tests for all data/context functions must pass before proceeding
- Mandatory Phase 4: Design Brief — generated by main agent and saved to `.mvp/research/design-brief.md` before any UI agents run
- Stack conventions system — `commands/mvp/conventions/` directory with per-stack mandatory rules injected into every agent prompt
  - `conventions/elixir.md` — Phoenix 1.8 standing rules, patterns, and quick reference (DaisyUI, LiveView, context, factory, migrations)
  - `conventions/typescript.md` — TypeScript/React 19 standing rules (placeholder, to be expanded with real build learnings)
- Elixir scaffold hardening in `/mvp start`: OTP 25+ check, `--no-install` flag, Faker dependency, asset binary pre-install, boilerplate file cleanup, `root.html.heex` flash group patch, `runtime.exs` port wrapping, port conflict detection
- Low-risk task flag — tasks like README writes and CSS appends skip quality review
- Worktree isolation is now the default for subagents touching more than 2 files

### Changed
- Build process expanded from 5 phases to 7 phases to accommodate test scaffolding and design brief gates
- Integration phase (Phase 7) now requires full test suite to pass with 0 failures before final commit
- `brainstorm.md` filename is no longer prompted — always defaults to `brainstorm.md`

## [0.1.0] - 2026-02-24

### Added
- Initial release of experimental plugin
- `/mvp start` command for brainstorming and project scaffolding
- `/mvp build` command for autonomous building with parallel AI agents
- `/mvp status` command for progress dashboard
- `/mvp summary` command for HTML analytics page generation
- JavaScript stack support (Vite + TypeScript + React 19 + TailwindCSS 4 + SQLite)
- Elixir stack support (Phoenix Framework + LiveView + SQLite)
- Persistent state management via `.mvp/` directory
- Lock-based agent concurrency control
- Quality review agent pattern for completed tasks
- Auto git commits at task completion checkpoints
