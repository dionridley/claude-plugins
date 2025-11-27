---
description: Create or refine a comprehensive PRD with extended thinking
argument-hint: [feature description OR @prd-file [refinement request]] [--no-confirm]
allowed-tools: Read, Write, Edit, Bash(ls:*), Bash(cp:*), Bash(grep:*), Grep
---

# Create or Refine Product Requirement Document (PRD)

This command creates a comprehensive Product Requirement Document OR refines an existing PRD with thoughtful analysis.

## Instructions for Claude

You are executing the `/dr-prd` command in dual-mode: CREATE or REFINE.

### Mode Detection

**CRITICAL**: The command arguments are provided in the `<command-args>` tag at the top of this message.

1. **Check the `<command-args>` value:**
   - If it starts with `@` → **REFINE mode** (jump to "REFINE Mode" section below)
   - If it does NOT start with `@` → **CREATE mode** (continue with steps below)
   - If it is empty or contains only whitespace → **CREATE mode** with interactive prompt

---

## CREATE MODE: Generate New PRD

### Phase 1: Gather Feature Description

1. **Check the `<command-args>` tag for arguments:**
   - If `<command-args>` contains text (not empty): Use it as the feature description
   - If `<command-args>` is empty: Ask user interactively for detailed feature description

2. **Interactive prompt (if needed):**
   - Ask: "What feature would you like to create a PRD for? Please provide details including:"
   - "  - Problem being solved"
   - "  - Target users"
   - "  - Key requirements or constraints"
   - "  - Technical context (stack, scale, etc.)"
   - "  - Success criteria"
   - Wait for user response before proceeding

### Phase 2: Analyze and Plan

1. **CRITICAL: Check current date/time**
   - Access the system environment to get the ACTUAL current date
   - Format as YYYY-MM-DD
   - NEVER use hardcoded or assumed dates

2. **Use extended thinking to deeply analyze the feature request:**
   - What problem does this really solve?
   - Who are the users and what are their needs?
   - What are the edge cases and potential challenges?
   - What technical considerations are important?
   - What could go wrong? What are the risks?
   - What would make this feature truly successful?
   - What questions remain unanswered?

3. **Extract feature name:**
   - Identify the main feature from the description
   - Create a slug (lowercase, hyphens for spaces, remove special chars)
   - Example: "Real-time Collaboration Features" → "realtime-collaboration-features"

### Phase 3: Create PRD

1. **Read PRD template:**
   - Read from plugin: `${CLAUDE_PLUGIN_ROOT}/templates/prd-template.md`

2. **Create PRD file:**
   - Path: `_claude/prd/[feature-slug].md`

3. **Thoughtfully populate ALL sections** based on your analysis:

   **Metadata:**
   - Status: Draft
   - Version: 1.0
   - Created: [ACTUAL current date from system]
   - Author: Claude Code
   - Last Updated: [same as Created]

   **Problem Statement:**
   - Deep understanding of the problem
   - Who experiences it and why it matters

   **Goals and Objectives:**
   - Clear, measurable primary goals
   - Specific success metrics with how they'll be measured

   **User Stories:**
   - Comprehensive stories covering different user types
   - Detailed acceptance criteria for each

   **Functional Requirements:**
   - Must Have: Core functionality
   - Should Have: Important but not critical
   - Nice to Have: Future enhancements

   **Non-Functional Requirements:**
   - Performance, Security, Scalability, Accessibility

   **Technical Considerations:**
   - Architecture notes, technology choices, constraints
   - Based on any technical context provided

   **Dependencies:**
   - What must exist or be done first

   **Timeline and Milestones:**
   - Realistic phases with target dates

   **Risks and Mitigation:**
   - Potential issues and how to address them

   **Open Questions:**
   - What needs clarification or decision

   **References:**
   - Link to any research documents or related PRDs

   **Refinement History:**
   - Version 1.0 - [Date] - Initial PRD creation

4. **If critical information is missing:**
   - Ask 1-2 clarifying questions before proceeding
   - Only ask if the answer would significantly improve the PRD

### Phase 4: Confirm Completion

