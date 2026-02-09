---
description: Initialize project with standard directory structure
allowed-tools: Bash(mkdir:*), Bash(ls:*), Bash(tree:*), Bash(find:*), Bash(touch:*), Bash(wc:*), Bash(cat:*), Bash(sed:*), Bash(test:*), Bash(echo:*), Read, Write
---

# Initialize Project Management Structure

This command initializes a project with the standard directory structure and CLAUDE.md file for project management workflows.

## Instructions for Claude

You are executing the `/dr-init` command to set up or verify the project management structure.

### Phase 1: Detect Project State

**CRITICAL**: Before doing anything else, determine the current state of the project.

1. **Check if CLAUDE.md exists** in the current working directory
   - Use Read tool to try reading `CLAUDE.md`
   - If file not found error: CLAUDE.md does not exist
   - If file exists, check if it has content (more than just whitespace)
   - If empty or only whitespace: Treat as if CLAUDE.md does not exist

2. **If CLAUDE.md exists AND has content, check for version marker:**
   - Look for the comment `<!-- Plugin: project-management` near the top of the file
   - If found: Project has version marker
   - If not found: Project does NOT have version marker

3. **Check if `_claude/` directory exists:**
   - Use `ls -la _claude` to check
   - If exists and has subdirectories: `_claude/` structure exists
   - If not found: `_claude/` does NOT exist

4. **Determine project state based on findings:**

   - **STATE A (Fresh Project):**
     - No CLAUDE.md exists (or CLAUDE.md is empty/whitespace only)
     - No `_claude/` directory exists
     - ACTION: Create everything from scratch

   - **STATE B (Already Initialized):**
     - CLAUDE.md has version marker `<!-- Plugin: project-management`
     - OR `_claude/` directory exists with expected structure
     - ACTION: Verify completeness, create missing pieces, NEVER modify CLAUDE.md

   - **STATE C (Uninitialized):**
     - CLAUDE.md exists AND has actual content
     - NO version marker in CLAUDE.md
     - NO `_claude/` directory
     - ACTION: Offer user 3 options (append/show/cancel)

### Phase 2: Handle Each State

#### STATE A - Fresh Project

Execute these steps in order:

1. **Create directory structure:**
   ```bash
   mkdir -p _claude/docs
   mkdir -p _claude/plans/draft
   mkdir -p _claude/plans/in_progress
   mkdir -p _claude/plans/completed
   mkdir -p _claude/prd
   mkdir -p _claude/resources
   mkdir -p _claude/research
   ```

2. **Create .gitkeep files in ALL 7 leaf directories:**
   - `_claude/docs/.gitkeep`
   - `_claude/plans/draft/.gitkeep`
   - `_claude/plans/in_progress/.gitkeep`
   - `_claude/plans/completed/.gitkeep`
   - `_claude/prd/.gitkeep`
   - `_claude/resources/.gitkeep`
   - `_claude/research/.gitkeep`

   Use: `touch _claude/docs/.gitkeep` for each file

3. **Get current date:**
   - Check system environment for current date
   - Format as YYYY-MM-DD

4. **Read the CLAUDE.md template:**
   - Read from plugin: `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE-template.md`

5. **Replace placeholders in template:**
   - Replace `{{CURRENT_DATE}}` with actual current date from system

6. **Create CLAUDE.md:**
   - Write the processed template to `CLAUDE.md` in working directory

7. **Show success feedback:**
   ```
   ✅ Project structure initialized!

   Created:
     _claude/docs/ (.gitkeep added)
     _claude/plans/draft/ (.gitkeep added)
     _claude/plans/in_progress/ (.gitkeep added)
     _claude/plans/completed/ (.gitkeep added)
     _claude/prd/ (.gitkeep added)
     _claude/resources/ (.gitkeep added)
     _claude/research/ (.gitkeep added)
     CLAUDE.md (project guidelines)

   Next steps:
     1. Review and customize CLAUDE.md for your project
     2. Add project-specific commands (build, test, lint, etc.)
     3. Commit the new structure: git add _claude/ CLAUDE.md
     4. Create your first PRD: /dr-prd [feature description]
     5. Or conduct research: /dr-research [topic]

   💡 Note: CLAUDE.md is yours to customize. The plugin will never automatically modify it.
   ```

#### STATE B - Already Initialized

Execute these steps:

