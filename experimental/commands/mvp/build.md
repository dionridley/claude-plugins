# /mvp build — Autonomous Build Orchestrator

This file is read by the main `mvp.md` router when the user runs `/mvp build`.

## Instructions for Claude

You are executing the BUILD mode of the `/mvp` command. Your job is to:
1. Load state, read conventions for the chosen stack
2. Orchestrate subagents to build the prototype phase by phase
3. Enforce testing, design quality, and browser testing gates
4. Maintain accurate state and brainstorm document throughout

---

## Phase 1: Load State and Conventions

### 1a. Load project state

1. **Read `.mvp/state.json`:**
   - If missing: Show error and STOP — "No MVP project found. Run /mvp start to begin."
   - If `status == "complete"`: Show "MVP already complete. Run /mvp summary." and STOP
   - If `status == "error"`: Show `resumePoint.notes` and ask user how to proceed

2. **Read `.mvp/[state.project.brainstormFile]`**
   - If missing: Show error and STOP

3. **Sweep stale subagent PIDs from previous sessions:**

   For each entry in `state.processes.subagentPids` where `status == "running"`:

   **Step 1 — verify the process is still ours before touching it:**
   ```bash
   ACTUAL=$(ps -p [pid] -o args= 2>/dev/null)
   ```
   Compare `$ACTUAL` against the stored `command` fragment. Only proceed if the actual running command contains the expected fragment (e.g. stored `"mix phx.server"` → check `$ACTUAL` contains `phx.server`; stored `"vite"` → check `$ACTUAL` contains `vite`).

   - **If command does NOT match** (PID was recycled to an unrelated process):
     - Update entry: `status: "recycled"`, `note: "PID recycled — skipped kill. Actual: [ACTUAL]"`
     - Do NOT kill. Log a warning but continue.

   - **If command DOES match** (process still ours):
     ```bash
     kill [pid] 2>/dev/null
     sleep 1
     ps -p [pid] > /dev/null 2>&1 || echo "confirmed stopped"
     ```
     Update entry: `status: "stopped"`, `stoppedAt: now`, `note: "killed on session start — stale from prior session"`

   Write `state.json` after the full sweep.

4. **Record this session** in `analytics.sessionLog`:
   ```json
   {
     "sessionId": "build-[N]",
     "mode": "build",
     "startedAt": "[current ISO timestamp]",
     "endedAt": null,
     "tasksCompleted": 0
   }
   ```
   Write updated `state.json`.

### 1b. Load stack conventions

Based on `state.project.stack`:

- **Elixir:** Read `${CLAUDE_PLUGIN_ROOT}/commands/mvp/conventions/elixir.md`
- **JS:** Read `${CLAUDE_PLUGIN_ROOT}/commands/mvp/conventions/typescript.md`

Store the **"Mandatory Rules for Every Agent"** block from that file. You will inject it into EVERY subagent prompt for the remainder of this session under a `## [Stack] Conventions — MANDATORY` section.

---

## Phase 2: Ensure Dev Server

The main agent (you) handles dev server concerns — never delegate this to subagents.

Read `state.project.serverManagement` and follow the appropriate path:

---

### If `serverManagement == "user"` (User-managed)

Show this prompt to the user and wait:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Start your dev server
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Please start the dev server in a separate terminal:

  Elixir:     mix phx.server
  JavaScript: npm run dev  (in one terminal)
              npm run server  (in a second terminal for Express)

  Press Enter / reply "ready" when the server is running.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Once the user confirms, verify with curl:
```bash
/usr/bin/curl -s -o /dev/null -w "%{http_code}" http://localhost:[state.project.port] 2>/dev/null
```

If not responding: ask the user to check the server output. Do NOT attempt to start it yourself.

**When a server restart is needed** (config change, new dependency, migration):
Show this message and wait for confirmation:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Server restart required
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Reason: [e.g. "new dependency added to mix.exs" /
           "config/dev.exs changed" / "migration run"]

  Please restart the server in your terminal (Ctrl+C, then
  re-run the start command), then reply "ready".
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### If `serverManagement == "agent"` (Agent-managed)

