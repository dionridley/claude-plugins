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
- **Database:** SQLite (via Ecto SQLite3 adapter)

## Prerequisites

### JavaScript Stack
- Node.js >= 18
- npm
- npx

### Elixir Stack
- Elixir >= 1.15
- Mix
- Hex (`mix local.hex`)

## How It Works

1. **`/mvp start`** — Interactive brainstorming session that captures your app idea, bounds scope to 3-5 screens and 1 core user flow, checks prerequisites, scaffolds the project, and creates a `.mvp/` directory with a brainstorm document and state file.

2. **`/mvp build`** — Builds the prototype autonomously. Reads the brainstorm document, identifies the next batch of tasks, dispatches parallel AI agents for independent work, runs quality reviews on completed tasks, and auto-commits to git at each checkpoint. Pauses at phase boundaries for user review. If prior progress exists, picks up where it left off.

3. **`/mvp status`** — Displays a terminal-friendly dashboard showing phase progress, task counts, agent statistics, and elapsed time.

4. **`/mvp summary`** — Generates a self-contained HTML page with charts and analytics about the entire build process.

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
- **Quality review agents** verify each completed task before it's committed
- **Lock-based concurrency** prevents conflicts on shared resources (migrations, design, dependencies)