1. **Check which folders exist:**
   - List all directories in `_claude/`
   - Check for: docs, plans/draft, plans/in_progress, plans/completed, prd, resources, research

2. **Create any missing folders:**
   - If any folder is missing, create it with `mkdir -p`
   - Create .gitkeep file in any newly created leaf directory

3. **Verify CLAUDE.md sections (Tier 1 + Tier 2):**

   a. **Read the CLAUDE-template from the plugin:**
      - Read: `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE-template.md`
      - Extract all section version markers matching the pattern `<!-- section: [name] v[N] -->`
      - Build a list of versioned sections, each with:
        - Section heading (the `##` heading immediately above the marker)
        - Section name (from the marker, e.g., `task-completion-protocol`)
        - Current version number (from the marker, e.g., `1`)
        - Section content (from the heading through to the next `##` heading or end of plugin-managed area)

   b. **Read the user's CLAUDE.md:**
      - Read the full contents of `CLAUDE.md`

   c. **Tier 1 — Section existence check:**
      - For each versioned section from the template, check if its `##` heading exists in the user's CLAUDE.md
      - If the heading is not found → mark as **missing**

   d. **Tier 2 — Content correctness check:**
      - For sections that DO exist in the user's CLAUDE.md, search for the version marker comment (e.g., `<!-- section: task-completion-protocol v1 -->`)
      - If the marker is not found at all → mark as **outdated** (pre-versioning content)
      - If the marker is found but the version number is lower than the template's current version → mark as **outdated**
      - If the marker matches the current version → mark as **current**

   e. **Categorize all results:**
      - ✓ **Current** — Section exists with current version marker
      - ⚠ **Outdated** — Section exists but version marker is missing or old
      - ✗ **Missing** — Section heading not found

4. **Show verification results and handle updates:**

   **If all sections are current and no folders were missing:**
   ```
   ✅ Project structure verified

   Directory structure: Complete (✓)
   CLAUDE.md sections:
     ✓ Plan Management Workflow (current)
     ✓ Available Commands (current)
     ✓ Task Completion Protocol (current)

   Your project is fully up to date.
   ```
   - STOP — nothing to do

   **If any sections are outdated or missing:**

   Show the verification summary:
   ```
   ✅ Project structure verified

   Directory structure: [Complete/Updated — list any created folders]

   CLAUDE.md: [N] section(s) need attention

     ✓ Plan Management Workflow (current)
     ⚠ Task Completion Protocol (outdated — update available)
     ✗ [New Section Name] (missing)

   ```

   Then use AskUserQuestion to ask the user what to do, with these options:
   - **Update** — Apply changes to outdated/missing sections (preserves customizations in other sections)
   - **Show** — Display the new section content so you can manually update
   - **Skip** — Keep CLAUDE.md as-is

5. **Handle user choice:**

   **If user chooses "Update":**

   a. Read the user's CLAUDE.md content

   b. For each **outdated** section:
      - Locate the section in the user's CLAUDE.md (from its `##` heading to the next `##` heading)
      - Replace the entire section content with the current version from the template

   c. For each **missing** section:
      - Find the `<!-- End of plugin-managed section -->` comment in the user's CLAUDE.md
      - Insert the section from the template immediately before that comment
      - If the comment doesn't exist, append the section at the end of the file

   d. Write the updated CLAUDE.md

   e. Show confirmation:
      ```
      ✅ CLAUDE.md updated

      Sections updated:
        ⚠→✓ Task Completion Protocol (updated to v1)
      Sections added:
        ✗→✓ [New Section Name] (added)
      Sections preserved (unchanged):
        ✓ Plan Management Workflow
        [... all other sections]

      Your project-specific customizations have been preserved.
      ```

   **If user chooses "Show":**

   a. For each outdated or missing section, display the full section content from the template
   b. Wrap in a code fence so the user can copy/paste
   c. Show instructions:
      ```
      Copy the sections above into your CLAUDE.md to update.

      Run /dr-init again after updating to verify.
      ```

   **If user chooses "Skip":**
   ```
   ℹ️  CLAUDE.md not modified

   Run /dr-init again when you're ready to update.
   ```

#### STATE C - Uninitialized (Has CLAUDE.md but no plugin structure)

Execute these steps:

1. **Count lines in existing CLAUDE.md:**
   - Use `wc -l CLAUDE.md` or similar