**Show this warning before starting any processes:**

For Elixir stack:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Process safety notice — Elixir stack
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  This build will start and stop `mix phx.server` processes.
  Process identity is verified by command name before any kill.

  However, if you have OTHER Phoenix applications running
  locally (from other projects), they share the same process
  name. If a PID is recycled to one of those processes it
  could be affected.

  Recommendation: stop other Phoenix servers before continuing.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

For JavaScript stack:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Process safety notice — JavaScript stack
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  This build will start and stop `vite` and `tsx` processes.
  Process identity is verified by command name before any kill.

  However, if you have OTHER Vite or Express/tsx applications
  running locally (from other projects), they share the same
  process names. If a PID is recycled to one of those
  processes it could be affected.

  Recommendation: stop other Vite/Node servers before continuing.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

1. **Kill previously stored dev server PID only** (never kill by port):
   If `state.processes.devServer.pid` is set from a prior session, verify and kill:
   ```bash
   ACTUAL=$(ps -p [stored-pid] -o args= 2>/dev/null)
   # Elixir: check contains "phx.server"
   # JS: check contains "vite" or "tsx"
   if echo "$ACTUAL" | grep -q "[expected-fragment]"; then
     kill [stored-pid] 2>/dev/null && sleep 1
   else
     echo "Stored PID [pid] is '$ACTUAL' — not ours, skipping"
   fi
   ```

2. **Start the server:**
   ```bash
   # Elixir
   PORT=[port] mix phx.server &
   DEV_PID=$!

   # JS — start both Vite and Express
   npm run dev -- --port [port] &
   VITE_PID=$!
   npm run server &
   EXPRESS_PID=$!
   ```
   Store PIDs and their command fragments in `state.processes.devServer` and `state.processes.expressServer`:
   ```json
   { "pid": 12345, "port": 4000, "command": "mix phx.server" }
   ```
   Retry the curl check up to 5 times (2s sleep between attempts).

3. **If server fails to start:** Set `status: "error"`, update `resumePoint.notes`, write `state.json`, and STOP.

**When a server restart is needed** (config change, new dependency, migration):
Verify the stored PID is still ours, then kill and restart:
```bash
ACTUAL=$(ps -p [stored-pid] -o args= 2>/dev/null)
if echo "$ACTUAL" | grep -q "[expected-fragment]"; then
  kill [stored-pid] 2>/dev/null && sleep 2
fi
# re-run start command, capture new PID, update state.json
```

---

## Phase 3: Determine Current Work

1. **Find current phase** from `state.currentPhase` (the index into `state.phases[]`)
2. **If phase status is "completed":** advance `currentPhase`, write `state.json`
3. **If all phases complete:** go to Phase 11 (Completion)
4. **If phase status is "pending":** set to "in_progress", record `startedAt`, write `state.json`

### User checkpoints at phase transitions

At the start of each new phase (except Phase 1 at session start), show:
```
Phase [N-1] — [Phase Name] — COMPLETE

Progress: [X]/[Total] tasks  |  Agents: [N]  |  Elapsed: [time]

Starting Phase [N]: [Phase Name]
[M] tasks to complete.
```

Use AskUserQuestion:
- "Continue to Phase [N]?" with options: "Continue" / "Pause here"
- If "Pause here": close session log (`endedAt`, `tasksCompleted`), update `resumePoint`, write `state.json`, STOP

---

## Phase 4: Phase-Specific Execution Logic

Each phase has special handling. Identify the current phase by its `id` field and follow the corresponding section below.

---

### Phase: `scaffold` — Project Scaffold & Foundation

This phase is handled almost entirely by the main agent (all tasks are `mainAgentOnly`). Agents are not typically needed here — the scaffold was done in `/mvp start`. This phase's tasks verify or clean up anything left.

**Gate:** Dev server starts, assets compile, base route renders 200. For Elixir: `mix compile` produces no errors.

---

### Phase: `data` — Data Layer & Schema

**Main agent handles all migration work** (never delegate).

