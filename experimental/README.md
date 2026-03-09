# Experimental Plugin

Autonomous MVP builder that brainstorms, scaffolds, and builds web app prototypes using parallel AI agents.

## Commands

| Command | Description |
|---------|-------------|
| `/mvp start` | Brainstorm an app idea, choose a tech stack, and scaffold the project |
| `/mvp build` | Build the prototype autonomously (resumes from saved state if prior progress exists) |
| `/mvp status` | View current progress dashboard |
| `/mvp summary` | Generate an HTML analytics page of the build process |

## Supported Tech Stacks

### JavaScript Stack
- **Bundler:** Vite (latest)
- **Language:** TypeScript
- **UI Framework:** React 19.x
- **CSS:** TailwindCSS 4.x
- **Database:** SQLite (via better-sqlite3 / Drizzle ORM)

### Elixir Stack
- **Framework:** Phoenix Framework (latest)
- **UI:** LiveView
- **CSS:** TailwindCSS 4.x + DaisyUI 5.x (ships by default in Phoenix 1.8)
- **Database:** SQLite (via Ecto SQLite3 adapter)

## Prerequisites

### JavaScript Stack
- Node.js >= 18
- npm / npx

### Elixir Stack
- Elixir >= 1.15
- OTP >= 25 (required for asset downloads)
- Mix + Hex (`mix local.hex`)

## How It Works

1. **`/mvp start`** — Interactive brainstorming session that captures your app idea, bounds scope to 3-5 screens and 1 core user flow, asks about Playwright E2E testing, checks prerequisites, scaffolds the project, and creates a `.mvp/` directory with a brainstorm document and state file.

2. **`/mvp build`** — Builds the prototype autonomously across 7 phases. Loads stack conventions at session start, identifies the next batch of tasks, dispatches parallel agents, runs quality reviews, and auto-commits to git at each checkpoint. Pauses at phase boundaries for user review. Picks up from saved state if prior progress exists.

3. **`/mvp status`** — Displays a terminal-friendly dashboard showing phase progress, task counts, agent statistics, and elapsed time.

4. **`/mvp summary`** — Generates a self-contained HTML page with charts and analytics about the entire build process.

## Build Phases

| Phase | Name | Description |
|-------|------|-------------|
| 1 | Scaffold | Project init, dependencies, routing skeleton, dev server verification |
| 2 | Data Layer | Schemas, migrations, seed data |
| 3 | Test Scaffolding | Unit tests for all data/context functions — gate: 0 failures |
| 4 | Design Brief | Main agent generates design brief; all UI agents use it |
| 5 | Core Feature | Primary user flow built screen by screen; optional Playwright tests |
| 6 | UI Polish | Remaining screens, navigation, responsive checks |
| 7 | Integration | Full test suite, README, final cleanup — gate: 0 failures |

## Project Artifacts

All state is persisted in a `.mvp/` directory in your project root:

```
.mvp/
├── brainstorm.md    # Source-of-truth document with vision, scope, and task tracking
├── state.json       # Machine-readable state for build orchestration and analytics
├── agent-logs/      # Individual agent run reports
├── research/        # Research artifacts gathered during brainstorming
└── resources/       # Downloaded assets (images, etc.)
```

## Agent Orchestration

The build orchestrator uses a main agent that coordinates subagents:

- **Main agent** handles infrastructure: dependency installs, server management, database migrations, git operations
- **Subagents** handle feature work: up to 2-3 running in parallel for independent tasks
- **Quality review agents** verify each completed task before it's committed; low-risk tasks (README, CSS appends) skip this step
- **Lock-based concurrency** prevents conflicts on shared resources (migrations, design, dependencies)
- **Worktree isolation** (`isolation: "worktree"`) is the default for agents touching more than 2 files

## Stack Conventions

Stack-specific standing rules are stored in `commands/mvp/conventions/` and injected into every agent prompt:

- `conventions/elixir.md` — Phoenix 1.8 rules, LiveView patterns, DaisyUI reference, test/factory patterns
- `conventions/typescript.md` — TypeScript/React 19 rules (expanded over time as builds surface new patterns)
