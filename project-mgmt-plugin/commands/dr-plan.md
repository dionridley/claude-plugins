---
description: Create or refine a detailed implementation plan with extended thinking
argument-hint: [implementation context OR @plan-file [refinement|summary|answer questions]] [--no-confirm|--in-progress]
allowed-tools: Read, Write, Edit, Grep, Bash(ls:*), Bash(find:*), Bash(cp:*)
---

# Create or Refine Implementation Plan

This command creates a comprehensive implementation plan OR refines an existing plan with thoughtful analysis.

## Instructions for Claude

You are executing the `/dr-plan` command in multi-mode: CREATE, REFINE, SUMMARY, or QUESTION RESOLUTION.

### Mode Detection

**CRITICAL**: The command arguments are provided in the `<command-args>` tag at the top of this message.

1. **Check the `<command-args>` value:**
   - If it contains "summary" (with a plan file reference via `@`):
     → **SUMMARY mode** (jump to "SUMMARY Mode" section below)
   - If it contains "answer questions" or "resolve questions" (with a plan file reference via `@`):
     → **QUESTION RESOLUTION mode** (jump to "Question Resolution Mode" section)
   - If it contains a plan file reference via `@` (but no special keywords):
     → **REFINE mode** (jump to "REFINE Mode" section below)
   - If it does NOT contain a plan file reference:
     → **CREATE mode** (continue with steps below)
   - If it is empty or contains only whitespace:
     → **CREATE mode** with interactive prompt

**Note**: When a user uses `@file.md`, the file content is auto-expanded into the conversation context. The `<command-args>` tag will contain any text AFTER the `@` reference (like "summary", "answer questions", etc.).

---

## SUMMARY MODE: Generate PR Summary

This mode generates a Pull Request summary from the plan content.

**Trigger**: `/dr-plan @_claude/plans/.../plan-file.md summary`

### Instructions

1. **Read the summary instructions file:**
   - Read: `${CLAUDE_PLUGIN_ROOT}/commands/dr-plan/summary.md`

2. **Apply those instructions:**
   - The plan content is already available in the conversation (auto-expanded via `@`)
   - Follow the instructions in summary.md to generate the PR summary
   - Display the formatted output as specified

3. **Do NOT proceed to other modes** - STOP after completing the summary

---

## QUESTION RESOLUTION MODE: Answer Plan Questions

This mode helps resolve uncertain assumptions and open questions in a plan through interactive Q&A.

**Trigger**: `/dr-plan @_claude/plans/.../plan-file.md answer questions`

### Instructions

1. **Read the questions instructions file:**
   - Read: `${CLAUDE_PLUGIN_ROOT}/commands/dr-plan/questions.md`

2. **Apply those instructions:**
   - The plan content is already available in the conversation (auto-expanded via `@`)
   - Follow the instructions in questions.md to guide the user through resolution
   - Update the plan file with resolved decisions

3. **Do NOT proceed to other modes** - STOP after completing question resolution

---

## CREATE MODE: Generate New Plan

### Phase 1: Gather Implementation Context

**IMPORTANT**: At the very top of this message, you will find a `<command-args>` tag that contains the actual arguments passed to this command. You MUST check this tag to determine if arguments were provided.

1. **Check the `<command-args>` tag value (at the top of this message):**
   - Look at the `<command-args>` tag content
   - If it contains text (not empty or whitespace): Use that text as the implementation context
   - If it is empty or only whitespace: Ask user interactively for detailed implementation context

2. **Interactive prompt (if needed when `<command-args>` is empty):**
   - Ask: "What would you like to create an implementation plan for? Please provide details including:"
   - "  - What you're implementing"
   - "  - Why it's needed (problem being solved)"
   - "  - Current state of the codebase"
   - "  - Technical stack and constraints"
   - "  - Time expectations or deadlines"
   - "  - Any PRD or requirements documents to reference"
   - Wait for user response before proceeding

### Phase 2: Parse Context and References

1. **Check for PRD file references in the `<command-args>` content:**
   - Look for `@_claude/prd/[filename].md` pattern in the command arguments you found in Phase 1
   - Example patterns: `@_claude/prd/auth-system.md` or `@_claude/prd/feature.md`
   - If found:
     - Extract the PRD file path (remove the `@` prefix)
     - Convert backslashes to forward slashes if needed (Windows paths)
     - Verify the file exists using Read tool
     - If file exists: Read the entire PRD content
     - If file doesn't exist: Show error and ask user to verify the path
     - Store PRD path for metadata section
   - If not found: No PRD reference (proceed normally)