**Writing migrations directly (do NOT use `mix ecto.gen.migration`):**
```bash
# Generate a timestamp
date -u +"%Y%m%d%H%M%S"
```
Write migration files directly to `priv/repo/migrations/[timestamp]_create_[table].exs` using the Write tool. This is faster than generating empty files and editing them.

**Delegatable tasks:** Schema module definitions, seed data file creation.

For **seed data agents** (Elixir), include in the prompt:
- Faker is available: `Faker.Person.name()`, `Faker.Internet.email()`, `Faker.Lorem.paragraph(2..3)`
- Make seeds idempotent (delete-then-insert)
- Create enough data to make every screen feel populated on first load

**Gate:** `mix ecto.migrate` / `drizzle-kit push` runs clean, `mix run priv/repo/seeds.exs` / seed script runs clean.

---

### Phase: `tests` — Test Scaffolding

**This phase is mandatory and non-skippable.**

**Goal:** Write unit tests for every context/data module before any UI is built. Having tests at this stage means UI agents can confidently call context functions without wondering if they work.

**Delegatable tasks:**
- Create `test/support/factory.ex` (Elixir) or `src/lib/db/*.test.ts` (JS) with factory/data helpers
- Write context unit tests (one agent per context module)

**Agent prompt additions for test agents:**
- Include the actual schema field names and changeset validations from the schema module
- Include expected success and failure cases
- For Elixir: read `test/support/data_case.ex` before writing tests — know what helpers are available

**Quality review:** Always run quality review on test agents (tests are code, they can have bugs).

**Gate — MANDATORY:**
```bash
# Elixir
cd [projectDir] && mix test

# JS
cd [projectDir] && npm test
```

**`mix test` / `npm test` MUST pass with 0 failures before this phase is marked complete.** If tests fail:
1. Read the failure output
2. Fix the failing tests (main agent or new agent)
3. Re-run until passing
4. Do NOT advance to the next phase until 0 failures

---

### Phase: `design` — Design Brief

**This phase is mandatory and non-skippable.**

The design brief establishes the visual identity for all screen agents in Phase `core` and `polish`. Without it, agents produce generic, indistinct UI.

**Steps (main agent executes all of this):**

1. Read the brainstorm document. Extract:
   - App name and vision
   - The 3-5 screens
   - The core user flow
   - The target audience

