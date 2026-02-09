<!--
  Plugin: project-management v1.0.0
  Template generated: {{CURRENT_DATE}}

  This file is yours to customize for your project.
  Sections managed by the plugin have version markers (e.g. <!-- section: name v1 -->).
  Running /dr-init will check these markers and offer to update outdated sections.
-->

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

This project follows a structured approach to planning, documentation, and implementation.

```
_claude/
|-- docs/           # Technical documentation and architecture
|   `-- .gitkeep
|-- plans/          # Implementation plans
|   |-- draft/      # Plans being developed or refined
|   |   `-- .gitkeep
|   |-- in_progress/ # Plans currently being implemented
|   |   `-- .gitkeep
|   `-- completed/  # Finished and archived plans
|       `-- .gitkeep
|-- prd/            # Product Requirement Documents
|   `-- .gitkeep
|-- resources/      # Reference materials and external docs
|   `-- .gitkeep
`-- research/       # Structured research output
    `-- .gitkeep
```

**Note:** `.gitkeep` files ensure empty directories can be committed to git.

### Directory Purposes

**`_claude/docs/`**
Technical documentation, architecture decisions, API specifications, and development notes.

**`_claude/plans/`**
Implementation plans following a structured template. Plans move through stages:
- **draft/** - Plans being refined, NOT ready for implementation
- **in_progress/** - Plans currently being actively worked on
- **completed/** - Finished plans for historical reference

**`_claude/prd/`**
Product Requirement Documents defining features, user stories, and requirements.

**`_claude/resources/`**
User-provided reference materials, external documentation, design specifications.

**`_claude/research/`**
Structured research output with multiple markdown files per topic.

## Plan Management Workflow
<!-- section: plan-management-workflow v2 -->

### IMPORTANT: Plan Execution Rules

⚠️ **STRICT ADHERENCE REQUIRED** ⚠️

1. **NEVER execute or implement tasks from plans in the `draft/` folder**
   - Draft plans are for review and refinement only
   - If asked to work on a draft plan, inform the user that the plan must be moved to `in_progress` first

2. **Only work on plans in the `in_progress/` folder**
   - These are the only plans approved for active development
   - Always verify a plan is in the correct folder before starting work

3. **If a user asks you to work on a draft plan:**
   - Politely explain that draft plans cannot be executed
   - Offer to move the plan to `in_progress` if they approve
   - Wait for explicit confirmation before moving any plans

4. **Actions outside of existing plans require explicit permission**
   - If a request is NOT part of any existing plan, ASK THE USER before executing
   - Questions are for information gathering, NOT permission to act
   - Only proceed with implementation when the user explicitly says to do it

5. **Understanding user intent**
   - "Can you..." or "How would..." questions are requests for information, not action
   - "Please..." or "Go ahead and..." or "Implement..." are explicit action requests
   - When in doubt, clarify what the user wants before proceeding

### Plan Status Workflow

1. **Create Plan**: `/dr-plan [detailed context]` creates numbered plan in `draft/` (e.g., `001-plan-name.md`)
2. **Review**: Examine the plan to identify any improvements or missing details
3. **Refine** (optional but recommended): `/dr-plan @_claude/plans/draft/001-plan.md [refinement request]` to enhance with extended thinking
   - Can be repeated multiple times
   - Shows diff summary before applying
4. **Move to Active**: Move the plan file from `draft/` to `in_progress/` when ready to implement
5. **Implement**: Work through plan phases systematically
6. **Minor Adjustments** (as needed): `/dr-plan @_claude/plans/in_progress/001-plan.md [minor changes]` for small corrections
7. **Complete**: Move the plan file from `in_progress/` to `completed/` when finished

**Plan Numbering:**
Plans are automatically numbered sequentially (001, 002, 003, ..., 999, 1000, ...) to track chronological order. The number is determined by scanning **all three folders** (draft/, in_progress/, completed/) to find the highest existing number, then incrementing by 1. The number stays with the plan when moved between folders.

Example: If your completed/ folder has plans 001-045 and in_progress/ has 046-047, the next plan created will be 048, even if draft/ is empty.

## Available Commands
<!-- section: available-commands v1 -->

This project uses the **project-management** plugin (dr- prefix) which provides:

- `/dr-init` - Initialize or update project structure
- `/dr-research [detailed prompt]` - Conduct deep research with extended thinking (supports multi-line prompts)
- `/dr-prd [detailed feature description OR @prd-file [refinement]]` - Create or refine comprehensive PRD with extended thinking
- `/dr-plan [detailed context OR @plan-file [refinement]]` - Create or refine implementation plan with extended thinking (dual-mode)

**Dual-Mode Refinement:**
Both PRDs and plans can be refined using the same commands. Use `/dr-prd @_claude/prd/feature.md [changes]` to refine PRDs or `/dr-plan @_claude/plans/draft/plan.md [changes]` to refine plans. Both commands use extended thinking and show diff summaries. They automatically detect whether you're creating or refining based on the `@` file reference.

**IMPORTANT - Date Handling:**
When creating any document with dates or timestamps, ALWAYS check the system environment for the current date/time. NEVER use hardcoded or assumed dates.

## Project-Specific Commands

<!-- Add your project's build, test, lint, and development commands here -->

**Build:**
```bash
# [Add your build command]
```

**Test:**
```bash
# [Add your test command]
```

**Lint:**
```bash
# [Add your lint command]
```

**Development:**
```bash
# [Add your development server command]
```

## Development Principles

1. **Incremental Progress**: Build features incrementally with working code at each step
2. **Documentation First**: Document plans and decisions before implementing major features
3. **Test Coverage**: Write tests alongside implementation
4. **Code Review**: Review changes before merging
5. **Clear Communication**: Write clear commit messages and PR descriptions

## Task Completion Protocol
<!-- section: task-completion-protocol v1 -->

When working on tasks from an implementation plan, follow this protocol **for each phase**:

1. **Work through one phase at a time** following the plan's phase order
2. **Complete all tasks in the phase** and verify each works (run tests, check behavior)
3. **Update the plan file immediately** after completing the phase:
   - Check the boxes (`[x]`) for all completed tasks in that phase
   - Check the boxes for completed test verification items
   - Update the "Actual Time" in Implementation Notes if time tracking is present
4. **Report to the user** with a summary of what was completed and what phase is next
5. **Proceed to the next phase** or wait for user direction

The plan file must always reflect the current state of implementation. Never leave completed tasks unchecked.

---

<!-- End of plugin-managed section -->
<!-- Add project-specific instructions below -->