1. **Show success message:**
   ```
   ✅ PRD created: _claude/prd/[feature-slug].md

   Comprehensive PRD includes:
     - Problem statement and goals (filled in based on your prompt)
     - User stories with acceptance criteria
     - Functional and non-functional requirements
     - Technical considerations and architecture notes
     - Dependencies and risks
     - Success metrics and timeline

   Key Considerations Identified:
     - [3-5 important points you identified while analyzing]

   Next steps:
     1. Review and refine the PRD
     2. Discuss with stakeholders
     3. Refine if needed: /dr-prd @_claude/prd/[feature-slug].md [changes]
     4. Create implementation plan: /dr-plan [implementation context]
   ```

---

## REFINE MODE: Update Existing PRD

### Phase 1: Parse Arguments

1. **Extract components from the `<command-args>` tag value:**
   - File reference: Everything starting with `@` until next space or flag
     - Example: `@_claude/prd/auth-system.md`
   - Refinement request: Text after file reference, before any flags
     - Example: "Add MFA requirement with TOTP support"
   - Flags: `--no-confirm` (if present, skip confirmation step)

2. **Parse the file path:**
   - Remove the `@` prefix
   - Convert backslashes to forward slashes if needed (Windows paths)
   - Extract just the filename (e.g., "auth-system.md")
   - Store full path for later use

### Phase 2: Validate PRD File

1. **Check if file exists:**
   - Try to read the file at the specified path
   - If not found: Show error with available PRDs

2. **Verify it's a PRD file:**
   - Check if file has PRD structure (has "# PRD:" or "**Status:**")
   - If not: Show error explaining it's not a valid PRD file

3. **If validation fails, show helpful error:**
   ```
   ❌ PRD file not found: [path]

   Available PRDs in _claude/prd/:
     - authentication-system.md
     - realtime-collaboration.md
     - notification-service.md

   Usage: /dr-prd @_claude/prd/[filename].md [refinement request]
   Tip: Use tab completion for file paths
   ```

### Phase 3: Read and Analyze Existing PRD

1. **Read the PRD file completely**

2. **Parse metadata:**
   - Extract Status (Draft/Under Review/Approved/Superseded)
   - Extract Version (e.g., 1.0, 1.1, 2.0)
   - Extract Created date
   - Extract Last Updated date
   - Extract Author

3. **Parse all sections:**
   - Problem Statement
   - Goals
   - User Stories
   - Requirements
   - Technical Considerations
   - Dependencies
   - Risks
   - etc.

### Phase 4: Check Status and Linked Plans

1. **Check PRD status:**

   - **If Superseded:**
     - STOP immediately and show error:
     ```
     ❌ Cannot refine Superseded PRD

     PRD: [Feature Name] (v[version])
     Status: Superseded
     Superseded by: [new-file-name] (if mentioned)

     This is a historical version and should not be modified.

     Instead:
       1. Refine the current version: /dr-prd @_claude/prd/[current-version].md [changes]
       2. Create a new major version if needed: /dr-prd [new requirements based on current]
     ```
     - DO NOT PROCEED with refinement

   - **If Approved:**
     - Note that this will need a warning later
     - Continue to next step

   - **If Under Review:**
     - Note for mild warning later
     - Continue to next step

   - **If Draft:**
     - No warnings needed
     - Continue to next step

2. **Search for linked plans:**

   **CRITICAL**: You MUST execute BOTH methods below. Do not skip Method 1!

   **METHOD 1: Find plan references IN the PRD content (PRD → Plan links)**

   **STEP 1a:** Search the PRD file content for plan references using Grep tool:
   ```
   Use Grep tool with:
   - pattern: "@_claude/plans/"
   - path: [the PRD file path you're refining]
   - output_mode: "content"
   ```

   **STEP 1b:** Parse the grep results to extract plan file paths:
   - Look for patterns like `@_claude/plans/draft/[filename].md`
   - Or `@_claude/plans/in_progress/[filename].md`
   - Or `@_claude/plans/completed/[filename].md`
   - Extract complete paths (e.g., `_claude/plans/draft/fake-plan.md`)
   - Remove the `@` prefix to get actual file paths
   - Store these in a list called `prd_to_plan_links`

   **Example:**
   If grep finds: `**Plan:** '@_claude/plans/draft/fake-plan.md'`
   Then extract: `_claude/plans/draft/fake-plan.md`

   **METHOD 2: Find PRD references IN plan files (Plan → PRD links)**

   **STEP 2a:** Search plan files for references to this PRD using Grep tool:
   ```
   Use Grep tool with:
   - pattern: "@_claude/prd/[actual-prd-filename].md"
   - path: "_claude/plans/"
   - output_mode: "files_with_matches"
   ```

   **STEP 2b:** Extract plan filenames from results:
   - Grep returns file paths of plans that reference this PRD
   - Store these in a list called `plan_to_prd_links`

   **COMBINE RESULTS:**

   **STEP 3:** Merge both lists:
   - Combine `prd_to_plan_links` + `plan_to_prd_links`
   - Remove duplicates
   - Store the final merged list as `linked_plans`
   - Use this `linked_plans` list in Phase 9 (confirmation) and Phase 11 (success message)

   **Example output to track:**
   ```
   METHOD 1 found: _claude/plans/draft/fake-plan.md
   METHOD 2 found: (none)
   Total linked plans: 1
   ```