2. Generate a design brief for this specific app. Think deeply about:
   - What visual tone fits this app and its users? (clinical, playful, professional, editorial, etc.)
   - What color palette serves the purpose? (don't default to blues and grays)
   - What typography style communicates the brand?
   - What card/component patterns will be used repeatedly?
   - What should the empty states look and feel like?
   - What micro-interactions would make it feel real? (hover states, transitions)

   **Avoid these generic AI defaults:**
   - Purple/blue gradient headers
   - Generic "Inter" font with no personality
   - Cookie-cutter card borders with box-shadow
   - Hero sections with "Get Started" CTAs
   - Default DaisyUI theme with no customization

3. Write the design brief to `.mvp/research/design-brief.md`:

```markdown
# Design Brief: [App Name]

## Visual Tone
[2-3 sentences describing the intended feel and why it fits this app and audience]

## Color Palette
- Primary: [color + hex] — [why this color]
- Secondary: [color + hex]
- Accent: [color + hex]
- Background: [color + hex]
- Surface/Card: [color + hex]
- Text: [color + hex]
- DaisyUI theme name or custom theme: [theme config if needed]

## Typography
- Headings: [font choice or system font] — [weight, tracking]
- Body: [font or system stack]
- Monospace: (if needed)

## Component Patterns
- Cards: [description — e.g. "white bg, rounded-xl, subtle shadow on hover, no visible border"]
- Buttons: [description — e.g. "solid primary with slight scale on hover, pill-shaped for CTAs"]
- Inputs: [description]
- Navigation: [description]
- Tags/Badges: [description]

## Empty States
[Description of empty state pattern — e.g. "centered with illustrative icon, muted text, one clear CTA"]

## Screen-Specific Notes
### [Screen 1 name]
[Any specific design notes for this screen]

### [Screen 2 name]
[Any specific design notes]
```

4. Update `resumePoint.notes` with the design brief path.
5. Write `state.json` with phase `startedAt` and the task marked complete.

**Gate:** `.mvp/research/design-brief.md` exists and is populated.

---

### Phase: `core` — Core Feature Implementation

**This is the most parallelizable phase.**

Before dispatching any agents:
1. Read `.mvp/research/design-brief.md` — you will include an excerpt in every agent prompt
2. Read the core feature tasks from the brainstorm document
3. For Elixir: read all context module files in `lib/[slug]/` — extract function signatures to include in agent prompts

**Agent dispatch rules:**
- Maximum 3 concurrent agents
- Never two agents touching the same file
- Never two agents implementing the same screen/LiveView
- Agents writing LiveViews that call context functions MUST receive the actual function signatures (not just descriptions)

**Recommended parallelization (Elixir):**

| Batch | Agents |
|-------|--------|
| Batch 1 | Up to 3 LiveView screens in parallel — each gets isolated files |
| Batch 2 | Remaining screens + any shared components |

**For every agent in this phase, include this section in the prompt:**

```
## Design System
[Paste the Color Palette, Component Patterns, and any screen-specific note
 from .mvp/research/design-brief.md relevant to this agent's screen]

## Context Functions Available
[For Elixir agents: paste the actual function signatures from the relevant
 context module(s). Example:]
  MyApp.Things.list_things/0 → [%Thing{id, name, status}]
  MyApp.Things.create_thing/1 → {:ok, %Thing{}} | {:error, changeset}
  MyApp.Things.get_thing!/1   → %Thing{} | raises Ecto.NoResultsError
```

**Playwright browser tests (if `state.project.playwrightEnabled == true`):**

After each agent completes and passes quality review, dispatch a browser test agent:

```
You are a Browser Test Agent for MVP: [App Name]
Stack: [stack]
Server is running at: http://localhost:[port]

## Screen to Test
[Screen name and description]

## Test Script
Navigate to [route] and verify:
1. [Assertion 1 — e.g. "heading 'My Forms' is visible"]
2. [Assertion 2 — e.g. "'New Form' button is visible and clickable"]
3. [Core interaction — e.g. "Fill in the title field, click Save, verify redirect"]
4. [Another interaction]

Take a screenshot after each major step.

## On Failure
If any step fails, report:
- Which step failed
- What was expected vs what was seen
- Screenshot path if available
- Your best diagnosis of the cause

## Expected Output — MANDATORY FORMAT
```json
{
  "screen": "[screen name]",
  "verdict": "PASS|FAIL",
  "steps": [
    {"step": "description", "result": "pass|fail", "notes": "..."}
  ],
  "screenshotPaths": [],
  "diagnosis": "only if FAIL"
}
```
```

If browser test returns FAIL:
- Update `analytics.browserTests.failed`
- Add a fix task to the current phase
- Dispatch an implementation agent with the failure context and diagnosis
- Re-test after the fix

If Playwright MCP is not available (tools not present):
- Log: "Playwright MCP not available — falling back to HTTP check only"
- Continue without browser testing

---

### Phase: `polish` — UI Polish & Remaining Screens

Parallelizable by screen (each screen is independent).

- Include design brief excerpt in each agent prompt
- One agent per remaining screen
- One additional agent for global responsive/accessibility polish if needed
- Do NOT run browser tests for every polish task — reserve for screens with significant changes

**Gate:** All screens render, all routes navigable, no broken imports.

---

### Phase: `integration` — Integration & Smoke Test

**This phase is mandatory. Tests must pass before completion.**

**Main agent tasks (not delegated):**
1. Run the full test suite and fix any failures:
   ```bash
   # Elixir
   cd [projectDir] && mix test

   # JS
   cd [projectDir] && npm test
   ```
   If failures exist, fix them (you or a targeted agent) and re-run. Do NOT skip this.

2. Attempt a production build to catch any remaining issues:
   ```bash
   # JS only
   cd [projectDir] && npm run build
   ```

**Delegatable tasks (one agent each):**
- Write LiveView interaction tests / React component tests for each screen
- Write README with accurate setup and run instructions
- Clean up boilerplate, remove unused imports

**Agent prompts for test-writing agents:**
- Include the actual routes and LiveView/component names
- Include example interaction patterns from the brainstorm's core user flow
- For Elixir test agents: include the factory helpers from `test/support/factory.ex`

**Gate — MANDATORY:**
`mix test` / `npm test` must pass with **0 failures** before this phase is marked complete.
If tests fail, they must be fixed before the final commit is made.

---

## Phase 5: The Orchestration Loop

For each phase (following the phase-specific logic above), repeat this loop:

### Step A: Main-agent-only tasks first

Execute directly. After each:
1. Mark task `status: "completed"`, set `completedAt`
2. Check checkbox in brainstorm.md
3. Git commit:
   ```bash
   cd [projectDir] && git add -A && git commit -m "mvp: [task description]"
   ```
4. Increment `analytics.completedTasks`, `analytics.gitCommits`
5. Write `state.json` immediately

### Step B: Dispatch subagent batch

For each delegatable task:

1. **Acquire any needed locks** in `state.json`
2. **Mark task as in_progress**, set `startedAt`, write `state.json`
3. **Build agent instructions:**

   ```
   You are Agent #[ID] working on MVP: [App Name]
   Stack: [JavaScript/Elixir]
   Project directory: [absolute path]

   ## Your Task
   [Specific task from brainstorm]

   ## App Context
   - Vision: [1-2 sentences from brainstorm vision]
   - Current phase: Phase [N] — [Phase Name]
   - Related screens: [relevant screen descriptions]

   ## What Already Exists
   [Read the relevant existing files BEFORE dispatching and paste key snippets:
    - For LiveView agents: paste context function signatures
    - For component agents: paste the base layout structure
    - For test agents: paste the factory helpers and existing context functions]

   ## Design System
   [Excerpt from .mvp/research/design-brief.md relevant to this screen]

   ## [Stack] Conventions — MANDATORY
   [Full conventions block from the loaded conventions file]

   ## Constraints
   - ONLY modify files directly related to your task
   - Do NOT install dependencies — list needed ones in your result
   - Do NOT run migrations
   - Do NOT start or stop the dev server
   - Do NOT modify .mvp/ files
   - Do NOT modify package.json or mix.exs
   - ALWAYS read a file before modifying it
   - If you must start a background process (e.g. a test runner or watcher):
     - Record its PID immediately after starting: `echo $!`
     - Stop it before returning: `kill [pid] 2>/dev/null && sleep 1`
     - Confirm it is stopped: `ps -p [pid] > /dev/null 2>&1 || echo "confirmed stopped"`
     - Report it in `processesStarted` with `stopped: true`
     - NEVER leave a background process running when you return

   ## Expected Output — MANDATORY JSON FORMAT
   ```json
   {
     "agentId": [ID],
     "status": "success|partial|failed",
     "taskCompleted": "description of what was done",
     "filesModified": ["relative/path"],
     "filesCreated": ["relative/path"],
     "dependenciesNeeded": [],
     "processesStarted": [
       {
         "pid": 12345,
         "command": "human-readable description",
         "stopped": true,
         "note": "started for X, stopped after Y"
       }
     ],
     "issues": [],
     "notes": "any context for the main agent"
   }
   ```
   ```

4. **Choose isolation:**
   - `isolation: "worktree"` for any agent creating or modifying more than 2 files (default for most agents)
   - No isolation for single-file low-risk tasks only

5. **Launch parallel agents** in a single message using multiple Task tool calls when tasks are independent

### Step C: Process completions

As each agent returns:

1. Parse JSON result
2. **Handle `dependenciesNeeded`:** install them (main agent only), then run `mix deps.get` or `npm install`
3. **Handle `processesStarted`:** for each entry in the array:
   - Append to `state.processes.subagentPids`:
     ```json
     {
       "pid": [pid],
       "command": "[command]",
       "taskId": "[task id]",
       "agentId": [agent id],
       "startedAt": "[agent startedAt]",
       "status": "running",
       "stoppedAt": null
     }
     ```
   - Check if the agent already stopped it (`stopped: true`):
     - If yes: verify with `ps -p [pid] > /dev/null 2>&1 || echo "gone"`, update status to `"stopped"`, set `stoppedAt: now`
     - If no (agent failed to stop it): verify command then kill:
       ```bash
       ACTUAL=$(ps -p [pid] -o args= 2>/dev/null)
       # verify ACTUAL contains the stored command fragment before killing
       ```
       If command matches: kill, update status to `"stopped"`, note `"force-killed by main agent — subagent did not stop"`
       If command does NOT match: update status to `"recycled"`, note `"PID recycled before cleanup — actual: [ACTUAL]"`, do NOT kill
   - Write `state.json` after processing all PIDs
4. **Release locks**
5. Update task in `state.json`:
   - `status`: from result
   - `completedAt`: now
   - `agentSpawn.completedAt`: now, `result`: from status
   - `filesChanged`: merged files list
6. Increment `analytics.agentSpawns.total` and `.successful` or `.failed`
7. Write `state.json` immediately

### Step D: Quality review

**Skip quality review for low-risk tasks** (`task.lowRisk == true`):
- README writing
- CSS-only appends
- File deletions
- Boilerplate cleanup
- Simple stub replacements

Mark these as skipped: `qualityReview: { skipped: true, reason: "low-risk task" }`
Increment `analytics.qualityReviews.skipped`.

**For all other completed tasks** (status "success" or "partial"), dispatch a review agent:

```
You are a Quality Review Agent for MVP: [App Name]

## Original Task
[task description]

## Agent's Report
[agent JSON result]

## Review Instructions
1. Verify every file in filesModified and filesCreated EXISTS
2. READ each file:
   - Syntax errors or compile issues?
   - Follows [Elixir/TypeScript] conventions?
   - Obvious bugs, missing imports, undefined references?
   - Does it accomplish the stated task?
3. Check integration:
   - Imports resolve correctly?
   - Exports match what callers expect?
4. Run compile check:
   - Elixir: `mix compile` (check for errors/warnings)
   - JS: look for TypeScript type errors in the modified files

## Expected Output — MANDATORY JSON
```json
{
  "reviewOf": "[task id]",
  "verdict": "PASS|FAIL",
  "issues": [
    {"severity": "critical|warning|info", "file": "path", "description": "what's wrong"}
  ],
  "suggestions": [],
  "notes": "overall assessment"
}
```

PASS = code compiles, follows conventions, accomplishes the task.
FAIL = critical issues that must be fixed first.
```

**If PASS:**
- Update `task.qualityReview`: `{ passed: true, reviewedAt: now, notes: "..." }`
- Check off task in brainstorm.md
- Git commit the agent's files:
  ```bash
  cd [projectDir] && git add [specific files] && git commit -m "mvp: [task description]"
  ```
- Increment `analytics.qualityReviews.passed`, `analytics.completedTasks`, `analytics.gitCommits`
- Write `state.json`

**If FAIL:**
- Update `task.qualityReview`: `{ passed: false, retryCount: N, reviewedAt: now, issues: [...] }`
- If `retryCount < 3`: set task back to "pending", update `resumePoint.notes` with failure context
- If `retryCount >= 3`: set task to "failed", increment `analytics.failedTasks`, warn user and skip
- Write `state.json`

### Step E: Update brainstorm document

After each agent batch:
1. Update Task Tracking section (recalculate counts)
2. Update Analytics table
3. Append to Agent Log for each agent that ran:
   ```markdown
   ### Agent #[ID] — [Task Name]
   - **Phase:** [phase name]
   - **Status:** SUCCESS / PARTIAL / FAILED
   - **Started:** [timestamp]
   - **Completed:** [timestamp]
   - **Duration:** [calculated]
   - **Files:** [list]
   - **Quality Review:** PASS / FAIL / SKIPPED
   - **Browser Test:** PASS / FAIL / N/A
   - **Notes:** [agent + review notes]
   ```

### Step F: Check phase completion and gate

When all tasks in the current phase are completed or failed:

1. **Sweep subagent PIDs for this phase** — final safety check:
   - Filter `state.processes.subagentPids` where `taskId` belongs to the current phase and `status == "running"`
   - For each: verify command matches stored fragment before killing:
     ```bash
     ACTUAL=$(ps -p [pid] -o args= 2>/dev/null)
     # only kill if ACTUAL contains stored command fragment
     ```
   - If matches: kill, mark `"stopped"`. If not: mark `"recycled"`, skip kill.
   - Write `state.json`

2. Run the phase gate check (see Phase 4 for gate definitions)
3. If gate fails: attempt fix, or mark as error and ask user
4. If gate passes:
   - Set phase `status: "completed"`, set `completedAt`
   - Calculate actual time
   - Git commit (phase boundary):
     ```bash
     git add -A && git commit -m "mvp: complete phase [id] — [phase name]"
     ```
   - Advance `currentPhase`
   - Write `state.json`

### Step G: Session checkpoint

After completing a full phase, check if context is getting large. If so:
```
Session checkpoint — Phase [N] complete.

Progress: [X]/[Total] tasks  |  [N] agents  |  [time] elapsed
```

Close session log entry (`endedAt`, `tasksCompleted`). Tell user: "Run /mvp build to continue."

---

## Phase 6: Error Recovery

1. **Agent returns "failed":** Retry up to 3 times with error context added to prompt. After 3 failures, skip and continue.
2. **Dev server crashes:** Detect via failed curl. Restart (main agent). Continue.
3. **Dependency install fails:** Try alternative package name. If still fails, mark dependent tasks blocked.
4. **Test failures at phase gate:** Fix failures before advancing — do not skip the gate.
5. **All tasks in phase failed/blocked:** Set `status: "error"`, update `resumePoint.notes`, STOP and ask user.

---

## Phase 7: Completion

When all phases complete:

1. **Final process sweep — kill everything:**

   Kill the dev server:
   ```bash
   kill [state.processes.devServer.pid] 2>/dev/null
   kill [state.processes.expressServer.pid] 2>/dev/null
   ```

   Kill any remaining subagent PIDs still marked `"running"`, with command verification:
   ```bash
   # For each pid in state.processes.subagentPids where status == "running":
   ACTUAL=$(ps -p [pid] -o args= 2>/dev/null)
   if echo "$ACTUAL" | grep -q "[stored-command-fragment]"; then
     kill [pid] 2>/dev/null
     # mark "stopped"
   else
     # mark "recycled", skip kill
   fi
   ```

   Also verify and kill dev/express server PIDs the same way — check command matches before killing.
   Write `state.json`.

2. **Final test run:**
   ```bash
   cd [projectDir] && mix test   # or npm test
   ```
   Record result in analytics.

3. **Final git commit:**
   ```bash
   git add -A && git commit -m "mvp: [project-name] prototype complete"
   ```

4. **Update state:** `status: "complete"`, close session log, write `state.json`

5. **Update brainstorm.md:** Set Status to `complete`, fill in "Build completed" timestamp, calculate total elapsed time.

6. **Show completion message:**
   ```
   MVP COMPLETE: [App Name]

   Stack:            [Stack Name]
   Phases:           7/7 complete
   Tasks:            [N]/[Total] ([failed] failed/skipped)
   Agents:           [N] dispatched ([success] success, [fail] failed)
   Quality reviews:  [passed] passed, [failed] failed, [skipped] skipped
   Browser tests:    [passed] passed, [failed] failed (or "disabled")
   Git commits:      [N]
   Total time:       [calculated across sessions]

   To run the app:
     cd [projectDir]
     [npm run dev | PORT=[port] mix phx.server]
     Open http://localhost:[port]

   Artifacts:
     .mvp/brainstorm.md          — Build log and project docs
     .mvp/state.json             — Analytics and machine state
     .mvp/agent-logs/            — Individual agent reports
     .mvp/research/design-brief.md — Design system

   Generate analytics page:
     /mvp summary
   ```

---

## Execute Now

Begin with Phase 1 (Load State and Conventions), then proceed through the orchestration loop.
