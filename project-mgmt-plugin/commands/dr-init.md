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
   - Read from plugin: `templates/CLAUDE-template.md`

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

3. **NEVER modify CLAUDE.md:**
   - Do not read it
   - Do not write to it
   - Leave it completely untouched

4. **Show appropriate feedback:**

   If nothing was missing:
   ```
   ✅ Project structure verified

   Directory structure: Already exists (✓)
   CLAUDE.md: Already exists (✓)

   Your project is already initialized.

   💡 Your CLAUDE.md is customized for your project. The plugin will never
      automatically modify it to preserve your customizations.

      To adopt new template features from plugin updates:
      1. Review plugin release notes
      2. Manually add desired improvements to your CLAUDE.md
   ```

   If folders were missing:
   ```
   ✅ Project structure updated

   Created missing directories:
     _claude/plans/in_progress/ (.gitkeep added)

   CLAUDE.md: Preserved (✓)

   Your existing CLAUDE.md has been preserved. Missing directories have been created.
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

   d. Read plugin template from `templates/CLAUDE-template.md`

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

   c. Read plugin template from `templates/CLAUDE-template.md`

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

1. **NEVER modify CLAUDE.md if it has the version marker** - This means the user has already customized it
2. **ALWAYS create .gitkeep files** - All 7 leaf directories need them
3. **ALWAYS check system date** - Never hardcode dates
4. **Be safe with existing content** - When in doubt, preserve user's work
5. **Folder paths must use forward slashes** - `_claude/plans/draft` not `_claude\plans\draft`

### Plugin Template Location

The template is located relative to the plugin root:
- `templates/CLAUDE-template.md`

Use the Read tool to access it.

## Execute Now

Follow the instructions above to initialize or update the project structure. Start with Phase 1: Detect Project State.