### Phase 5: Create Backup

1. **Create automatic backup:**
   - Copy current PRD to `.{filename}.backup` in same directory
   - Example: `_claude/prd/auth-system.md` → `_claude/prd/.auth-system.md.backup`
   - Use: `cp _claude/prd/[filename].md _claude/prd/.[filename].backup`
   - Overwrite if backup already exists (keep only most recent)

### Phase 6: Analyze Refinement Request

1. **Use extended thinking to deeply analyze:**

   **Understand existing PRD:**
   - What problem is it solving?
   - What are the current requirements and scope?
   - What's the technical approach?
   - What's well-defined vs. what's vague?

   **Understand refinement request:**
   - What specific changes are being requested?
   - Why are these changes needed?
   - How do they impact the overall PRD?
   - Are these minor clarifications or major scope changes?

   **Plan integration:**
   - How to incorporate changes while preserving structure?
   - What sections need updates?
   - What should remain unchanged?
   - Impact on linked plans (if any)?

   **Determine version increment:**
   - Minor refinement (clarifications, small additions): Increment minor version (1.0 → 1.1)
   - Major changes (scope change, significant new requirements): Increment major version (1.1 → 2.0)

### Phase 7: Generate Refined PRD

1. **Apply requested changes thoughtfully:**
   - Make the specific changes requested
   - Update related sections for consistency
   - Preserve overall PRD structure and format
   - Maintain professional tone and clarity

2. **Update metadata:**
   - Status: Keep same UNLESS major changes warrant "Under Review"
   - Version: Increment appropriately (minor or major)
   - Created: KEEP ORIGINAL (never change)
   - Last Updated: Set to ACTUAL current date from system
   - Author: Keep original

3. **Add to Refinement History section:**
   ```markdown
   **Version [new-version]** - [current-date]

   - [Brief description of what was refined]
   ```

### Phase 8: Generate Diff Summary

1. **Compare original vs. refined PRD and categorize changes:**

   **Additions (+):**
   - New sections added
   - New requirements added
   - New user stories added
   - New content in existing sections

   **Modifications (~):**
   - Changed content in existing sections
   - Updated requirements
   - Clarified or expanded text

   **Deletions (-):**
   - Removed content (if any)
   - Deleted requirements or sections

   **Preserved:**
   - Major sections that remain unchanged
   - Metadata that's preserved

2. **Create clear, terminal-friendly summary:**
   ```
   Changes preview:
     + Added: [Section/Content name]
       - [Key point 1]
       - [Key point 2]
     ~ Modified: [Section name]
       - [What changed]
     - Deleted: [Content name] (if any)

   Preserved:
     - [Major sections unchanged]

   Version: [old] → [new]
   ```

### Phase 9: Request Confirmation

1. **If --no-confirm flag is present:**
   - Skip to Phase 10 immediately
   - Do not ask for confirmation