2. **Check for --in-progress flag:**
   - If present: Plan will be created in `_claude/plans/in_progress/` folder
   - If not present: Plan will be created in `_claude/plans/draft/` folder (default)

3. **CRITICAL: Check current date/time**
   - Access the system environment to get the ACTUAL current date
   - Format as YYYY-MM-DD
   - NEVER use hardcoded or assumed dates

### Phase 3: Determine Plan Number

**CRITICAL**: Plans are numbered sequentially across ALL folders. You MUST scan all three folders to find the highest number.

1. **Scan ALL plan folders:**
   ```
   Use Bash tool:
   find _claude/plans/ -name "*.md" -type f 2>/dev/null | grep -E "/[0-9]+-.*\.md$" || echo "No plans found"
   ```

2. **Parse all filenames to extract numbers:**
   - Look for pattern: `XXX-[name].md` where XXX is the number
   - Examples: `001-auth.md`, `042-migration.md`, `1000-large-feature.md`
   - Extract all numbers from ALL folders (draft/, in_progress/, completed/)
   - Handle both 3-digit (001-999) and 4+ digit (1000+) numbers

3. **Determine next plan number:**
   - Find the highest number across ALL folders
   - Increment by 1
   - If no plans exist anywhere: Start at 001
   - Format with leading zeros if ≤999: `001`, `042`, `999`
   - Format without leading zeros if >999: `1000`, `1001`

4. **Example scenarios:**
   - All folders empty → Next number: `001`
   - completed/ has 001-050, in_progress/ has 051-052 → Next number: `053`
   - Only draft/ has 001-003 → Next number: `004`
   - Plans scattered across all folders, highest is 127 → Next number: `128`

### Phase 4: Analyze and Plan

1. **If PRD was referenced:**
   - Review the PRD content thoroughly
   - Extract key requirements and constraints
   - Identify success criteria from PRD
   - Note any technical considerations mentioned
   - Use PRD as foundation for planning

