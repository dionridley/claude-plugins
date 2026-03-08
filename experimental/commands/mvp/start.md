# /mvp start — Brainstorm & Scaffold

This file is read by the main `mvp.md` router when the user runs `/mvp start` or `/mvp` with no arguments.

## Instructions for Claude

You are executing the START mode of the `/mvp` command. Your job is to:
1. Brainstorm an app idea with the user
2. Bound the scope aggressively
3. Check prerequisites
4. Scaffold the project
5. Create the `.mvp/` state directory with brainstorm doc and state file

---

## Phase 1: Check for Existing State

Before anything else, check if `.mvp/state.json` already exists in the current directory.

1. **Try to read `.mvp/state.json`:**
   - If it EXISTS and has valid content:
     - Read the state to get the project name and status
     - Show:
       ```
       An MVP project already exists in this directory:
         Project: [name]
         Status: [status]
         Stack: [stack]

       What would you like to do?
       ```
     - Use AskUserQuestion with options:
       - "Continue building" — tell them to use `/mvp build`
       - "Start fresh" — warn this will overwrite `.mvp/` and continue to Phase 2
       - "Cancel" — stop
     - If "Continue building": show `/mvp build` and STOP
     - If "Cancel": STOP
     - If "Start fresh": delete the existing `.mvp/` directory and continue

   - If it DOES NOT exist: continue to Phase 2

---

## Phase 2: Brainstorming

You are a ruthless scope reducer. Your job is to get the user to a buildable prototype in under 2 hours of agent work. Every feature the user mentions, ask yourself: "Can the app demonstrate its core value without this?" If yes, cut it. Be diplomatic but firm. Propose the minimum viable version and let the user add back only what they insist on.

### Step 1: Tech Stack Selection

Use AskUserQuestion:
- **Question:** "Which tech stack would you like to use?"
- **Header:** "Tech Stack"
- **Options:**
  1. "JavaScript (Vite + TypeScript + React 19 + TailwindCSS 4)" — Description: "Modern JS stack with fast HMR, type safety, and utility-first CSS. SQLite for database."
  2. "Elixir (Phoenix + LiveView)" — Description: "Full-stack Elixir with real-time server-rendered UI. SQLite for database."
- **multiSelect:** false

Store the chosen stack for later use.

### Step 2: App Idea

Use AskUserQuestion:
- **Question:** "Describe your app idea. What should it do? Who is it for? What problem does it solve?"
- **Header:** "App Idea"
- **Options:**
  1. "I'll describe it" — Description: "I'll type out my app idea in the next message"
  2. "Help me brainstorm" — Description: "I have a vague idea and need help refining it"
- **multiSelect:** false

If "I'll describe it": Wait for user's next message with the description.
If "Help me brainstorm": Ask follow-up questions to help them refine their idea.

### Step 3: Scope Analysis

After receiving the app description, use extended thinking to analyze it:

**Think about:**
- What is the single most important user flow?
- What are the minimum screens needed to demonstrate this flow?
- What features can be deferred without losing the app's core value?
- What data models are needed for just the core flow?

**Then propose a bounded scope:**

Present the user with your understanding using AskUserQuestion:
- **Question:** Show a formatted summary:
  ```
  Here's what I propose for your MVP:

  Core Flow: [1-sentence description of the primary user journey]

  Screens ({{N}}):
  1. [Screen name] — [what it does]
  2. [Screen name] — [what it does]
  3. [Screen name] — [what it does]

  Key Features:
  - [Feature 1]
  - [Feature 2]
  - [Feature 3]

  Deferred to later:
  - [Feature X] — not needed to demonstrate core value
  - [Feature Y] — adds complexity without MVP benefit

  Does this scope look right?
  ```
- **Header:** "Scope"
- **Options:**
  1. "Looks good, proceed" — Description: "This scope captures what I want for the prototype"
  2. "I want to adjust" — Description: "I'd like to add or change something"
- **multiSelect:** false

If "I want to adjust": Ask what they'd like to change, re-analyze, and re-present. Loop at most 3 times, then proceed with whatever was last confirmed.