2. **Otherwise, show confirmation prompt:**

   **IMPORTANT**: Check the list of linked plans you found in Phase 4. If any plans were found, include the linked plans warning in your confirmation message.

   **If PRD status is Approved:**

   Show this message:
   ```
   ⚠️  WARNING: This PRD is Approved

   PRD: [Feature Name]
   Current Version: [version]
   Status: Approved

   Your requested change: "[refinement request]"

   This PRD has been approved by stakeholders. Changes may require:
     - Stakeholder re-approval
     - Updates to linked implementation plans
   ```

   Then, IF you found linked plans in Phase 4, add this section:
   ```
   Referenced by [N] plans:
     - [plan-file-1]
     - [plan-file-2]
   ```

   Then continue with:
   ```

   Consider:
     1. Moving status back to "Under Review" for significant changes
     2. Creating a new major version (v[N].0) for major scope changes
     3. Continuing with minor refinements only

   Continue with changes to Approved PRD? [y/n]:
   ```

   **If PRD status is Under Review:**

   Show this message:
   ```
   ℹ️  Note: This PRD is Under Review

   PRD: [Feature Name]
   Status: Under Review

   Stakeholders may be reviewing this PRD. Consider coordinating changes.

   [Show diff summary from Phase 8]
   ```

   Then, IF you found linked plans in Phase 4, add this warning:
   ```
   ⚠️  This PRD is referenced by [N] plans:
     - [plan-file-1]
     - [plan-file-2]
   ```

   Then finish with:
   ```
   Apply these changes? [y/n/diff]:
   ```

   **If PRD status is Draft (normal case):**

   Show diff summary:
   ```
   [Show diff summary from Phase 8]
   ```

   Then, IF you found linked plans in Phase 4, add this warning:
   ```
   ⚠️  This PRD is referenced by [N] plans:
     - [plan-file-1]
     - [plan-file-2]
   ```

   Then finish with:
   ```
   Apply these changes? [y/n/diff]:
   ```

3. **Wait for user response:**
   - **y** or **yes**: Proceed to Phase 10
   - **n** or **no**: Cancel, show cancellation message, STOP
   - **diff**: Show detailed line-by-line diff using a comparison, then ask again

4. **If user cancels (n):**
   ```
   ❌ Refinement cancelled - no changes made

   Your PRD remains unchanged. The backup was not needed and has been removed.

   To try again with different changes:
     /dr-prd @_claude/prd/[filename].md [different refinement request]
   ```
   - Remove the backup file created
   - STOP - do not proceed

### Phase 10: Apply Changes

1. **Write refined PRD to original file:**
   - Use Write tool to replace entire file content
   - Ensure atomic operation (write succeeds completely or not at all)
   - Preserve file permissions

### Phase 11: Confirm Success

1. **Show success message:**

   **IMPORTANT**: Check the list of linked plans from Phase 4. Show appropriate success message based on whether linked plans exist.

   **If NO linked plans were found in Phase 4:**

   Show basic success message:
   ```
   ✅ PRD refined successfully

   PRD: [Feature Name] (v[old-version] → v[new-version])
   Location: _claude/prd/[filename].md
   Status: [current-status]

   Changes applied:
     [Concise summary of changes - same as diff summary]

   Backup saved: _claude/prd/.[filename].backup

   Next steps:
     1. Review the refined PRD
     2. Update status if stakeholder re-approval needed
     3. Refine again if needed: /dr-prd @_claude/prd/[filename].md [more changes]
     4. Create or update implementation plans
   ```

   **If linked plans WERE found in Phase 4:**

   Show success with linked plans warning:
   ```
   ✅ PRD refined successfully

   PRD: [Feature Name] (v[old-version] → v[new-version])
   Location: _claude/prd/[filename].md
   Status: [current-status] [⚠️ May need stakeholder re-approval if Approved]

   Changes applied:
     [Concise summary of changes]

   Backup saved: _claude/prd/.[filename].backup

   ⚠️  This PRD is referenced by [N] plans:
     - [plan-file-1]
     - [plan-file-2]

   Recommended actions:
     1. Review PRD changes and update status if needed
     2. Review each linked plan for alignment with updated requirements
     3. Consider refining plans that may be affected:
        /dr-plan @[plan-path] Align with PRD v[new-version] [change description]
   ```

---

## Important Notes

1. **Always check system date/time** - Never use hardcoded dates
2. **Use extended thinking** - Deeply analyze both the feature request (CREATE) and refinement request (REFINE)
3. **Be comprehensive in CREATE mode** - Fill in ALL sections thoughtfully, don't leave templates empty
4. **Be surgical in REFINE mode** - Make requested changes while preserving everything else
5. **Respect PRD status** - Warn about Approved PRDs, refuse Superseded PRDs
6. **Create backups** - Always backup before refining
7. **Check for linked plans** - Warn user if plans reference this PRD
8. **Version appropriately** - Minor for clarifications, major for scope changes
9. **Maintain history** - Always update Refinement History section
10. **Confirm before changing** - Unless --no-confirm flag is present

## Execute Now

Follow the instructions above based on the detected mode (CREATE or REFINE).
