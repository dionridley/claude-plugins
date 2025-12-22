---
description: Move a plan between draft, in_progress, and completed
argument-hint: [plan-name-or-number-or-@file] [draft|in-progress|completed]
allowed-tools: Bash(mv:*), Glob, Read, AskUserQuestion
---

# Move Plan Between Stages

This command moves a plan between the `draft/`, `in_progress/`, and `completed/` folders while preserving the plan number.

## Instructions for Claude

You are executing the `/dr-move-plan` command. Parse the arguments to identify the plan and destination, then move the plan file.

### Arguments Structure

The command receives TWO arguments:
1. **Plan identifier** - One of:
   - **Plan number**: e.g., `001`, `42`, `123`
   - **Plan name/slug**: e.g., `authentication`, `auth`, `user-auth-system`
   - **File reference**: e.g., `@_claude/plans/draft/001-authentication-system.md`

2. **Destination**: Must be one of:
   - `draft` - Move to `_claude/plans/draft/`
   - `in-progress` - Move to `_claude/plans/in_progress/`
   - `completed` - Move to `_claude/plans/completed/`

**Arguments received:** `$ARGUMENTS`

### Phase 1: Parse Arguments

1. **Check if arguments are provided:**
   - If `$ARGUMENTS` is empty, show usage help and ask for the plan and destination

2. **Split arguments** into plan identifier and destination
   - The LAST word should be the destination (draft, in-progress, or completed)
   - Everything before that is the plan identifier

3. **Validate destination:**
   - Must be exactly one of: `draft`, `in-progress`, `completed`
   - If invalid, show error with valid options

### Phase 2: Find the Plan

**Depending on the plan identifier type:**

#### Case A: File Reference (starts with `@`)

1. Extract the file path from the `@` reference (remove the `@` prefix)
2. Verify the file exists using Read tool
3. Verify the file is in one of the three plan folders:
   - `_claude/plans/draft/`
   - `_claude/plans/in_progress/`
   - `_claude/plans/completed/`
4. Verify the filename matches plan pattern: `XXX-*.md` (where XXX is digits)
5. If any validation fails, show helpful error
6. Extract plan number and name from filename
7. Proceed to Phase 3

#### Case B: Plan Number (digits only, e.g., `001`, `42`)

1. Normalize the number (keep leading zeros for matching)
2. Search all three folders for plan files using Glob:
   ```
   Use Glob tool with pattern: _claude/plans/**/*.md
   ```
3. Filter the results to find files matching the plan number:
   - From Glob results, identify files matching pattern `XXX-*.md` where XXX starts with the search number
   - Match files where the number prefix matches exactly (e.g., `001` matches `001-auth.md` but not `0011-foo.md`)
4. Handle results:
   - **No matches**: Go to "Handle No Matches" section
   - **One match**: Proceed to Phase 3
   - **Multiple matches**: Go to "Handle Multiple Matches" section

#### Case C: Plan Name/Slug (contains letters)

1. Search all three folders for plan files using Glob:
   ```
   Use Glob tool with pattern: _claude/plans/**/*.md
   ```
2. Filter the results to find files containing the search term:
   - From Glob results, identify files where the slug portion contains the search term
   - Match case-insensitively where possible
3. Handle results:
   - **No matches**: Go to "Handle No Matches" section
   - **One match**: Proceed to Phase 3
   - **Multiple matches**: Go to "Handle Multiple Matches" section

### Handle No Matches

If no plan files match the search term:

1. List all available plans using Glob:
   ```
   Use Glob tool with pattern: _claude/plans/**/*.md
   ```
   Group the results by folder (draft/, in_progress/, completed/) for display.

2. Show error message:
   ```
   ❌ No plans found matching '[search term]'

   Available plans:
     draft/:
       - 001-authentication-system.md
       - 024-new-feature.md
     in_progress/:
       - 015-oauth-authentication.md
     completed/:
       - 003-setup-database.md
       - 012-api-endpoints.md

   Usage:
     - By number: /dr-move-plan 001 in-progress
     - By name: /dr-move-plan authentication in-progress
     - By file: /dr-move-plan @_claude/plans/draft/001-authentication-system.md in-progress
   ```

### Handle Multiple Matches

If multiple plan files match the search term:

1. Display all matching plans with their locations:
   ```
   ⚠️  Multiple plans match '[search term]':

   1) #001 - authentication-system (in draft/)
   2) #015 - oauth-authentication (in completed/)
   3) #023 - add-auth-logging (in in_progress/)

   Which plan did you mean? Please specify by:
     - Plan number: /dr-move-plan 001 in-progress
     - More specific name: /dr-move-plan authentication-system in-progress
     - File reference: /dr-move-plan @_claude/plans/draft/001-authentication-system.md in-progress
   ```

2. Use AskUserQuestion tool to ask which plan the user wants:
   - Question: "Which plan do you want to move?"
   - Options: List each matching plan (e.g., "#001 - authentication-system (draft)")

3. Once user selects, proceed to Phase 3 with the selected plan

### Phase 3: Validate and Execute Move

1. **Determine source and destination paths:**
   - Source: Current location of the plan file
   - Destination folder: Based on user-specified destination
   - Destination file: Same filename (preserving plan number)

2. **Check if already in destination:**
   - If source folder == destination folder, show message:
     ```
     ℹ️  Plan #[number] - [name] is already in [destination]/

     No action needed.
     ```
   - Exit without error

3. **Execute the move:**
   ```bash
   mv "_claude/plans/[source_folder]/[filename]" "_claude/plans/[destination_folder]/"
   ```

4. **Verify the move succeeded:**
   - Check that file exists in new location
   - Check that file no longer exists in old location

5. **Show success message:**
   ```
   ✅ Plan moved: #[number] - [Plan Name]

   From: _claude/plans/[source]/[filename]
   To:   _claude/plans/[destination]/[filename]

   Status: [Status message based on destination]
   ```

   Status messages:
   - `draft`: "Returned to draft for further refinement"
   - `in-progress`: "Ready for implementation"
   - `completed`: "Marked as complete"

6. **Add contextual next steps based on destination:**

   For `in-progress`:
   ```
   You can now begin working on this plan.
   Remember: Only work on plans in the in_progress/ folder.
   ```

   For `completed`:
   ```
   This plan is now archived for reference.
   Completed plans should not be modified.
   ```

   For `draft`:
   ```
   This plan has been returned to draft status.
   Review and refine before moving back to in-progress.
   ```

### Error Handling

**Invalid file reference:**
```
❌ Invalid file reference: [path]

The file must be:
  - A valid path in _claude/plans/ (draft/, in_progress/, or completed/)
  - A markdown file matching pattern XXX-*.md (e.g., 001-plan-name.md)

Example: /dr-move-plan @_claude/plans/draft/001-auth.md in-progress
```

**File not found:**
```
❌ File not found: [path]

Check that the file path is correct. Available plans:
  [List available plans by folder]
```

**Invalid destination:**
```
❌ Invalid destination: '[destination]'

Valid destinations:
  - draft       → _claude/plans/draft/
  - in-progress → _claude/plans/in_progress/
  - completed   → _claude/plans/completed/

Example: /dr-move-plan 001 in-progress
```

**Move failed:**
```
❌ Failed to move plan

Source: [source path]
Destination: [destination path]

Please check file permissions and try again.
```

### Usage Examples Shown When No Arguments

If user runs `/dr-move-plan` with no arguments:

```
📋 Move Plan Between Stages

Usage: /dr-move-plan [plan] [destination]

Plan can be specified as:
  - Plan number: 001, 42, 123
  - Plan name: authentication, user-auth (partial match supported)
  - File reference: @_claude/plans/draft/001-auth.md

Destination must be one of:
  - draft       → For plans being refined
  - in-progress → For plans ready to implement
  - completed   → For finished plans

Examples:
  /dr-move-plan 001 in-progress
  /dr-move-plan authentication completed
  /dr-move-plan @_claude/plans/draft/001-auth.md in-progress

Current plans:
  [List available plans]
```

### Important Notes

1. **NEVER guess which plan the user means** - If multiple matches, always ask for clarification
2. **PRESERVE the plan number** - The filename number prefix must remain unchanged
3. **Use forward slashes** for paths: `_claude/plans/draft/` not `_claude\plans\draft\`
4. **Handle empty folders gracefully** - Don't error if a folder has no plans
5. **Trim whitespace** from arguments before processing
6. **Normalize destination** - Accept `in-progress`, `in_progress`, or `inprogress` as equivalent

## Execute Now

Follow the instructions above to move the plan. Start with Phase 1: Parse Arguments.
