---
description: Autonomous MVP builder — brainstorm, scaffold, build, and track web app prototypes
argument-hint: <start|build|status|summary>
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(*), WebSearch, WebFetch, Task
---

# MVP Builder

Autonomous command that brainstorms an app idea with the user, scaffolds the project, and builds a working prototype using parallel AI agents.

## Instructions for Claude

You are executing the `/mvp` command. Detect the mode from `<command-args>` and delegate to the appropriate mode file.

### Mode Detection

**CRITICAL**: The command arguments are provided in the `<command-args>` tag at the top of this message.

1. **Check the `<command-args>` value:**
   - If it starts with "start" (or is empty/whitespace):
     -> **START mode**: Read `${CLAUDE_PLUGIN_ROOT}/commands/mvp/start.md` and follow those instructions
   - If it starts with "build":
     -> **BUILD mode**: Read `${CLAUDE_PLUGIN_ROOT}/commands/mvp/build.md` and follow those instructions
   - If it starts with "status":
     -> **STATUS mode**: Read `${CLAUDE_PLUGIN_ROOT}/commands/mvp/status.md` and follow those instructions
   - If it starts with "summary":
     -> **SUMMARY mode**: Read `${CLAUDE_PLUGIN_ROOT}/commands/mvp/summary.md` and follow those instructions

2. **If no recognized mode**, show usage help:
   ```
   MVP Builder - Autonomous web app prototype builder

   Usage:
     /mvp start     Brainstorm an idea, choose tech stack, scaffold project
     /mvp build     Build the prototype (resumes from saved state if prior progress exists)
     /mvp status    View current progress dashboard
     /mvp summary   Generate HTML analytics page

   Get started:
     /mvp start
   ```

---

## Shared Conventions

All modes share these conventions. The mode-specific files inherit this context.

### State Directory

All MVP state is persisted in `.mvp/` in the current working directory:

| File/Folder | Purpose |
|-------------|---------|
| `.mvp/brainstorm.md` | Source-of-truth document: vision, scope, task tracking, agent log |
| `.mvp/state.json` | Machine-readable state for build orchestration and analytics |
| `.mvp/agent-logs/` | Individual agent run reports (one file per agent) |
| `.mvp/research/` | Research artifacts gathered during brainstorming |
| `.mvp/resources/` | Downloaded assets (images, icons, etc.) |

### Supported Tech Stacks

**JavaScript Stack:**
- Vite (latest) + TypeScript
- React 19.x
- TailwindCSS 4.x
- SQLite via better-sqlite3 / Drizzle ORM

**Elixir Stack:**
- Phoenix Framework (latest)
- LiveView for page rendering
- SQLite via Ecto SQLite3 adapter

### Timestamps

All timestamps use ISO 8601 format: `YYYY-MM-DDTHH:MM:SS` (local time). Use the current date from the conversation context — NEVER hardcode dates.

### Git Conventions

- Commit messages use prefix: `mvp: [description]`
- Auto-commit after each quality-reviewed task completion
- Phase completion gets a summary commit
- Final completion gets a wrap-up commit

### Agent Communication Format

Subagents MUST return results as structured JSON:

```json
{
  "agentId": 1,
  "status": "success|partial|failed",
  "taskCompleted": "description of what was done",
  "filesModified": ["path/to/file1"],
  "filesCreated": ["path/to/new-file"],
  "dependenciesNeeded": ["package-name"],
  "processesStarted": [{"pid": 12345, "command": "npm run dev"}],
  "issues": ["any problems encountered"],
  "notes": "additional context"
}
```

### Lock System

`state.json` tracks resource locks to prevent agent conflicts:

| Lock | What it protects |
|------|-----------------|
| `migrations` | Database schema changes — only one agent at a time |
| `design` | Global CSS, theme, design system files |
| `dependencies` | package.json / mix.exs modifications (main agent only) |

### Main Agent Responsibilities

The main agent (not subagents) ALWAYS handles:
- Installing/removing dependencies
- Starting/stopping dev servers
- Running database migrations
- Git add, commit, push operations
- Modifying `.mvp/state.json` and `.mvp/brainstorm.md`
- Managing process PIDs
- Acquiring/releasing locks

---

## Execute Now

Detect the mode from `<command-args>` and read the appropriate mode file.