2. **Show detection message with options:**
   ```
   ⚠️  CLAUDE.md exists but project is not initialized with plugin structure

   Current CLAUDE.md: [X] lines

   This appears to be a custom or minimal CLAUDE.md file. The project-management
   plugin can help set up the recommended structure.

   Options:
     [a] Append plugin template to your existing CLAUDE.md
         - Adds plugin sections to end of your file
         - Preserves all existing content
         - Creates _claude/ folder structure

     [s] Show plugin template (you copy/paste what you want)
         - Displays full template content
         - Creates _claude/ folder structure
         - You manually add desired sections

     [c] Cancel (do nothing)

   Choice [a/s/c]:
   ```

3. **Wait for user to type choice** (Use AskUserQuestion tool if available, otherwise inform user to respond)

4. **Handle user choice:**

   **If user chooses 'a' (Append):**

   a. Create `_claude/` folder structure and .gitkeep files (same as State A steps 1-2)

   b. Read existing CLAUDE.md content

   c. Get current date from system

   d. Read plugin template from `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE-template.md`

   e. Replace `{{CURRENT_DATE}}` with actual date

   f. Combine content:
      - Version marker comment at top
      - Original CLAUDE.md content
      - Separator line: `\n---\n\n`
      - Plugin template sections (everything after the comment block)

   g. Write combined content to CLAUDE.md

   h. Show feedback:
      ```
      ✅ Plugin structure added!

      Created:
        _claude/docs/ (.gitkeep added)
        _claude/plans/draft/ (.gitkeep added)
        _claude/plans/in_progress/ (.gitkeep added)
        _claude/plans/completed/ (.gitkeep added)
        _claude/prd/ (.gitkeep added)
        _claude/resources/ (.gitkeep added)
        _claude/research/ (.gitkeep added)

      Updated: CLAUDE.md
        - Added version tracking comment at top
        - Appended plugin template sections:
          • Project Structure explanation
          • Plan Management Workflow
          • Available Commands
          • Development principles

      Your existing content has been preserved at the beginning of the file.

      Next steps:
        1. Review CLAUDE.md - your content is at the top, plugin sections at the bottom
        2. Reorganize/merge sections as desired
        3. Commit the new structure: git add _claude/ CLAUDE.md
        4. Start using plugin commands: /dr-prd, /dr-plan, /dr-research
      ```

   **If user chooses 's' (Show):**

   a. Create `_claude/` folder structure and .gitkeep files (same as State A steps 1-2)

   b. Get current date from system

   c. Read plugin template from `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE-template.md`

   d. Replace `{{CURRENT_DATE}}` with actual date

   e. Display template content:
      ```
      ✅ Folder structure created!

      Created:
        _claude/docs/ (.gitkeep added)
        _claude/plans/draft/ (.gitkeep added)
        _claude/plans/in_progress/ (.gitkeep added)
        _claude/plans/completed/ (.gitkeep added)
        _claude/prd/ (.gitkeep added)
        _claude/resources/ (.gitkeep added)
        _claude/research/ (.gitkeep added)

      CLAUDE.md: Preserved (not modified)

      📋 Plugin template content (copy sections you want to your CLAUDE.md):

      ---BEGIN TEMPLATE---

      [Display full template content here]

      ---END TEMPLATE---

      Next steps:
        1. Review the template above
        2. Copy sections you want to your CLAUDE.md
        3. Commit the new structure: git add _claude/ CLAUDE.md
      ```

   **If user chooses 'c' (Cancel):**

   Show message:
   ```
   ℹ️  Cancelled - no changes made

   Your CLAUDE.md and project structure remain unchanged.

   Run /dr-init again when you're ready to set up the plugin structure.
   ```

### Important Notes

1. **STATE B verifies CLAUDE.md sections** - Use the template's version markers to detect outdated or missing sections, always ask before modifying
2. **ALWAYS create .gitkeep files** - All 7 leaf directories need them
3. **ALWAYS check system date** - Never hardcode dates
4. **Be safe with existing content** - When updating sections, only replace the specific outdated section, preserve everything else
5. **Folder paths must use forward slashes** - `_claude/plans/draft` not `_claude\plans\draft`

### Plugin Template Location

The template path uses the plugin root environment variable:
- `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE-template.md`

Use the Read tool to access it with the full path.

## Execute Now

Follow the instructions above to initialize or update the project structure. Start with Phase 1: Detect Project State.