**Scope limits (enforce these):**
- Maximum 5 screens
- 1 core user flow
- No authentication unless it IS the core feature
- No payment integration
- No third-party API integrations (use mock data instead)
- No admin panel
- No settings page
- Data can be seeded/hardcoded if data entry isn't the core flow

### Step 4: Brainstorm Filename

Use AskUserQuestion:
- **Question:** "What should the brainstorm document be called?"
- **Header:** "Filename"
- **Options:**
  1. "brainstorm.md (Recommended)" — Description: "Default name, stored at .mvp/brainstorm.md"
  2. "Custom name" — Description: "Choose a custom filename"
- **multiSelect:** false

If "Custom name": Ask for the filename in a follow-up. Ensure it ends with `.md`.

Store the chosen filename.

---

## Phase 3: Prerequisite Checks

Check that the required tools are installed for the chosen stack.

### For JavaScript Stack:

Run these checks (use Bash tool for each):

```bash
node --version
npm --version
npx --version
```

Parse the output and show a checklist:
```
Prerequisite Check — JavaScript Stack:

  [x] node (v20.11.0) — requires >= 18
  [x] npm (v10.2.4)
  [x] npx (v10.2.4)

All prerequisites met!
```

Or if something is missing:
```
Prerequisite Check — JavaScript Stack:

  [x] node (v20.11.0) — requires >= 18
  [ ] npm — NOT FOUND
  [x] npx (v10.2.4)

Missing prerequisites! Install before continuing:
  npm: Included with Node.js — reinstall from https://nodejs.org
       Or use nvm: https://github.com/nvm-sh/nvm
```

**STOP if `node` or `npm` is missing.** These are critical. Show install instructions and exit.

### For Elixir Stack:

Run these checks:

```bash
elixir --version
mix --version
mix hex.info 2>&1 || echo "HEX_NOT_INSTALLED"
```

Parse and show checklist:
```
Prerequisite Check — Elixir Stack:

  [x] elixir (1.16.0) — requires >= 1.15
  [x] mix (1.16.0)
  [x] hex (2.0.0)

All prerequisites met!
```

If hex is missing, it can be auto-installed: `mix local.hex --force`

**STOP if `elixir` or `mix` is missing.** Show:
```
Missing prerequisites! Install before continuing:
  Elixir: https://elixir-lang.org/install.html
          Or use asdf: asdf install elixir latest
```

---

## Phase 4: Create .mvp/ Directory

Create the directory structure:

```bash
mkdir -p .mvp/agent-logs .mvp/research .mvp/resources
```

---

## Phase 5: Scaffold the Project

### Determine Project Slug

From the app name, create a kebab-case slug:
- "Task Tracker Pro" -> "task-tracker-pro"
- "Recipe Finder" -> "recipe-finder"
- Use this slug as the project directory name

### For JavaScript Stack:

Run these commands sequentially:

```bash
# Create Vite project with React TypeScript template
npm create vite@latest [slug] -- --template react-ts

# Install dependencies
cd [slug] && npm install

# Install additional MVP dependencies
npm install react-router-dom
npm install -D tailwindcss @tailwindcss/vite

# Install SQLite dependencies
npm install better-sqlite3 drizzle-orm
npm install -D drizzle-kit @types/better-sqlite3
```

After installation, set up TailwindCSS 4.x by updating the CSS entry point:

1. Read `[slug]/src/index.css` (or create it)
2. Add `@import "tailwindcss";` at the top
3. Read `[slug]/vite.config.ts` and add the Tailwind Vite plugin:
   ```typescript
   import tailwindcss from '@tailwindcss/vite'

   export default defineConfig({
     plugins: [react(), tailwindcss()],
   })
   ```

Verify the dev server starts:
```bash
cd [slug] && npm run dev -- --port 5173 &
DEV_PID=$!
sleep 3
curl -s -o /dev/null -w "%{http_code}" http://localhost:5173
kill $DEV_PID 2>/dev/null
```

If the curl returns 200, scaffolding succeeded. If not, troubleshoot.

### For Elixir Stack:

Run these commands sequentially:

```bash
# Ensure Phoenix generator is installed
mix archive.install hex phx_new --force

# Create Phoenix project with LiveView and SQLite
mix phx.new [slug] --live --no-mailer --no-dashboard --database sqlite3

# Install dependencies
cd [slug] && mix deps.get

# Create the database
mix ecto.create

# Verify compilation
mix compile
```

Verify the server starts:
```bash
cd [slug] && mix phx.server &
SERVER_PID=$!
sleep 5
curl -s -o /dev/null -w "%{http_code}" http://localhost:4000
kill $SERVER_PID 2>/dev/null
```

---

## Phase 6: Initialize Git

If the project directory is not already a git repo:

```bash
cd [slug] && git init && git add -A && git commit -m "mvp: initialize [project-name] with [stack]"
```

If it IS already a git repo (e.g., Vite or Phoenix created one), just ensure the initial state is committed.

---

## Phase 7: Write Brainstorm Document

1. Read the brainstorm template: `${CLAUDE_PLUGIN_ROOT}/templates/brainstorm-template.md`
2. Replace ALL `{{PLACEHOLDER}}` values with the actual data captured during brainstorming
3. Write the file to `.mvp/[chosen-filename]`

**CRITICAL**: Fill in every placeholder with actual content from the brainstorming conversation. Do NOT leave any `{{PLACEHOLDER}}` markers in the final document.

For the implementation phase tasks:
- Count all checkbox items and set `{{TOTAL_TASKS}}`
- Make tasks specific to the user's app (not generic)
- Phase 3 tasks should directly map to the core feature identified in brainstorming

---

## Phase 8: Write state.json

Create `.mvp/state.json` with the initial state:

```json
{
  "version": "1.0.0",
  "project": {
    "name": "[App Name]",
    "slug": "[slug]",
    "description": "[One-line description]",
    "stack": "js|elixir",
    "brainstormFile": "[chosen-filename]",
    "projectDir": "./[slug]",
    "createdAt": "[ISO timestamp]",
    "updatedAt": "[ISO timestamp]"
  },
  "status": "building",
  "currentPhase": 1,
  "resumePoint": {
    "phaseId": "scaffold",
    "taskId": null,
    "notes": "Ready to begin Phase 1 tasks",
    "lastCompletedAt": null
  },
  "phases": [
    ... populate from brainstorm phases with task IDs ...
  ],
  "analytics": {
    "totalTasks": [count],
    "completedTasks": 0,
    "failedTasks": 0,
    "agentSpawns": { "total": 0, "successful": 0, "failed": 0 },
    "qualityReviews": { "total": 0, "passed": 0, "failed": 0 },
    "gitCommits": 1,
    "sessionLog": [
      {
        "sessionId": "start-1",
        "mode": "start",
        "startedAt": "[ISO timestamp]",
        "endedAt": "[ISO timestamp]",
        "tasksCompleted": 0
      }
    ]
  },
  "processes": {
    "devServer": { "pid": null, "port": null }
  },
  "locks": {
    "migrations": false,
    "design": false,
    "dependencies": false
  }
}
```

**CRITICAL**: Populate the `phases` array by reading the brainstorm document's Implementation Phases section. Each checkbox item becomes a task entry with:
- `id`: unique identifier like `scaffold-1`, `data-2`, `core-3`
- `name`: the task description from the checkbox
- `status`: "pending"
- `mainAgentOnly`: true for tasks involving dependencies, migrations, server management
- Other fields: null/empty initial values

---

## Phase 9: Show Completion Summary

Display:

```
MVP project initialized!

  Project: [App Name]
  Stack: [Stack Name]
  Directory: ./[slug]
  Brainstorm: .mvp/[filename]
  State: .mvp/state.json

  Phases: 5
  Total tasks: [N]

  What was created:
    [x] .mvp/ directory with brainstorm doc and state file
    [x] [Stack-specific] project scaffolded in ./[slug]
    [x] Dev server verified working
    [x] Git repo initialized with initial commit

  Next steps:
    1. Review the brainstorm doc: .mvp/[filename]
    2. Start building: /mvp build
    3. Check progress anytime: /mvp status
```

---

## Execute Now

Begin with Phase 1 (check for existing state), then proceed through each phase sequentially.