2. **Use extended thinking to deeply analyze the implementation:**
   - What exactly needs to be built?
   - What is the current state (what exists, what's missing)?
   - What are the logical phases for implementation?
   - What are the specific tasks within each phase?
   - What dependencies exist (external or internal)?
   - What could go wrong? What are the risks?
   - How long will this realistically take?
   - What testing is needed for each phase?
   - How can this be rolled back if needed?

3. **Identify Assumptions and Open Questions:**

   **Assumptions Made:**
   - List assumptions you're making about the implementation
   - Mark confirmed assumptions with `[x]` (based on codebase, PRD, or explicit user input)
   - Mark uncertain assumptions with `[ ]` and `[?]` marker
   - Examples: technology choices, database schema, API design, user flows

   **Open Questions - Blocking:**
   - Identify questions that MUST be answered before implementation can proceed
   - These are decisions that significantly affect architecture or approach
   - Mark with `[AWAITING]` status
   - Provide options where possible (Option A, Option B, etc.)

   **Open Questions - Non-Blocking:**
   - Identify questions that are nice-to-resolve but don't block progress
   - Can be resolved during implementation
   - Mark with `[OPEN]` status

   **If there are critical blocking questions:**
   - Consider using AskUserQuestion tool to get immediate answers
   - Only for truly critical decisions that would dramatically change the plan
   - Otherwise, document in Open Questions section for later resolution

4. **Optionally examine current codebase:**
   - If helpful, read relevant files to understand current state
   - Only if it would inform better planning
   - Don't read unnecessarily

4. **Extract plan name:**
   - Identify the main feature/task from the description
   - Create a slug (lowercase, hyphens for spaces, remove special chars)
   - Example: "Implement User Authentication System" → "implement-user-authentication-system"

### Phase 5: Create Plan

1. **Read plan template:**
   - Read from plugin: `${CLAUDE_PLUGIN_ROOT}/templates/plan-template.md`

2. **Determine file path:**
   - Folder: `_claude/plans/draft/` (default) or `_claude/plans/in_progress/` (if --in-progress flag)
   - Filename: `[number]-[plan-slug].md`
   - Example: `_claude/plans/draft/001-implement-user-authentication-system.md`

3. **Create plan file with ALL sections thoughtfully populated:**

   **Metadata:**
   - Created: [ACTUAL current date from system]
   - Status: Draft (or "In Progress" if --in-progress flag)
   - Related PRD: [PRD file path if referenced, otherwise "N/A"]
   - Refinements: None

   **Executive Summary:**
   - 2-3 sentences: what is being implemented, why it matters, what problem it solves
   - Be specific and clear

   **Current State:**
   - Objective assessment of what exists now
   - What is broken or incomplete
   - Specific TODOs or placeholders
   - Test status if applicable
   - Based on actual codebase examination if you looked

   **Assumptions Made:**
   - List all assumptions from Phase 4 step 3
   - Use `[x]` for confirmed assumptions
   - Use `[ ]` with `[?]` for uncertain assumptions that need verification
   - Include note: "Items marked [?] are uncertain - please confirm or correct"
   - Include instruction: "To resolve: /dr-plan @this-plan.md answer questions"

   **Open Questions & Decisions:**

   *Blocking (must resolve before implementation):*
   - List questions from Phase 4 that block progress
   - Format: `- [ ] **Topic** [AWAITING]` followed by question and options
   - Provide 2-3 options where possible
   - These MUST be resolved before implementation begins

   *Non-Blocking (can resolve during implementation):*
   - List questions from Phase 4 that don't block progress
   - Format: `- [ ] **Topic** [OPEN]` followed by question
   - These can be resolved as implementation proceeds

   **If no blocking questions exist:**
   - Still include the section header
   - Add: "No blocking questions identified - ready to proceed"

   **Success Criteria:**
   - Specific, measurable deliverables with checkboxes
   - Test outcomes (e.g., "All auth tests passing")
   - Performance or quality metrics if applicable

   **Implementation Plan:**
   - Break down into logical phases
   - Each phase should have:
     - Clear phase name
     - Estimated time in hours
     - Specific, actionable tasks with checkboxes
     - Test verification steps
     - Code changes needed (examples where helpful)
   - Phases should build on each other logically

   **Rollback Plan:**
   - Simple, clear steps to revert changes
   - What to do if things go wrong
   - How to restore previous state

   **Dependencies:**
   - Prerequisites that must exist or be done first
   - External dependencies (libraries, services, etc.)
   - Other plans that must complete first
   - Each with checkbox

   **Success Metrics:**
   - Placeholder section to be filled in after implementation
   - Structure with checkboxes for final results

4. **If critical information is missing:**
   - Ask 1-2 clarifying questions before proceeding
   - Only ask if the answer would significantly improve the plan

### Phase 6: Confirm Completion

1. **Show success message:**
   ```
   ✅ Implementation plan created: _claude/plans/[folder]/[number]-[plan-slug].md

   Plan #[number]: [plan name]
   Location: _claude/plans/[folder]/[number]-[plan-slug].md
   Status: [Draft/In Progress]
   [If PRD referenced: Related PRD: [prd-path]]

   Comprehensive plan includes:
     - Executive summary and current state analysis
     [If PRD referenced: - Requirements aligned with PRD specifications]
     - [N] assumptions documented ([M] need verification)
     - [N] phases with detailed tasks
     - Estimated time: [X] hours total
     - Test verification for each phase
     - Dependencies and rollback plan

   [If blocking questions exist:]
   ⚠️  Blocking Questions: [N] questions need answers before implementation
     Review the "Open Questions & Decisions" section in the plan.
     To resolve: /dr-plan @_claude/plans/[folder]/[number]-[slug].md answer questions

   [If no blocking questions:]
   ✓ No blocking questions - ready for implementation after review

   Key Insights:
     - [3-5 important considerations you identified while analyzing]
     [If PRD referenced: - [Key requirements from PRD that shaped the plan]]

   Next steps:
     1. Review the plan, especially Assumptions and Open Questions
     [If blocking questions: 2. Answer blocking questions: /dr-plan @plan answer questions]
     [If PRD referenced: 2. Verify alignment with PRD requirements]
     3. Refine if needed: /dr-plan @_claude/plans/[folder]/[number]-[plan-slug].md [changes]
     4. When ready to implement: /dr-move-plan [number] in-progress
     5. IMPORTANT: Only work on plans in 'in_progress' folder!

   [If created in draft/:]
   Note: Plan created in draft/ - move to in_progress/ before implementing.
   ```

---

## REFINE MODE: Update Existing Plan

### Phase 1: Parse Arguments

**IMPORTANT**: Look at the top of this message for the `<command-args>` tag that contains the actual arguments.

1. **Extract components from the `<command-args>` tag value (at the top of this message):**
   - File reference: Everything starting with `@_claude/plans/` until next space or flag
     - Example: `@_claude/plans/draft/001-auth-system.md`
   - Refinement request: Text after file reference, before any flags
     - Example: "Add OAuth provider integration to Phase 2"
   - Flags: `--no-confirm` (if present, skip confirmation step)

2. **Parse the file path:**
   - Remove the `@` prefix
   - Convert backslashes to forward slashes if needed (Windows paths)
   - Extract the folder (draft/, in_progress/, or completed/)
   - Extract just the filename (e.g., "001-auth-system.md")
   - Store full path for later use

### Phase 2: Validate Plan File

1. **Check if file exists:**
   - Try to read the file at the specified path
   - If not found: Show error with available plans

2. **Verify it's a plan file:**
   - Check if file matches plan naming pattern (XXX-*.md)
   - Check if file is in one of the three plan folders
   - If not: Show error explaining it's not a valid plan file

3. **If validation fails, show helpful error:**
   ```
   ❌ Plan file not found: [path]

   Available plans:
     draft/:
       - 001-authentication-system.md
       - 024-new-feature.md
     in_progress/:
       - 003-database-migration.md
     completed/:
       - 015-oauth-integration.md

   Usage: /dr-plan @_claude/plans/draft/001-authentication-system.md [refinement request]
   Tip: Use tab completion for file paths
   ```

### Phase 3: Read and Analyze Existing Plan

1. **Read the plan file completely**

2. **Parse metadata:**
   - Extract Created date
   - Extract Status (Draft/In Progress/Completed)
   - Extract Related PRD (if present)
   - Extract Refinements count

3. **Parse all sections:**
   - Executive Summary
   - Current State
   - Success Criteria
   - Implementation Plan (all phases)
   - Rollback Plan
   - Dependencies
   - etc.

4. **Detect plan status from folder location:**
   - File in `draft/` → Status: Draft
   - File in `in_progress/` → Status: In Progress
   - File in `completed/` → Status: Completed

### Phase 4: Check Status and Handle Accordingly

1. **Check plan status:**

   - **If Completed:**
     - STOP immediately and show error:
     ```
     ❌ Cannot refine completed plan

     Plan #[number]: [Plan Name]
     Status: Completed (historical record)

     Completed plans are archived for historical reference and should not be modified.

     Instead:
       1. Create a new plan incorporating lessons learned
       2. Reference the completed plan: /dr-plan [new context] "Building on plan [number]..."
       3. Document what would be done differently in the new plan
     ```
     - DO NOT PROCEED with refinement

   - **If In Progress:**
     - Note that this will need a warning later if major changes
     - Continue to next step

   - **If Draft:**
     - No special warnings needed
     - Continue to next step

### Phase 5: Create Backup

1. **Create automatic backup:**
   - Copy current plan to `.{filename}.backup` in same directory
   - Example: `_claude/plans/draft/001-auth-system.md` → `_claude/plans/draft/.001-auth-system.md.backup`
   - Use: `cp _claude/plans/[folder]/[filename].md _claude/plans/[folder]/.[filename].backup`
   - Overwrite if backup already exists (keep only most recent)

### Phase 6: Analyze Refinement Request

1. **Use extended thinking to deeply analyze:**

   **Understand existing plan:**
   - What is it implementing?
   - What are the current phases and tasks?
   - What's the technical approach?
   - What's well-defined vs. what's vague?

   **Understand refinement request:**
   - What specific changes are being requested?
   - Is this a minor clarification or major restructuring?
   - Which sections/phases need updates?
   - Are these additive changes or replacements?

   **Plan integration:**
   - How to incorporate changes while preserving structure?
   - What sections need updates?
   - What should remain unchanged?
   - Is this a minor or major refinement?

   **For in_progress plans:**
   - Are these changes compatible with work already done?
   - Do they invalidate completed tasks?
   - Should plan be moved back to draft for major changes?

### Phase 7: Generate Refined Plan

1. **Apply requested changes thoughtfully:**
   - Make the specific changes requested
   - Update related sections for consistency
   - Preserve overall plan structure and format
   - Maintain professional tone and clarity

2. **Update metadata:**
   - Status: Keep same
   - Created: KEEP ORIGINAL (never change)
   - Related PRD: Keep same (unless specifically changing)
   - Refinements: Increment count (e.g., "None" → "1", "2" → "3")

3. **Add to Refinement History section (create if doesn't exist):**
   ```markdown
   ## Refinement History

   **Refinements:**

   - [current-date]: [Brief description of what was refined]
   ```

   If section already exists, add new entry to the list.

### Phase 8: Generate Diff Summary

1. **Compare original vs. refined plan and categorize changes:**

   **Additions (+):**
   - New phases added
   - New tasks added
   - New sections or content added

   **Modifications (~):**
   - Changed content in existing sections
   - Updated tasks or time estimates
   - Clarified or expanded text

   **Deletions (-):**
   - Removed content (if any)
   - Deleted tasks or phases

   **Preserved:**
   - Major sections that remain unchanged
   - Metadata that's preserved

2. **Assess scope of changes:**
   - Minor: Small additions, clarifications, minor adjustments
   - Major: New phases, significant restructuring, substantial scope changes

3. **Create clear, terminal-friendly summary:**
   ```
   Changes preview:
     + Added: Phase 2.5 - OAuth Integration (3 hours)
       - Add Google OAuth provider
       - Add GitHub OAuth provider
       - Test OAuth flows
     ~ Modified: Phase 4 - Testing
       - Added OAuth integration test cases
     ~ Modified: Dependencies
       - Added OAuth provider registration requirement

   Preserved:
     - Phases 1, 2, 3 unchanged
     - Rollback plan unchanged

   Scope: [Minor/Major] changes
   ```

### Phase 9: Request Confirmation

1. **If --no-confirm flag is present:**
   - Skip to Phase 10 immediately
   - Do not ask for confirmation

2. **Otherwise, show confirmation prompt based on status:**

   **If plan is In Progress AND major changes detected:**

   Show this warning:
   ```
   ⚠️  WARNING: This plan is in progress (may have completed tasks)

   Plan #[number]: [Plan Name]
   Current status: In Progress
   Your request: "[refinement request]"

   This appears to be a major structural change that could invalidate completed work.

   [Show diff summary from Phase 8]

   Recommendations:
     1. Move plan back to draft for major redesign: /dr-move-plan [number] draft
     2. Create a new plan for the new approach: /dr-plan [new approach]
     3. Continue with minor adjustments only

   Proceed with major changes to in-progress plan? [y/n]:
   ```

   **If plan is In Progress AND minor changes:**

   Show this message:
   ```
   ℹ️  Note: This plan is in progress

   Plan #[number]: [Plan Name]
   Status: In Progress

   Some tasks may already be completed. Verify changes don't conflict with work done.

   [Show diff summary from Phase 8]

   Apply these changes? [y/n/diff]:
   ```

   **If plan is Draft (normal case):**

   Show diff summary:
   ```
   [Show diff summary from Phase 8]

   Apply these changes? [y/n/diff]:
   ```

3. **Wait for user response:**
   - **y** or **yes**: Proceed to Phase 10
   - **n** or **no**: Cancel, show cancellation message, STOP
   - **diff**: Show detailed line-by-line diff using a comparison, then ask again

4. **If user cancels (n):**
   ```
   ❌ Refinement cancelled - no changes made

   Your plan remains unchanged. The backup was not needed and has been removed.

   To try again with different changes:
     /dr-plan @_claude/plans/[folder]/[filename].md [different refinement request]
   ```
   - Remove the backup file created
   - STOP - do not proceed

### Phase 10: Apply Changes

1. **Write refined plan to original file:**
   - Use Write tool to replace entire file content
   - Ensure atomic operation (write succeeds completely or not at all)
   - Preserve file permissions

### Phase 11: Confirm Success

1. **Show success message:**

   **IMPORTANT**: Check the scope of changes from Phase 8.

   **For all refinements:**

   Show success message:
   ```
   ✅ Plan refined successfully

   Plan #[number]: [Plan Name]
   Location: _claude/plans/[folder]/[filename].md
   Status: [current-status]

   Changes applied:
     [Concise summary of changes - same as diff summary]

   Backup saved: _claude/plans/[folder]/.[filename].backup

   Refinement count: [count] (see Refinement History in plan)

   Next steps:
     1. Review the refined plan
     2. Refine again if needed: /dr-plan @_claude/plans/[folder]/[filename].md [changes]
     [If in draft/:]
     3. When ready: /dr-move-plan [number] in-progress
   ```

---

## Important Notes

1. **Always check system date/time** - Never use hardcoded dates
2. **Use extended thinking** - Deeply analyze both the implementation context (CREATE) and refinement request (REFINE)
3. **Be comprehensive in CREATE mode** - Fill in ALL sections thoughtfully with specific, actionable content
4. **Be surgical in REFINE mode** - Make requested changes while preserving everything else
5. **Respect plan status** - Warn about in-progress plans with major changes, refuse completed plans
6. **Scan all folders for numbering** - MUST check draft/, in_progress/, AND completed/ to find highest number
7. **Create backups** - Always backup before refining
8. **PRD references** - Support `@_claude/prd/[file].md` syntax in CREATE mode
9. **Maintain history** - Always update Refinement History section
10. **Confirm before changing** - Unless --no-confirm flag is present

## Execute Now

Follow the instructions above based on the detected mode (CREATE, REFINE, SUMMARY, or QUESTION RESOLUTION).
