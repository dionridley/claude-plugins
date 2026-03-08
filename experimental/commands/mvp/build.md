# /mvp build — Autonomous Build Orchestrator

This file is read by the main `mvp.md` router when the user runs `/mvp build`.

## Instructions for Claude

You are executing the BUILD mode of the `/mvp` command. Your job is to:
1. Read saved state and brainstorm document
2. Determine what work needs to be done next
3. Orchestrate subagents to build the prototype in parallel where safe
4. Run quality reviews on completed work
5. Update state and brainstorm document continuously

---

## Phase 1: Load and Validate State

1. **Read `.mvp/state.json`:**
   - If missing: Show error and STOP:
     ```
     No MVP project found in this directory.
     Run /mvp start to begin a new project.
     ```
   - If `status` is `"complete"`: Show "MVP already complete. Run /mvp summary to view analytics." and STOP
   - If `status` is `"error"`: Show the error context from `resumePoint.notes` and ask user how to proceed

2. **Read the brainstorm file** at `.mvp/[state.project.brainstormFile]`
   - If missing: Show error and STOP

3. **Record this session** in `analytics.sessionLog`:
   ```json
   {
     "sessionId": "build-[N]",
     "mode": "build",
     "startedAt": "[current ISO timestamp]",
     "endedAt": null,
     "tasksCompleted": 0
   }
   ```
   Write the updated `state.json`.

---

## Phase 2: Ensure Dev Server

The dev server must be running for testing. The main agent (you) handles this — never delegate server management to subagents.

1. **Check if dev server is already running:**
   ```bash
   # For JS stack
   curl -s -o /dev/null -w "%{http_code}" http://localhost:5173 2>/dev/null

   # For Elixir stack
   curl -s -o /dev/null -w "%{http_code}" http://localhost:4000 2>/dev/null
   ```

2. **If not running, start it:**
   ```bash
   # For JS stack
   cd [projectDir] && npm run dev -- --port 5173 &

   # For Elixir stack
   cd [projectDir] && mix phx.server &
   ```
   - Capture the PID
   - Store in `state.processes.devServer`: `{ "pid": [PID], "port": [PORT] }`
   - Wait for server to respond (retry curl up to 5 times with 2s sleep between)

3. **If server fails to start:** Set status to "error", update `resumePoint.notes` with the error, and STOP.

---

## Phase 3: Determine Current Work

1. **Identify current phase** from `state.currentPhase`
2. **Read that phase's data** from `state.phases[currentPhase - 1]`
3. **If phase status is "completed":**
   - Advance `currentPhase` by 1
   - If all phases complete, go to Phase 8 (Completion)
   - Otherwise, continue with the new current phase

4. **If phase status is "pending":**
   - Set it to "in_progress"
   - Record `startedAt` timestamp
   - Write `state.json`

5. **User checkpoint at phase boundaries:**
   - At the START of each new phase (except Phase 1 if just starting), show progress summary:
     ```
     Phase [N-1] complete! Moving to Phase [N]: [Phase Name]

     Progress so far:
       Completed: [X]/[Total] tasks
       Agents dispatched: [N]
       Quality reviews: [passed]/[total] passed
       Elapsed: [time]

     Phase [N] has [M] tasks.
     ```
   - Use AskUserQuestion:
     - "Continue to Phase [N]?" with options: "Continue" / "Pause here"
     - If "Pause here": Update `resumePoint`, end session log, and STOP

---

## Phase 4: Parse and Categorize Tasks

1. **Get uncompleted tasks** for the current phase from `state.phases[].tasks[]` where `status` is "pending" or "failed" (with retries < 3)

2. **Categorize each task:**

   **Main-agent-only tasks** (`mainAgentOnly: true`):
   - Any task involving `npm install`, `mix deps.get`, dependency changes
   - Any task involving database migrations (`mix ecto.migrate`, drizzle-kit push)
   - Any task involving dev server start/stop/restart
   - Any task modifying `package.json`, `mix.exs`, config files shared across the app

   **Delegatable tasks** (`mainAgentOnly: false`):
   - Component creation
   - Page/route implementation
   - Utility function creation
   - Styling work
   - Seed data creation
   - Test writing

3. **Identify parallelizable groups** among delegatable tasks:

   **NEVER run in parallel:**
   - Two tasks modifying the same file
   - Two tasks implementing the same screen/component
   - Two tasks both doing database work
   - Two tasks both doing global design/CSS changes
   - Two tasks for features that import from each other

   **SAFE to parallelize:**
   - Different screens that don't share custom components
   - Backend data layer + unrelated frontend component
   - Independent utility modules
   - Different pages that only share the base layout

   **Maximum 3 concurrent subagents.** Prefer 2 for safety in early phases.

---

## Phase 5: Execute Work — The Orchestration Loop

Loop through tasks until the current phase is complete:

### Step A: Handle main-agent-only tasks FIRST

For each pending main-agent-only task in the current phase:
1. Record `startedAt` timestamp on the task
2. Execute the task directly (you are the main agent)
3. Verify the result (e.g., migration ran, dependency installed)
4. Record `completedAt` timestamp and set `status: "completed"`
5. Update the corresponding checkbox in the brainstorm document: `- [ ]` -> `- [x]`
6. Git commit:
   ```bash
   cd [projectDir] && git add -A && git commit -m "mvp: [task description]"
   ```
7. Update `state.json` analytics: increment `completedTasks`, `gitCommits`
8. Write `state.json`

### Step B: Dispatch subagents for parallelizable work

For each batch of parallelizable tasks (max 3 per batch):

1. **Acquire locks** in `state.json` for any resources the tasks need
2. **Record agent spawn** in `state.json`:
   ```json
   {
     "id": "scaffold-1",
     "status": "in_progress",
     "startedAt": "[timestamp]",
     "agentSpawn": {
       "spawnedAt": "[timestamp]",
       "completedAt": null,
       "result": null
     }
   }
   ```
3. **Dispatch agents** using the Task tool. For each agent, provide these instructions:

   ```
   You are Agent #[ID] working on MVP: [App Name]
   Stack: [JavaScript/Elixir]
   Project directory: [absolute path to project]

   ## Your Task
   [Specific task description]

   ## Context
   - Current phase: Phase [N] - [Phase Name]
   - Related screens: [relevant screen descriptions from brainstorm]
   - App vision: [1-2 sentence vision from brainstorm]

   ## What Already Exists
   [List key existing files and their purpose that the agent needs to know about.
    Read these files before dispatching and include relevant snippets.]

   ## Constraints — READ CAREFULLY
   - ONLY modify files directly related to your task
   - Do NOT install new dependencies — if you need one, list it in your result
   - Do NOT run or modify database migrations
   - Do NOT start or stop the dev server
   - Do NOT modify .mvp/ directory files
   - Do NOT modify package.json, mix.exs, or config files
   - If you start any background processes, report their PIDs

   ## Stack Conventions
   [For JS stack:]
   - Use functional React components with TypeScript
   - Use React Router for navigation (import from 'react-router-dom')
   - Use TailwindCSS 4 utility classes for styling
   - Use proper TypeScript types (no `any`)

   [For Elixir stack:]
   - Use LiveView components (live views, live components)
   - Follow Phoenix conventions for contexts, schemas, live views
   - Use Tailwind for styling (included in Phoenix by default)
   - Keep business logic in context modules

   ## Expected Output — MANDATORY FORMAT
   When complete, respond with EXACTLY this JSON (no other text after it):

   ```json
   {
     "agentId": [ID],
     "status": "success|partial|failed",
     "taskCompleted": "description of what was done",
     "filesModified": ["relative/path/to/file1"],
     "filesCreated": ["relative/path/to/new-file"],
     "dependenciesNeeded": ["package-name"],
     "processesStarted": [],
     "issues": [],
     "notes": "any additional context"
   }
   ```
   ```

4. **Choose isolation mode:**
   - Use `isolation: "worktree"` for tasks that create/modify multiple files
   - Use no isolation for simple single-file tasks
   - NEVER use worktree isolation for tasks that need to read files created by other recent agents (the worktree won't have them)

5. **Launch agents in parallel** using multiple Task tool calls in a single message when tasks are independent.

### Step C: Process agent results

As each agent returns:

1. **Parse the structured JSON result** from the agent's output
   - Look for the JSON block in the response
   - If no valid JSON found, treat as a failed task

2. **Handle dependency requests:**
   - If `dependenciesNeeded` is non-empty, the main agent installs them:
     ```bash
     # JS
     cd [projectDir] && npm install [packages]
     # Elixir
     cd [projectDir] && mix deps.get
     ```
   - This is a main-agent-only operation

3. **Handle started processes:**
   - If `processesStarted` is non-empty, record PIDs in `state.json`
   - Consider whether these processes should be killed (agent cleanup)

4. **Release locks** that this agent's task held

5. **Update task in state.json:**
   ```json
   {
     "status": "[from agent result]",
     "completedAt": "[current timestamp]",
     "agentSpawn": {
       "spawnedAt": "[original time]",
       "completedAt": "[current timestamp]",
       "result": "[success|partial|failed]"
     },
     "filesChanged": ["[from agent filesModified + filesCreated]"]
   }
   ```

6. **Update analytics:**
   - Increment `agentSpawns.total`
   - Increment `agentSpawns.successful` or `agentSpawns.failed`

### Step D: Quality review

For EACH completed agent task (status "success" or "partial"), dispatch a quality review agent:

1. **Dispatch review agent** using Task tool:

   ```
   You are a Quality Review Agent reviewing work done on MVP: [App Name]

   ## Original Task
   [The task description that was assigned to the agent]

   ## Agent's Report
   [The structured JSON result from the agent]

   ## Review Instructions

   1. Verify each file in filesModified and filesCreated EXISTS
   2. READ each file and check:
      - Does the code compile/parse without syntax errors?
      - Does it follow [JS/Elixir] conventions?
      - Are there obvious bugs, missing imports, or undefined references?
      - Does it match what the task asked for?
   3. Check integration:
      - Do imports/references to other files resolve correctly?
      - Is the component/module properly exported?
      - Would this work with the rest of the application?
   4. Run a basic check:
      [For JS:] Try to detect TypeScript errors in the files
      [For Elixir:] Run `mix compile` and check for warnings/errors

   ## Expected Output — MANDATORY FORMAT
   Respond with EXACTLY this JSON:

   ```json
   {
     "reviewOf": "[agent task id]",
     "verdict": "PASS|FAIL",
     "issues": [
       {"severity": "critical|warning|info", "file": "path", "description": "what's wrong"}
     ],
     "suggestions": [],
     "notes": "overall assessment"
   }
   ```

   PASS = Code works, follows conventions, accomplishes the task.
   FAIL = Critical issues that must be fixed before proceeding.
   ```

2. **Process review result:**

   - If **PASS**:
     - Update task `qualityReview`: `{ "reviewedAt": "[timestamp]", "passed": true, "notes": "[review notes]" }`
     - Check off the task in brainstorm.md: `- [ ]` -> `- [x]`
     - Git commit the agent's files:
       ```bash
       cd [projectDir] && git add [specific files] && git commit -m "mvp: [task description]"
       ```
     - Update analytics: increment `qualityReviews.total`, `qualityReviews.passed`, `gitCommits`, `completedTasks`

   - If **FAIL**:
     - Update task `qualityReview`: `{ "reviewedAt": "[timestamp]", "passed": false, "notes": "[review notes]", "retryCount": [N] }`
     - If `retryCount < 3`:
       - Set task status back to "pending" (it will be retried in the next loop iteration)
       - Update `resumePoint.notes` with the failure context so the retry agent knows what went wrong
     - If `retryCount >= 3`:
       - Set task status to "failed"
       - Increment `analytics.failedTasks`
       - Show warning to user:
         ```
         Task failed after 3 attempts: [task description]
         Issues: [list issues from review]

         Skipping and continuing with remaining tasks.
         ```
     - Update analytics: increment `qualityReviews.total`, `qualityReviews.failed`

### Step E: Update brainstorm document

After processing all agents in a batch:

1. **Update Task Tracking section** with current counts
2. **Update Analytics section** with current metrics
3. **Append to Agent Log section** for each agent that ran:
   ```markdown
   ### Agent #[ID] — [Task Name]
   - **Phase:** [N] - [Phase Name]
   - **Status:** [SUCCESS/PARTIAL/FAILED]
   - **Started:** [timestamp]
   - **Completed:** [timestamp]
   - **Duration:** [calculated from timestamps]
   - **Files:** [list from agent result]
   - **Quality Review:** [PASS/FAIL]
   - **Notes:** [agent notes + review notes]
   ```

### Step F: Check phase completion

1. **Are all tasks in the current phase completed (or failed)?**
   - If YES:
     - Set phase `status: "completed"` and `completedAt`
     - Calculate `actualMinutes` from `startedAt` to `completedAt`
     - Update `currentPhase` to the next phase
     - **Run the phase gate check** (see Phase 6 below)
     - Go to Phase 3 (Determine Current Work) for the next phase
   - If NO:
     - Loop back to Step B with remaining uncompleted tasks

### Step G: Continue or pause

After each batch of agents:
- Check if context is getting large (many agents have been dispatched this session)
- If you've been running for a while (many tasks completed), consider pausing:
  ```
  Session checkpoint: [X] tasks completed this session.

  Progress: [completed]/[total] tasks overall
  Current phase: [N] - [Phase Name]
  ```
  - Update `resumePoint` with current state
  - End session log entry with `endedAt` and `tasksCompleted`
  - Tell user: "Run `/mvp build` to continue"

---

## Phase 6: Phase Gate Checks

At the end of each phase, verify the phase's gate criteria:

### Phase 1 Gate: Scaffold
- Dev server starts and responds (curl check)
- Base route renders without errors
- If FAIL: attempt fix, or mark as error and ask user

### Phase 2 Gate: Data Layer
- For JS: Drizzle schema compiles, DB file exists
- For Elixir: `mix compile` succeeds, `mix ecto.migrate` runs clean, `mix run priv/repo/seeds.exs` works
- If FAIL: attempt fix, or mark as error and ask user

### Phase 3 Gate: Core Feature
- Core user flow can be navigated (check that key routes exist and render)
- No compilation errors
- If FAIL: list what's broken, ask user if they want to proceed to polish or fix first

### Phase 4 Gate: UI Polish
- All screens render without errors
- Navigation between routes works
- No broken imports or references
- If FAIL: list issues, proceed to integration anyway (polish issues are non-blocking)

### Phase 5 Gate: Integration
- Full user flow works end-to-end
- `npm run build` (JS) or `mix compile` (Elixir) succeeds without errors
- README exists with accurate run instructions
- If FAIL: list remaining issues, mark as complete anyway (it's a prototype)

---

## Phase 7: Error Recovery

If something goes wrong during orchestration:

1. **Agent returns "failed" status:**
   - Log the failure
   - Retry up to 3 times with error context added to the prompt
   - After 3 failures, skip and continue

2. **Dev server crashes:**
   - Detect via failed curl check before dispatching new agents
   - Restart server (main agent)
   - Continue orchestration

3. **Dependency install fails:**
   - Log the error
   - Try alternative package names if obvious (e.g., `@types/` prefix)
   - If still fails, mark dependent tasks as blocked and continue with others

4. **Git commit fails:**
   - Check for merge conflicts (shouldn't happen if locks are used properly)
   - If conflicts: resolve manually (main agent), then commit
   - If other error: log and continue (git issues are non-critical for prototype)

5. **All remaining tasks in a phase are failed/blocked:**
   - Set `status: "error"` in state.json
   - Update `resumePoint.notes` with context
   - Show error summary to user
   - STOP and ask user to intervene

---

## Phase 8: Completion

When all phases are complete:

1. **Kill dev server** if running:
   ```bash
   kill [state.processes.devServer.pid] 2>/dev/null
   ```

2. **Final git commit:**
   ```bash
   cd [projectDir] && git add -A && git commit -m "mvp: [project-name] prototype complete"
   ```

3. **Update state:**
   - Set `status: "complete"`
   - Set `resumePoint.lastCompletedAt`
   - End session log entry
   - Write `state.json`

4. **Update brainstorm.md:**
   - Set Status to `complete`
   - Update `Last Updated` timestamp
   - Fill in "Build completed" timestamp in Analytics
   - Calculate total elapsed time

5. **Show completion message:**
   ```
   MVP COMPLETE: [App Name]

   Stack: [Stack Name]
   Phases completed: 5/5
   Tasks completed: [N]/[Total]
   Tasks failed: [N] (skipped)
   Agents dispatched: [N]
   Quality reviews: [passed]/[total] passed
   Git commits: [N]
   Total elapsed time: [calculated across all sessions]

   To run the app:
     cd [projectDir]
     [npm run dev | mix phx.server]
     Open http://localhost:[port]

   Artifacts:
     .mvp/brainstorm.md   — Full build log and project documentation
     .mvp/state.json      — Machine-readable state and analytics
     .mvp/agent-logs/     — Individual agent reports

   Generate analytics page:
     /mvp summary
   ```

---

## Execute Now

Begin with Phase 1 (Load and Validate State), then proceed through the orchestration loop.
