# Project Management Plugin - Implementation Plan

## Executive Summary

This plan outlines the creation of a reusable Claude Code plugin that brings structured project management practices to any codebase. The plugin provides standardized directory structures, research workflows, PRD creation, and implementation planning with consistent templates - all language and framework agnostic.

**Key Goals:**
- Create reusable project structure (`_claude/` with docs, plans, prd, resources, research)
- Provide explicit workflows for research, PRD writing, and plan creation
- Use templates to ensure consistency across projects
- Enable easy initialization and updates via `/dr-init` command

---

## Research Summary

### 1. Claude Code Plugin Architecture

**Plugin Components:**
Claude Code plugins support five component types:
- **Commands** (slash commands): Manual invocation, single markdown files
- **Agents**: Specialized subagents for complex tasks
- **Skills**: Autonomous capabilities Claude discovers and uses
- **Hooks**: Event-driven automation
- **MCP Servers**: Integration with external tools

**Plugin Structure:**
```
plugin-name/
├── .claude-plugin/
│   └── plugin.json           # Manifest with metadata
├── commands/                 # Slash commands (.md files)
├── agents/                   # Subagent definitions (optional)
├── skills/                   # Autonomous capabilities (optional)
├── hooks/                    # Event handlers (optional)
└── templates/                # Supporting template files
```

**Critical Requirements:**
- Component directories must be at plugin root (not nested in .claude-plugin/)
- plugin.json requires `name` field (kebab-case)
- Paths in manifest must be relative, starting with `./`

**Distribution:**
- Plugins can be distributed via marketplace catalogs
- Teams can configure repository-level settings for automatic installation
- Users install via `/plugin install plugin-name@marketplace-name`

### 2. Skills vs Slash Commands Decision Framework

**Slash Commands:**
- **Invocation**: Manual via `/command-name`
- **Structure**: Single markdown file
- **Arguments**: Supports `$ARGUMENTS`, `$1`, `$2`, etc.
- **Use Cases**: Explicit workflows, parameterized templates, repeated manual tasks
- **Discovery**: Listed in `/help` command

**Skills:**
- **Invocation**: Automatic discovery by Claude based on description
- **Structure**: Directory with SKILL.md + supporting files
- **Arguments**: Context-driven (no parameters)
- **Use Cases**: Autonomous capabilities, complex multi-step workflows
- **Discovery**: Claude matches task to description

**Decision Framework:**
| Aspect | Use Slash Command | Use Skill |
|--------|-------------------|-----------|
| Trigger | User explicitly invokes | Claude auto-discovers |
| Complexity | Single file sufficient | Multiple files needed |
| Control | User wants manual control | Claude should autonomously use |
| Parameters | Needs arguments | Context-driven |

### 3. CLAUDE.md Priority and File References

**Hierarchy (Highest to Lowest):**
1. Enterprise Policy (`/etc/claude-code/CLAUDE.md` or system-wide location)
2. Project Memory (`./CLAUDE.md` or `./.claude/CLAUDE.md`)
3. User Memory (`~/.claude/CLAUDE.md`)
4. Project Memory Local (`./CLAUDE.local.md` - deprecated)

**Key Findings:**
- Claude recursively searches from working directory upward
- Files can reference external files via `@path/to/file` syntax
- **Critical**: Referenced files inherit the priority of the importing file
- No priority difference between content in CLAUDE.md vs imported files
- Maximum import recursion depth: 5 hops
- Check loaded files using `/memory` command

**Implications:**
- No benefit to separating instructions into referenced files for priority reasons
- Separation only useful for organization or reusing same content in multiple places
- For plugin use case, keep core instructions in CLAUDE.md template for simplicity

### 4. Template Mechanisms Available

**Slash Command Templates:**
- `$ARGUMENTS`: Captures all arguments
- `$1`, `$2`, `$3`: Positional parameters
- `@file`: Include file contents
- `!command`: Execute bash before expansion

**File Imports:**
- `@path/to/file` syntax in CLAUDE.md
- `@~/.claude/file` for user-level files
- Relative and absolute paths supported

**Supporting Files in Skills:**
- Skills can have templates/ directories
- Files loaded only when skill is invoked
- Supports complex multi-file templates

**MCP Integration:**
- `${CLAUDE_PLUGIN_ROOT}` expands to plugin directory
- Enables portable script references

### 5. Plan Format Template Analysis

Based on analyzing completed plans from par-v2-migration, the consistent template structure includes:

**Required Sections:**
1. **Executive Summary** - 2-3 sentences: what, why, problem solved
2. **Current State** - Objective assessment of existing state
3. **Success Criteria** - Measurable checkboxes
4. **Implementation Plan** - Phase-based with tasks, time estimates, test verification
5. **Rollback Plan** - Simple revert steps
6. **Dependencies** - Prerequisites and requirements
7. **Success Metrics** - Final results with checkboxes

**Phase Structure:**
```markdown
### Phase N: [Clear Phase Name] [Status Icon]

**Estimated Time**: X hours (Actual: Y hours)

#### Tasks
- [ ] Specific, actionable task 1
- [ ] Specific, actionable task 2

#### Test Verification (if applicable)
- [ ] Specific test file or suite

#### Code Changes Needed (if applicable)
[Actual code examples]
```

**Formatting Conventions:**
- Status icons: ✅ (completed), 🔄 (in progress), none (not started)
- Checkboxes: `- [ ]` incomplete, `- [x]` completed
- Time tracking: Always estimated vs actual
- Code examples: Full implementations with syntax highlighting
- Test references: Specific file paths

---

## Architectural Decisions

### Decision 1: How to Make Directory Structure Reusable

**Problem:** Need a way to initialize projects with standard structure and update existing projects when plugin improves.

**Options Considered:**

**Option A: Skill that autonomously maintains structure**
- ❌ Too autonomous - creates directories without explicit permission
- ❌ Users may not want structure in all projects
- ❌ Hard to control when it activates

**Option B: Separate files with @ references in CLAUDE.md**
- ❌ No priority benefit (references inherit parent priority)
- ❌ More complex to maintain
- ✅ Slightly easier to update across projects
- ❌ Requires manual CLAUDE.md editing to add reference

**Option C: Slash command that generates/updates CLAUDE.md** ✅ SELECTED
- ✅ Explicit user control over initialization
- ✅ Can be re-run to update with plugin improvements
- ✅ Works for both new and existing projects
- ✅ Single command handles directory creation + CLAUDE.md
- ✅ Idempotent - safe to run multiple times

**Decision:** Create `/dr-init` slash command that:
1. Creates directory structure if missing
2. Generates or updates CLAUDE.md with standardized instructions
3. Preserves existing content while adding/updating plugin sections
4. Can be re-run as plugin evolves to gain new features

### Decision 2: Skills vs Slash Commands for Core Features

**Research Workflow:**
- **Decision:** Slash command (`/dr-research`)
- **Rationale:** Users explicitly decide when to conduct research, provide research topic as argument, want control over when research happens
- **Usage:** `/dr-research [topic]` creates `_claude/research/[topic-slug]/` with structured markdown files

**PRD Creation:**
- **Decision:** Slash command (`/dr-prd`)
- **Rationale:** Explicit document creation, user provides feature name, manual trigger point
- **Usage:** `/dr-prd [feature-name]` creates `_claude/prd/[feature-name].md` from template

**Plan Creation:**
- **Decision:** Slash command (`/dr-plan`)
- **Rationale:** Explicit document creation, uses consistent template, user decides when planning happens
- **Usage:** `/dr-plan [plan-name]` creates `_claude/plans/draft/[plan-name].md` with full template structure

**Summary:** All core features should be slash commands for explicit user control.

### Decision 3: CLAUDE.md vs Separate Referenced Files

**Problem:** Should plugin instructions live in CLAUDE.md or separate files referenced via @?

**Analysis:**
- Referenced files have same priority as importing file (no hierarchy benefit)
- Separation adds complexity without priority gain
- For directory structure instructions, always need them loaded
- Templates (plan format) should be separate for reusability by commands

**Decision:** Hybrid approach ✅
- **CLAUDE.md**: Core project structure instructions (always loaded, simple)
- **Separate template files**: Plan templates, PRD templates (used by slash commands)
- **No @ references**: Keep CLAUDE.md self-contained for simplicity

**Benefits:**
- CLAUDE.md remains the single source of truth for session instructions
- Templates are reusable by slash commands without duplication
- Simpler to understand and maintain
- Future updates only need to modify CLAUDE.md template

### Decision 4: Language-Agnostic Design

**Requirements:**
- No Elixir/Phoenix/Ash-specific instructions
- No framework assumptions
- No language-specific conventions
- Works for Python, JavaScript, Go, Rust, Java, etc.

**Approach:**
- Directory structure is language-neutral (docs, plans, prd, resources, research)
- Plan templates use generic "implementation" sections, not code-specific
- CLAUDE.md template has placeholders for project-specific commands
- Examples show multiple languages where applicable

### Decision 5: Plan Status Workflow

**Preserve from par-v2-migration:**
- Plans start in `_claude/plans/draft/` folder
- Move to `_claude/plans/in_progress/` when work begins
- Move to `_claude/plans/completed/` when finished
- Strong guidance: NEVER work on draft plans without moving them first

**Plugin Implementation:**
- `/dr-plan` creates in `draft/` by default
- `/dr-plan --in-progress` creates in `in_progress/` directly
- CLAUDE.md includes strict rules about not executing draft plans

---

## Plugin Structure

### Directory Layout

```
project-mgmt-plugin/
├── .claude-plugin/
│   └── plugin.json                          # Plugin manifest
├── commands/
│   ├── dr-init.md                           # Initialize/update project structure
│   ├── dr-research.md                       # Conduct structured research
│   ├── dr-prd.md                            # Create or refine PRD (dual-mode)
│   ├── dr-plan.md                           # Create or refine plan (dual-mode)
│   └── dr-move-plan.md                      # Move plan between stages
├── templates/
│   ├── CLAUDE.md.template                   # Project structure instructions
│   ├── plan-template.md                     # Implementation plan format
│   ├── prd-template.md                      # PRD format
│   └── research-index-template.md           # Research folder structure
└── README.md                                # Plugin documentation
```

### Plugin Manifest (plugin.json)

```json
{
  "name": "project-management",
  "version": "1.0.0",
  "description": "Structured project management with research, PRDs, and implementation plans",
  "author": {
    "name": "Your Name"
  },
  "keywords": [
    "project-management",
    "planning",
    "research",
    "prd",
    "documentation"
  ],
  "license": "MIT",
  "commands": [
    "./commands/dr-init.md",
    "./commands/dr-research.md",
    "./commands/dr-prd.md",
    "./commands/dr-plan.md",
    "./commands/dr-move-plan.md"
  ]
}
```

---

## Component Specifications

### 1. Slash Command: `/dr-init`

**Purpose:** Initialize project with standard directory structure and CLAUDE.md (one-time setup)

**Location:** `commands/dr-init.md`

**Frontmatter:**
```yaml
---
description: Initialize project with standard directory structure
allowed-tools: Bash(mkdir:*), Bash(ls:*), Read, Write
---
```

**Behavior:**
1. Check if `_claude/` directory exists
2. Create directory structure if missing:
   ```
   _claude/
   ├── docs/           # Technical documentation
   ├── plans/          # Implementation plans
   │   ├── draft/      # Plans being refined
   │   ├── in_progress/ # Active implementation
   │   └── completed/  # Finished plans
   ├── prd/            # Product requirement documents
   ├── resources/      # Reference materials
   └── research/       # Structured research output
   ```
3. **Create `.gitkeep` files in all leaf directories** (ensures git commits empty folders):
   - `_claude/docs/.gitkeep`
   - `_claude/plans/draft/.gitkeep`
   - `_claude/plans/in_progress/.gitkeep`
   - `_claude/plans/completed/.gitkeep`
   - `_claude/prd/.gitkeep`
   - `_claude/resources/.gitkeep`
   - `_claude/research/.gitkeep`
4. Check if CLAUDE.md exists
5. **If missing:** Create from template (with plugin version tracking)
6. **If exists:** Verify structure is complete, create any missing folders
   - **NEVER modify existing CLAUDE.md** (user-owned after creation)
   - Show friendly message that project is already initialized

**CLAUDE.md Template Content:**
- **Version tracking comment** (plugin version, generation date)
- Welcome message
- Directory structure explanation
- Commands available from plugin
- Plan workflow (draft → in_progress → completed)
- Strict rules about not executing draft plans
- Section for project-specific commands (placeholder)
- Development principles (general, not language-specific)
- **Note to users:** "This file is yours to customize. The plugin will never automatically modify it."

**User Feedback (First Run):**
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

**User Feedback (Subsequent Runs):**
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

**User Feedback (Missing Folders, CLAUDE.md Exists):**
```
✅ Project structure updated

Created missing directories:
  _claude/plans/in_progress/ (.gitkeep added)

CLAUDE.md: Preserved (✓)

Your existing CLAUDE.md has been preserved. Missing directories have been created.
```

### 2. Slash Command: `/dr-research`

**Purpose:** Conduct deep, comprehensive research on a topic using extended thinking and document findings in organized markdown files

**Location:** `commands/dr-research.md`

**Frontmatter:**
```yaml
---
description: Conduct deep research on a topic with extended thinking
argument-hint: [detailed research prompt - can be multi-line]
allowed-tools: WebSearch, WebFetch, Read, Write, Bash(mkdir:*)
---
```

**Usage:**
- **With prompt:** `/dr-research [full detailed research prompt]` (supports multi-line)
- **Interactive:** `/dr-research` (Claude will ask for details)

**Examples:**
```bash
/dr-research I need to understand OAuth 2.1 implementation patterns
in modern frameworks. Focus on PKCE flow, token rotation, and
security best practices. Compare approaches in Express.js, FastAPI,
and Phoenix/Elixir. I'm particularly concerned about refresh token
storage and XSS attack vectors.
```

**Behavior:**
1. **Check for arguments**: If `$ARGUMENTS` is empty, ask user for detailed research prompt interactively
2. **CRITICAL: Check current date/time** using system environment before any document creation
3. **Use extended thinking**: Take time to deeply analyze the research request, identify key questions, and plan comprehensive research approach
4. Extract core topic and create slug (e.g., "OAuth 2.1 Implementation" → "oauth-2.1-implementation")
5. Create timestamp using actual current date (YYYY-MM-DD format)
6. Create directory: `_claude/research/[slug]-[timestamp]/`
7. Conduct thorough research using WebSearch, WebFetch as needed
8. Structure comprehensive findings into multiple markdown files:
   - `index.md` - Overview and navigation
   - `findings.md` - Main research findings
   - `resources.md` - Links and references
   - `recommendations.md` - Actionable recommendations
   - Additional files as needed for complex topics
6. Use markdown links between files for navigation

**Directory Structure Created:**
```
_claude/research/oauth-implementation-2025-01-05/
├── index.md              # Overview with links to other files
├── findings.md           # Main research content
├── resources.md          # External links, documentation
├── recommendations.md    # Actionable next steps
└── [additional-files].md # Topic-specific deep-dives
```

**index.md Template:**
```markdown
# Research: [Topic]

**Date:** [Current date from system - YYYY-MM-DD format]
**Research Question:** [Original question/topic]

## Overview

[1-2 paragraph summary of research]

## Structure

This research is organized into multiple documents:

- **[Findings](./findings.md)** - Core research findings and analysis
- **[Resources](./resources.md)** - Links, documentation, and references
- **[Recommendations](./recommendations.md)** - Actionable recommendations

## Key Takeaways

1. [Key takeaway 1]
2. [Key takeaway 2]
3. [Key takeaway 3]
```

**Command Structure in Markdown:**
The command file should instruct Claude to:
- Check if `$ARGUMENTS` is provided
- If not, ask for detailed research prompt
- Use extended thinking to analyze the request
- Conduct comprehensive research
- Create well-structured, thorough documentation

**User Feedback:**
```
✅ Research completed: [topic]

Created: _claude/research/[slug]-[timestamp]/
  - index.md (overview and navigation)
  - findings.md (comprehensive research findings)
  - resources.md (links and references)
  - recommendations.md (actionable next steps)
  - [additional topic-specific files as needed]

Key Findings:
  - [Bullet point summary of 3-5 most important findings]

Next steps:
  - Review detailed findings in _claude/research/[slug]-[timestamp]/
  - Create PRD if ready: /dr-prd [feature description]
  - Create implementation plan: /dr-plan [plan description]
```

### 3. Slash Command: `/dr-prd`

**Purpose:** Create comprehensive Product Requirement Document OR refine existing PRD with thoughtful analysis

**Location:** `commands/dr-prd.md`

**Frontmatter:**
```yaml
---
description: Create or refine a comprehensive PRD with extended thinking
argument-hint: [feature description OR @prd-file [refinement request]] [--no-confirm]
allowed-tools: Read, Write, Edit, Bash(ls:*), Bash(cp:*), Grep
---
```

**Dual-Mode Operation:**
This command operates in two modes based on the arguments:
1. **CREATE mode**: When no file reference is provided, creates a new PRD
2. **REFINE mode**: When `@path/to/prd.md` is provided, refines existing PRD

**Usage:**

**CREATE Mode:**
- **With prompt:** `/dr-prd [full feature description and context]` (supports multi-line)
- **Interactive:** `/dr-prd` (Claude will ask for details)

**REFINE Mode:**
- **Basic:** `/dr-prd @_claude/prd/feature.md [refinement request]`
- **Skip confirmation:** `/dr-prd @_claude/prd/feature.md [refinement] --no-confirm`

**Examples:**

**Creating a new PRD:**
```bash
/dr-prd We need to add real-time collaboration features to our document editor.
Users should be able to see each other's cursors, make simultaneous edits,
and see changes in real-time. This is for a team of 5-50 users per document.
We're concerned about conflict resolution and performance at scale. Our backend
is Node.js with PostgreSQL, frontend is React.

# Creates: _claude/prd/realtime-collaboration-features.md
```

**Refining an existing PRD:**
```bash
# Add MFA requirement to authentication PRD
/dr-prd @_claude/prd/authentication-system.md Add multi-factor authentication requirement with TOTP and SMS backup. Should support 99.9% of users enabling MFA.

# Clarify performance requirements
/dr-prd @_claude/prd/notification-system.md Clarify that "real-time" means sub-500ms delivery latency, not instant delivery.

# Adjust scope based on technical constraints
/dr-prd @_claude/prd/file-upload.md Reduce max file size from 500MB to 100MB due to API gateway limitations discovered during planning.

# Quick refinement without confirmation
/dr-prd @_claude/prd/search-feature.md Fix typo in success metrics section --no-confirm
```

**Behavior:**

**Mode Detection:**
1. Check if `$ARGUMENTS` starts with `@`
2. If yes → **REFINE mode** (go to Refinement Behavior below)
3. If no → **CREATE mode** (continue with steps below)

**CREATE Mode Behavior:**
1. **Check for arguments**: If `$ARGUMENTS` is empty, ask user for detailed feature description interactively
2. **CRITICAL: Check current date** using system environment before document creation
3. **Use extended thinking**: Deeply analyze the feature request, think through user needs, edge cases, technical considerations, and potential challenges
4. Extract feature name and create slug
5. Read PRD template from plugin
6. Create file: `_claude/prd/[feature-slug].md`
7. **Thoughtfully populate all sections** based on the prompt:
   - Problem statement with deep understanding
   - Well-defined goals and metrics
   - Comprehensive user stories with acceptance criteria
   - Detailed functional and non-functional requirements
   - Technical considerations and constraints
   - Risk analysis and mitigation strategies
8. Ask clarifying questions if critical information is missing

**REFINE Mode Behavior:**
1. **Parse arguments:**
   - Extract file reference (`@path/to/prd.md`)
   - Extract refinement request (text after file reference, before flags)
   - Parse flags: `--no-confirm` (skip confirmation prompt)

2. **Validate PRD file:**
   - Verify file exists in `_claude/prd/` directory
   - Confirm file has PRD structure (has required sections)
   - If validation fails: show error with available PRDs

3. **Read and parse existing PRD:**
   - Extract metadata (Status, Version, Created, Last Updated)
   - Parse all sections
   - Identify current structure and content

4. **Detect status and check for warnings:**
   - **Draft**: Proceed with refinement normally
   - **Under Review**: Show mild warning ("stakeholders may be reviewing")
   - **Approved**: Show strong warning + check for linked plans
   - **Superseded**: Refuse with message ("create new version instead")

5. **Check for linked plans:**
   - Search all plan files for references to this PRD: `grep -r "@PRD_PATH" _claude/plans/`
   - If found: List all plans that reference this PRD
   - Include in confirmation message

6. **Create automatic backup:**
   - Copy PRD to `.{filename}.backup` in same directory
   - Example: `auth-system.md` → `.auth-system.md.backup`
   - Overwrite backup if exists (keep only most recent)

7. **Use extended thinking to analyze:**
   - Deeply understand the existing PRD (problem, requirements, scope)
   - Analyze the refinement request (what changes, why, implications)
   - Plan how to integrate changes while preserving structure
   - Consider impact on linked plans if any

8. **Generate refined PRD:**
   - Apply requested changes thoughtfully
   - Preserve PRD structure and format
   - Update metadata:
     - Increment version (1.0 → 1.1 for minor, 1.1 → 2.0 for major)
     - Update "Last Updated" to current date
     - Keep "Created" date unchanged
     - Maintain or update Status if appropriate
   - Add entry to "Refinement History" section:
     - Current date
     - Brief description of what was refined
   - Maintain all other metadata

9. **Generate diff summary:**
   - Identify additions (new sections, requirements, content)
   - Identify modifications (changed sections, updated content)
   - Identify deletions (removed content, if any)
   - Show what's preserved
   - Provide clear, terminal-friendly summary

10. **Request confirmation (unless --no-confirm):**
    - Display diff summary
    - If linked plans found: Show warning about plans
    - Show: "Apply these changes? [y/n/diff]"
    - If user enters **y**: Proceed to step 11
    - If user enters **n**: Cancel, no changes made
    - If user enters **diff**: Show detailed line-by-line diff, then ask again
    - If --no-confirm flag present: Skip to step 11

11. **Apply changes:**
    - Write refined PRD to original file (atomic operation)
    - Preserve file permissions

12. **Confirm success:**
    - Show success message with PRD name and version change
    - Show backup location
    - List what changed (summary)
    - If linked plans exist: Remind user to review them
    - Suggest next steps

**PRD Template Structure:**
```markdown
# PRD: [Feature Name]

**Status:** Draft
**Version:** 1.0
**Created:** [Current date from system - YYYY-MM-DD format]
**Author:** [Name or "Claude Code"]
**Last Updated:** [Current date from system - YYYY-MM-DD format]

## Problem Statement

[What problem does this feature solve? Who experiences this problem?]

## Goals and Objectives

### Primary Goals
- [ ] Goal 1
- [ ] Goal 2

### Success Metrics
- Metric 1: [How measured]
- Metric 2: [How measured]

## User Stories

### As a [user type]
I want [capability]
So that [benefit]

**Acceptance Criteria:**
- [ ] Criterion 1
- [ ] Criterion 2

## Functional Requirements

### Must Have
1. Requirement 1
2. Requirement 2

### Should Have
1. Requirement 1
2. Requirement 2

### Nice to Have
1. Requirement 1

## Non-Functional Requirements

- **Performance:** [Requirements]
- **Security:** [Requirements]
- **Scalability:** [Requirements]
- **Accessibility:** [Requirements]

## Technical Considerations

[Architecture notes, technology choices, constraints]

## Dependencies

- [ ] Dependency 1
- [ ] Dependency 2

## Timeline and Milestones

| Milestone | Target Date | Status |
|-----------|-------------|--------|
| Phase 1   | [Date]      | [ ]    |
| Phase 2   | [Date]      | [ ]    |

## Risks and Mitigation

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Risk 1 | High | Medium | Strategy |

## Open Questions

- [ ] Question 1
- [ ] Question 2

## References

- [Research documents]
- [Related PRDs]
- [External resources]

---

## Refinement History

(This section is populated by `/dr-prd` refinement mode)

**Version 1.0** - [Date]
- Initial PRD creation
```

**Command Structure in Markdown:**
The command file should instruct Claude to:
- Detect mode (CREATE vs REFINE) based on `@` in arguments
- **CREATE mode:**
  - Check if `$ARGUMENTS` is provided
  - If not, ask for detailed feature description
  - Use extended thinking to analyze requirements
  - Create comprehensive PRD with all sections thoughtfully filled in
  - Ask clarifying questions only if critical gaps exist
- **REFINE mode:**
  - Parse file reference and refinement request
  - Validate PRD file exists
  - Check status and linked plans
  - Use extended thinking to analyze existing PRD and requested changes
  - Create backup
  - Generate diff summary
  - Request confirmation (unless --no-confirm)
  - Apply changes and update metadata

**User Feedback (CREATE mode):**
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
  - [3-5 important points Claude identified while analyzing]

Next steps:
  1. Review and refine the PRD
  2. Discuss with stakeholders
  3. Refine if needed: /dr-prd @_claude/prd/[feature-slug].md [changes]
  4. Create implementation plan: /dr-plan [implementation context]
```

**User Feedback (REFINE mode - Success):**
```
✅ PRD refined successfully

PRD: Authentication System (v1.0 → v1.1)
Location: _claude/prd/authentication-system.md
Status: Draft

Changes applied:
  + Added: Multi-Factor Authentication requirement section
    - TOTP support (authenticator apps)
    - SMS backup codes
    - Recovery codes for account access
  + Added: Success metric - 95% MFA adoption within 6 months
  ~ Modified: Security Requirements - added MFA mention
  ~ Modified: Timeline - extended by 2 weeks for MFA implementation

Backup saved: _claude/prd/.authentication-system.md.backup

Next steps:
  1. Review the refined PRD
  2. Update status if stakeholder re-approval needed
  3. Refine again if needed: /dr-prd @_claude/prd/authentication-system.md [more changes]
  4. Create or update implementation plans
```

**User Feedback (REFINE mode - With Linked Plans Warning):**
```
✅ PRD refined successfully

PRD: Authentication System (v2.0 → v2.1)
Location: _claude/prd/authentication-system.md
Status: Approved (⚠️ May need stakeholder re-approval)

Changes applied:
  + Added: OAuth provider support (Google, GitHub, Microsoft)
  ~ Modified: Timeline - adjusted for OAuth integration
  ~ Modified: Technical Considerations - OAuth token management

Backup saved: _claude/prd/.authentication-system.md.backup

⚠️  This PRD is referenced by 2 plans:
  - _claude/plans/draft/001-implement-auth-core.md
  - _claude/plans/in_progress/003-oauth-integration.md

Recommended actions:
  1. Review PRD changes and update status if needed
  2. Review each linked plan for alignment with updated requirements
  3. Consider refining plans that may be affected:
     /dr-plan @_claude/plans/draft/001-implement-auth-core.md Align with PRD v2.1 OAuth requirements
```

**User Feedback (REFINE mode - Approved PRD Warning):**
```
⚠️  WARNING: This PRD is Approved

PRD: Authentication System
Current Version: 2.0
Status: Approved

Your requested change: "Add biometric authentication support"

This PRD has been approved by stakeholders. Changes may require:
  - Stakeholder re-approval
  - Updates to linked implementation plans (2 plans found)

Referenced by plans:
  - 001-implement-auth-core.md (draft)
  - 003-oauth-integration.md (in progress)

Consider:
  1. Moving status back to "Under Review" for significant changes
  2. Creating a new major version (v3.0) for major scope changes
  3. Continuing with minor refinements only

Continue with changes to Approved PRD? [y/n]:
```

**User Feedback (REFINE mode - Superseded PRD Refusal):**
```
❌ Cannot refine Superseded PRD

PRD: Authentication System (v1.0)
Status: Superseded
Superseded by: authentication-system-v2.md (v2.0)

This is a historical version and should not be modified.

Instead:
  1. Refine the current version: /dr-prd @_claude/prd/authentication-system-v2.md [changes]
  2. Create a new major version if needed: /dr-prd [new requirements based on v2]
```

### 4. Slash Command: `/dr-plan`

**Purpose:** Create comprehensive implementation plan OR refine existing plan with deep analysis

**Location:** `commands/dr-plan.md`

**Frontmatter:**
```yaml
---
description: Create or refine a detailed implementation plan with extended thinking
argument-hint: [implementation context OR @plan-file [refinement request]] [--no-confirm|--in-progress]
allowed-tools: Read, Write, Edit, Grep, Bash(ls:*), Bash(find:*), Bash(cp:*)
---
```

**Dual-Mode Operation:**
This command operates in two modes based on the arguments:
1. **CREATE mode**: When no `@plan-file` reference is provided, creates a new plan
2. **REFINE mode**: When `@path/to/plan.md` is provided, refines existing plan

**Usage:**

**CREATE Mode:**
- **With prompt:** `/dr-plan [full implementation context]` (supports multi-line)
- **With PRD reference:** `/dr-plan [context] @_claude/prd/feature.md` (links to PRD document)
- **With flag:** `/dr-plan [context] --in-progress` (creates in in_progress/ instead of draft/)
- **Interactive:** `/dr-plan` (Claude will ask for details)

**REFINE Mode:**
- **Basic:** `/dr-plan @plan-file [refinement request]`
- **Skip confirmation:** `/dr-plan @plan-file [refinement request] --no-confirm`

**Examples:**

**Creating a new plan:**
```bash
# Basic plan without PRD
/dr-plan Implement user authentication system with email/password and OAuth.
Need to support password reset, email verification, session management,
and rate limiting. Backend is Express.js with PostgreSQL. Security is
critical - must prevent common attacks. We have about 3 weeks for this.
Current codebase has no auth system, starting from scratch.

# Creates: _claude/plans/draft/001-implement-user-authentication-system.md

# Plan with PRD reference
/dr-plan Implement the user authentication system as specified in the PRD.
@_claude/prd/user-authentication-system.md
Focus on the core flows first, then add OAuth providers in phase 2.
Backend is Express.js with PostgreSQL.

# Creates: _claude/plans/draft/002-implement-user-authentication-system.md
# (with "Related PRD: _claude/prd/user-authentication-system.md" in metadata)
```

**Refining an existing plan:**
```bash
# Add OAuth support to existing auth plan
/dr-plan @_claude/plans/draft/001-authentication-system.md Add OAuth 2.0 support with Google and GitHub providers. Include token refresh logic and profile data fetching.

# Add more detail to specific phase
/dr-plan @_claude/plans/draft/001-auth.md Add detailed code examples for password hashing with Argon2 and security best practices checklist to Phase 3

# Minor adjustment to in-progress plan
/dr-plan @_claude/plans/in_progress/003-database-migration.md Add Redis 7.x requirement to Dependencies section for session storage

# Skip confirmation for quick updates
/dr-plan @_claude/plans/draft/002-api.md Fix typo in Phase 2 task description --no-confirm
```

**Behavior:**

**Mode Detection:**
1. Check if `$ARGUMENTS` contains `@_claude/plans/` (plan file reference)
2. If yes → **REFINE mode** (go to Refinement Behavior below)
3. If no → **CREATE mode** (continue with steps below)

**CREATE Mode Behavior:**
1. **Check for arguments**: If `$ARGUMENTS` is empty, ask user for detailed implementation context interactively
2. **Parse for file references**: Check for `@path/to/prd.md` syntax in arguments
   - If PRD reference found, extract the file path
   - Verify the file exists and read its contents
   - Use PRD content to inform plan creation
   - Store PRD reference path for metadata section
3. **Parse for --in-progress flag**: Determines destination folder
4. **CRITICAL: Check current date** using system environment before document creation
5. **Determine plan number** (critical - must scan ALL folders):
   - Use `ls` or `find` to list all files in draft/, in_progress/, AND completed/ folders
   - Parse ALL filenames matching pattern `XXX-*.md` to extract numbers
   - Find the highest number across all three folders (plans may all be in completed/)
   - Increment highest number by 1 for new plan
   - Format as 3-digit number with leading zeros (001, 002, ..., 999)
   - If over 999, use number as-is (1000, 1001, etc.)
   - If no existing plans found in any folder, start at 001
   - **Example**: If completed/ has 001-050 and in_progress/ has 051-052, new plan is 053
6. **Use extended thinking**: Deeply analyze the implementation requirements
   - Break down into logical phases
   - Identify all tasks and sub-tasks
   - Consider dependencies and ordering
   - Think through technical challenges
   - Estimate time realistically
   - Identify risks and rollback strategies
7. **Optionally read existing code**: If appropriate, examine relevant files to understand current state
8. **If PRD referenced**: Read and analyze the PRD document to ensure plan aligns with requirements
9. Extract plan name and create slug
10. Determine destination folder (draft/ or in_progress/)
11. Read plan template from plugin
12. Create file: `_claude/plans/[folder]/[number]-[plan-slug].md` (e.g., `001-authentication-system.md`)
13. **Thoughtfully populate all sections**:
    - Executive summary with clear problem statement
    - Current state analysis (check actual code if needed)
    - Specific, measurable success criteria
    - Detailed phase breakdown with realistic tasks
    - Test verification steps for each phase
    - Dependencies identified
    - Comprehensive rollback plan

**REFINE Mode Behavior:**
1. **Parse arguments:**
   - Extract file reference (`@path/to/plan.md`)
   - Extract refinement request (text after file reference, before flags)
   - Parse flags: `--no-confirm` (skip confirmation prompt)

2. **Validate plan file:**
   - Verify file exists and is accessible
   - Confirm file is in a plan folder (draft/, in_progress/, or completed/)
   - Verify file matches plan naming pattern (XXX-*.md)
   - If validation fails: show helpful error with available plans

3. **Detect plan status:**
   - Determine which folder: draft/, in_progress/, or completed/
   - If **completed/**: Refuse with message "Completed plans are historical records and should not be modified. Create a new plan incorporating lessons learned instead."
   - If **in_progress/** and major changes requested: Warn user and suggest moving to draft first
   - If **draft/**: Proceed with all requested changes

4. **Read and analyze existing plan** using extended thinking:
   - Parse plan structure (metadata, sections, phases)
   - Understand current approach and reasoning
   - Identify what works well and should be preserved
   - Note any gaps or weaknesses
   - Consider plan context (PRD references, dependencies, etc.)

5. **Analyze refinement request** using extended thinking:
   - What specific changes are being requested?
   - Is this targeted improvement or major restructuring?
   - What sections need to change vs. remain unchanged?
   - What are the implications of this refinement?
   - For in_progress plans: Is this a minor clarification or major change?

6. **Create automatic backup:**
   - Before any changes, copy plan to `.{filename}.backup`
   - Example: `001-auth.md` → `.001-auth.md.backup`
   - Backup stays in same directory
   - Only keep most recent backup (overwrite if exists)

7. **Generate refined plan:**
   - Apply requested changes thoughtfully
   - Preserve plan number, slug, and filename
   - Maintain plan structure and format
   - Update metadata:
     - Increment refinement count or set to "1" if first refinement
     - Keep creation date unchanged
     - Status remains the same
   - Add entry to Refinement History section:
     - Current date
     - Brief description of what was refined
   - Preserve Related PRD reference if present
   - Maintain all formatting conventions

8. **Generate diff summary:**
   - Compare original vs. refined plan
   - Categorize changes:
     - **ADDITIONS:** New phases, tasks, sections, content
     - **MODIFICATIONS:** Changed content in existing sections
     - **DELETIONS:** Removed content (if any)
     - **PRESERVED:** What remains unchanged
   - Provide clear, readable summary suitable for terminal display
   - Estimate scope: minor changes vs. major restructuring

9. **Confirmation (unless --no-confirm):**
   - Display diff summary
   - Show: "Apply these changes? [y/n/diff]"
   - If user enters **y**: Proceed to step 10
   - If user enters **n**: Cancel, no changes made
   - If user enters **diff**: Show detailed line-by-line diff, then ask again
   - If --no-confirm flag present: Skip this step entirely

10. **Apply changes:**
    - Write refined plan to original file
    - Atomic operation (write to temp, then move)
    - Preserve file permissions

11. **Confirm success:**
    - Show success message with:
      - Plan number and name
      - What was changed (summary)
      - Backup location
      - Next steps suggestion

**Plan Template Structure:**
(Based on analyzed format from par-v2-migration)

```markdown
# [Plan Name]

**Created:** [YYYY-MM-DD]
**Status:** [Draft/In Progress/Completed]
**Related PRD:** [Path to PRD if referenced, otherwise "N/A"]
**Refinements:** [Number of times refined, or "None"]

## Executive Summary

[2-3 sentences: what is being implemented, why it matters, what problem it solves]

## Current State

- [What exists now]
- [What is broken or incomplete]
- [Specific TODOs or placeholders]
- [Test status if applicable]

## Success Criteria

- [ ] Specific deliverable 1
- [ ] Specific deliverable 2
- [ ] Specific deliverable 3
- [ ] Test outcomes (e.g., "X tests passing")

## Implementation Plan

### Phase 1: [Clear Phase Name]

**Estimated Time:** [X hours]

#### Tasks
- [ ] Specific, actionable task 1
- [ ] Specific, actionable task 2
- [ ] Specific, actionable task 3

#### Test Verification
- [ ] Specific test file or suite to verify
- [ ] Expected test results

#### Code Changes Needed
```[language]
// Example code showing the implementation
```

### Phase 2: [Clear Phase Name]

**Estimated Time:** [X hours]

#### Tasks
- [ ] Specific, actionable task 1
- [ ] Specific, actionable task 2

#### Test Verification
- [ ] Specific test file or suite

### Phase N: Final Steps

**Estimated Time:** [X hours]

#### Tasks
- [ ] Documentation updates
- [ ] Final testing
- [ ] Review and cleanup

## Rollback Plan

1. Simple revert step 1
2. Simple revert step 2
3. Simple revert step 3

## Dependencies

- [ ] Dependency 1 (with specific details)
- [ ] Dependency 2 (with specific details)
- [ ] Other plans that must complete first

## Success Metrics

(To be filled in after implementation)

- [ ] Metric 1 with actual results
- [ ] Metric 2 with actual results
- [ ] Final test count
- [ ] Performance metrics if applicable

---

## Refinement History

(Optional section - populated by `/dr-plan` refine mode)

**Refinements:**
- [YYYY-MM-DD]: [Brief description of what was refined]
- [YYYY-MM-DD]: [Brief description of what was refined]

---

## Implementation Notes

**Actual Time Tracking:**
- Phase 1: [Estimated: X hours] (Actual: Y hours)
- Phase 2: [Estimated: X hours] (Actual: Y hours)

**Key Decisions:**
- [Document important decisions made during implementation]

**Lessons Learned:**
- [Document what worked well and what didn't]
```

**Command Structure in Markdown:**
The command file should instruct Claude to:
- Detect mode (CREATE vs REFINE) based on `@_claude/plans/` in arguments
- **CREATE mode:**
  - Check if `$ARGUMENTS` is provided
  - If not, ask for detailed implementation context
  - Parse for `@path/to/prd.md` file references
  - If PRD referenced, read and analyze it to inform plan creation
  - Parse for `--in-progress` flag
  - Use extended thinking to break down the implementation
  - Optionally examine current codebase to understand state
  - Create comprehensive plan with all phases and tasks detailed
  - Include PRD reference path in metadata if provided
  - Provide realistic time estimates
- **REFINE mode:**
  - Parse file reference and refinement request
  - Validate plan file exists and is in correct location
  - Detect plan status and apply appropriate behavior
  - Use extended thinking to analyze existing plan deeply
  - Use extended thinking to understand refinement request
  - Create automatic backup before any changes
  - Generate refined plan with intelligent changes
  - Produce clear diff summary
  - Request confirmation (unless --no-confirm)
  - Apply changes atomically
  - Update refinement metadata and history

**User Feedback (CREATE mode):**
```
✅ Implementation plan created: _claude/plans/[folder]/[number]-[plan-slug].md

Plan #[number]: [plan name]
Location: _claude/plans/draft/[number]-[plan-slug].md
Status: Draft (not ready for implementation)
[If PRD referenced: Related PRD: _claude/prd/[prd-name].md]

Comprehensive plan includes:
  - Executive summary and current state analysis
  [If PRD referenced: - Requirements aligned with PRD specifications]
  - [N] phases with detailed tasks
  - Estimated time: [X] hours total
  - Test verification for each phase
  - Dependencies and rollback plan
  - Success metrics

Key Insights:
  - [3-5 important considerations Claude identified]
  [If PRD referenced: - [Key requirements from PRD that shaped the plan]]

Next steps:
  1. Review the detailed implementation plan
  [If PRD referenced: 2. Verify alignment with PRD requirements]
  2. Refine if needed: /dr-plan @_claude/plans/draft/[number]-[plan-slug].md [changes]
  3. When ready to implement: /dr-move-plan [plan-name] in-progress
  4. IMPORTANT: Only work on plans in 'in_progress' folder!

Note: Plan created in draft/ - move to in_progress/ before implementing.
```

**User Feedback (REFINE mode - Success with draft plan):**
```
✅ Plan refined successfully

Plan #001: Authentication System
Location: _claude/plans/draft/001-authentication-system.md
Status: Draft (ready for review)

Changes applied:
  + Added Phase 2.5: OAuth Integration (4 hours)
  + Added dependency: OAuth provider registration
  ~ Modified Phase 4: Testing - Added OAuth test cases

Backup saved: _claude/plans/draft/.001-authentication-system.md.backup

Refinement count: 2 (see Refinement History in plan)

Next steps:
  1. Review the refined plan
  2. Refine again if needed: /dr-plan @_claude/plans/draft/001-authentication-system.md [changes]
  3. When ready: /dr-move-plan 001 in-progress
```

**User Feedback (REFINE mode - Warning for in-progress plan with major changes):**
```
⚠️  WARNING: This plan is in progress (may have completed tasks).

Current status: in_progress
Your request: "Completely redesign to use OAuth-first with optional email/password"

This appears to be a major structural change that could invalidate completed work.

Recommendations:
  1. Move plan back to draft for major redesign: /dr-move-plan 003 draft
  2. Create a new plan for the OAuth-first approach: /dr-plan [new approach]
  3. Continue with minor adjustments only

Proceed with major changes to in-progress plan? [y/n]:
```

**User Feedback (REFINE mode - Refusal for completed plan):**
```
❌ Cannot refine completed plan

Plan #015: Authentication System
Status: Completed (historical record)

Completed plans are archived for historical reference and should not be modified.

Instead:
  1. Create a new plan incorporating lessons learned
  2. Reference the completed plan: /dr-plan [new context] "Building on plan 015..."
  3. Document what would be done differently in the new plan
```

**User Feedback (REFINE mode - Error: file not found):**
```
❌ Plan file not found: _claude/plans/draft/999-nonexistent.md

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

### 5. Slash Command: `/dr-move-plan`

**Purpose:** Move plan between draft, in_progress, and completed folders (preserves plan number)

**Location:** `commands/dr-move-plan.md`

**Frontmatter:**
```yaml
---
description: Move a plan between draft, in_progress, and completed
argument-hint: [plan-name-or-number-or-@file] [draft|in-progress|completed]
allowed-tools: Bash(mv:*), Bash(ls:*), Read
---
```

**Usage:** `/dr-move-plan [plan-name-or-number-or-@file] [destination]`

**Examples:**
- `/dr-move-plan 001 in-progress` - Move plan #001 to active work (exact number match)
- `/dr-move-plan authentication in-progress` - Move to active work (searches by name)
- `/dr-move-plan auth in-progress` - Partial match (may ask for clarification if multiple "auth" plans exist)
- `/dr-move-plan @_claude/plans/draft/001-authentication-system.md in-progress` - Move using file reference (exact file)
- `/dr-move-plan 042 completed` - Mark plan #042 as done
- `/dr-move-plan authentication-system draft` - Move back to draft (full name match)

**Behavior:**
1. Parse argument - could be plan number (001), plan name/slug, or file reference (@path)
2. **If file reference (`@path`) provided:**
   - Extract the file path from the `@` reference
   - Verify the file exists and is a plan file (matches pattern `XXX-*.md`)
   - Verify the file is in one of the three plan folders (draft/, in_progress/, or completed/)
   - Extract plan number and name from the filename
   - Proceed directly to step 5 (skip search - we have exact file)
3. **Otherwise, search for plan file in all three folders:**
   - If number provided: Look for files starting with that number (e.g., `001-*.md`)
   - If name provided: Search for files containing that name (partial match allowed)
4. **Handle search results:**
   - **No matches found**: Show error message listing all available plans
   - **Exactly one match**: Proceed to step 5
   - **Multiple matches found**:
     - Display all matching plans with their numbers, names, and current locations
     - Ask user: "Multiple plans match '[search term]'. Which plan did you mean?"
     - List options: "1) 001-authentication-system (draft/), 2) 015-oauth-authentication (completed/), ..."
     - Suggest using file reference for precision: "Or use: /dr-move-plan @_claude/plans/draft/001-authentication-system.md [destination]"
     - Wait for user to select the correct plan by number or clarify their search
     - Once clarified, proceed to step 5
5. Confirm source and destination with full plan details
6. Move file to new location **preserving the number prefix**
7. Update any internal status markers if present
8. Confirm move to user with before/after locations

**User Feedback:**

**Success (single match):**
```
✅ Plan moved: #001 - Authentication System

From: _claude/plans/draft/001-authentication-system.md
To:   _claude/plans/in_progress/001-authentication-system.md

Status: Ready for implementation

You can now begin working on this plan.
```

**Ambiguous (multiple matches):**
```
⚠️  Multiple plans match 'auth':

1) #001 - authentication-system (in draft/)
2) #015 - oauth-authentication (in completed/)
3) #023 - add-auth-logging (in in_progress/)

Which plan did you mean? Please specify by:
  - Plan number: /dr-move-plan 001 in-progress
  - More specific name: /dr-move-plan authentication-system in-progress
  - File reference: /dr-move-plan @_claude/plans/draft/001-authentication-system.md in-progress
```

**No match:**
```
❌ No plans found matching 'xyz'

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

---

## Templates

### Template: CLAUDE.md.template

```markdown
<!--
  Plugin: project-management v1.0.0
  Template generated: [YYYY-MM-DD from system date]

  This file is yours to customize for your project.
  The plugin will never automatically modify it after creation.

  To adopt new template features from future plugin versions:
  - Review plugin release notes
  - Manually add desired improvements to this file
-->

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

This project follows a structured approach to planning, documentation, and implementation.

```
_claude/
├── docs/           # Technical documentation and architecture
│   └── .gitkeep
├── plans/          # Implementation plans
│   ├── draft/      # Plans being developed or refined
│   │   └── .gitkeep
│   ├── in_progress/ # Plans currently being implemented
│   │   └── .gitkeep
│   └── completed/  # Finished and archived plans
│       └── .gitkeep
├── prd/            # Product Requirement Documents
│   └── .gitkeep
├── resources/      # Reference materials and external docs
│   └── .gitkeep
└── research/       # Structured research output
    └── .gitkeep
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
   - Automatically creates backup before changes
   - Shows diff summary before applying
4. **Move to Active**: `/dr-move-plan [plan-number-or-name] in-progress` when ready to implement
5. **Implement**: Work through plan phases systematically
6. **Minor Adjustments** (as needed): `/dr-plan @_claude/plans/in_progress/001-plan.md [minor changes]` for small corrections
7. **Complete**: `/dr-move-plan [plan-number-or-name] completed` when finished

**Plan Numbering:**
Plans are automatically numbered sequentially (001, 002, 003, ..., 999, 1000, ...) to track chronological order. The number is determined by scanning **all three folders** (draft/, in_progress/, completed/) to find the highest existing number, then incrementing by 1. The number stays with the plan when moved between folders.

Example: If your completed/ folder has plans 001-045 and in_progress/ has 046-047, the next plan created will be 048, even if draft/ is empty.

## Available Commands

This project uses the **project-management** plugin (dr- prefix) which provides:

- `/dr-init` - Initialize or update project structure
- `/dr-research [detailed prompt]` - Conduct deep research with extended thinking (supports multi-line prompts)
- `/dr-prd [detailed feature description OR @prd-file [refinement]]` - Create or refine comprehensive PRD with extended thinking
- `/dr-plan [detailed context OR @plan-file [refinement]]` - Create or refine implementation plan with extended thinking (dual-mode)
- `/dr-move-plan [plan-number-or-name] [stage]` - Move plan between stages (preserves number)

**Dual-Mode Refinement:**
Both PRDs and plans can be refined using the same commands. Use `/dr-prd @_claude/prd/feature.md [changes]` to refine PRDs or `/dr-plan @_claude/plans/draft/plan.md [changes]` to refine plans. Both commands use extended thinking, create automatic backups, and show diff summaries. They automatically detect whether you're creating or refining based on the `@` file reference.

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

When working on tasks from a plan phase:

1. **Complete the implementation** of the tasks
2. **Verify completion** by:
   - Testing the functionality (if applicable)
   - Confirming all requirements are met
   - Checking for any remaining work
3. **Report to the user** with a summary of what was completed
4. **Wait for user confirmation** before marking tasks as complete in the plan
5. **Update the plan** only after user confirms the tasks are done
6. **Then ask** about next steps

This ensures proper tracking and prevents premature task completion marking.

---

<!-- End of plugin-managed section -->
<!-- Add project-specific instructions below -->
```

---

## Implementation Plan

### Phase 1: Core Plugin Structure ✅ (Planning Complete)

**Estimated Time:** 2 hours

#### Tasks
- [x] Research Claude Code plugin architecture
- [x] Analyze existing plan formats
- [x] Make architectural decisions
- [x] Create comprehensive plugin plan document

#### Deliverables
- [x] plugin-plan.md with research and specifications

### Phase 2: Plugin Foundation ✅

**Estimated Time:** 2 hours

#### Tasks
- [x] Create plugin directory structure
- [x] Write plugin.json manifest
- [x] Create README.md with plugin documentation
- [x] Set up templates/ directory
- [x] Create .gitignore for plugin

#### Deliverables
- [x] project-mgmt-plugin/ with proper structure
- [x] plugin.json with metadata
- [x] README.md with installation and usage instructions

#### Success Criteria
- [x] Plugin directory follows Claude Code standards
- [x] Manifest is valid JSON with required fields
- [x] Documentation is clear and comprehensive

### Phase 3: CLAUDE.md Template

**Estimated Time:** 1 hour

#### Tasks
- [ ] Extract language-agnostic content from par-v2-migration CLAUDE.md
- [ ] Create CLAUDE.md.template in templates/
- [ ] Add plugin-specific instructions
- [ ] Add placeholders for project-specific content
- [ ] Include plan workflow rules

#### Deliverables
- [ ] templates/CLAUDE.md.template

#### Success Criteria
- [ ] Template is language-agnostic
- [ ] Includes all directory structure explanations
- [ ] Has clear plan workflow rules
- [ ] Includes placeholders for customization

### Phase 4: Plan Template

**Estimated Time:** 1.5 hours

#### Tasks
- [ ] Create plan-template.md based on analyzed format
- [ ] Include all standard sections
- [ ] Add phase structure with checkboxes
- [ ] Include time tracking fields
- [ ] Add placeholders and instructions
- [ ] Test template with example content

#### Deliverables
- [ ] templates/plan-template.md

#### Success Criteria
- [ ] Template matches analyzed format from par-v2-migration
- [ ] All sections present with clear structure
- [ ] Checkboxes formatted correctly
- [ ] Easy to fill in with actual content

### Phase 5: Supporting Templates

**Estimated Time:** 2 hours

#### Tasks
- [ ] Create prd-template.md
- [ ] Create research-index-template.md
- [ ] Create research section templates (findings, resources, recommendations)
- [ ] Add example content to each template
- [ ] Ensure consistency across templates

#### Deliverables
- [ ] templates/prd-template.md
- [ ] templates/research-index-template.md
- [ ] templates/research-findings-template.md
- [ ] templates/research-resources-template.md
- [ ] templates/research-recommendations-template.md

#### Success Criteria
- [ ] All templates are comprehensive
- [ ] Examples are clear and helpful
- [ ] Formatting is consistent

### Phase 6: Slash Command - /dr-init

**Estimated Time:** 3 hours

#### Tasks
- [ ] Create commands/dr-init.md
- [ ] Add frontmatter with description and allowed-tools
- [ ] Write command logic:
  - [ ] Check for existing _claude/ directory
  - [ ] Create directory structure if missing (docs, plans/draft, plans/in_progress, plans/completed, prd, resources, research)
  - [ ] **Create .gitkeep files in ALL leaf directories** (7 files total):
    - [ ] _claude/docs/.gitkeep
    - [ ] _claude/plans/draft/.gitkeep
    - [ ] _claude/plans/in_progress/.gitkeep
    - [ ] _claude/plans/completed/.gitkeep
    - [ ] _claude/prd/.gitkeep
    - [ ] _claude/resources/.gitkeep
    - [ ] _claude/research/.gitkeep
  - [ ] Check if CLAUDE.md exists
  - [ ] **If CLAUDE.md missing:** Read template from plugin and create CLAUDE.md (with plugin version comment)
  - [ ] **If CLAUDE.md exists:** NEVER modify it, show friendly message that project is already initialized
  - [ ] Verify structure is complete, create any missing folders/gitkeep files
  - [ ] Provide appropriate user feedback based on what was created
- [ ] Test on new project (verify .gitkeep files created)
- [ ] Test on existing project with CLAUDE.md
- [ ] Test in git repo: verify empty directories are tracked after git add

#### Deliverables
- [ ] commands/dr-init.md

#### Success Criteria
- [ ] Creates directory structure correctly
- [ ] **Creates .gitkeep files in all 7 leaf directories**
- [ ] .gitkeep files allow empty directories to be committed to git
- [ ] Generates CLAUDE.md from template (first run only)
- [ ] CLAUDE.md includes plugin version comment for tracking
- [ ] **NEVER modifies existing CLAUDE.md** (user-owned after creation)
- [ ] Safe to run multiple times (creates missing pieces, preserves CLAUDE.md)
- [ ] Shows appropriate feedback for first run vs. subsequent runs
- [ ] Clear user feedback mentioning git commit (first run)
- [ ] Friendly "already initialized" message (subsequent runs)
- [ ] Works in both git and non-git repositories

### Phase 7: Slash Command - /dr-research

**Estimated Time:** 4 hours

#### Tasks
- [ ] Create commands/dr-research.md
- [ ] Add frontmatter with description and multi-line argument support
- [ ] Write command that instructs Claude to:
  - [ ] Check if `$ARGUMENTS` is provided, if not ask interactively
  - [ ] **FIRST: Check system date/time** (critical requirement)
  - [ ] Use extended thinking to analyze research request
  - [ ] Extract topic and create slug using ACTUAL current date
  - [ ] Create research directory
  - [ ] Conduct thorough research using WebSearch/WebFetch
  - [ ] Structure comprehensive findings into multiple files
  - [ ] Create cross-references between files
  - [ ] Generate index.md with navigation and current date
  - [ ] Provide summary of key findings
- [ ] Test with detailed multi-line research prompt
- [ ] Test interactive mode (no arguments)

#### Deliverables
- [ ] commands/dr-research.md

#### Success Criteria
- [ ] Accepts multi-line prompts via `$ARGUMENTS`
- [ ] Falls back to interactive mode if no arguments
- [ ] Uses extended thinking for deep analysis
- [ ] Creates proper directory structure with correct dates
- [ ] Generates multiple linked markdown files
- [ ] Research is comprehensive and well-organized
- [ ] Index provides clear navigation with key findings summary
- [ ] Works with various topic types and complexity levels

### Phase 8: Slash Command - /dr-prd

**Estimated Time:** 6 hours (includes refinement functionality)

#### Tasks
- [ ] Create commands/dr-prd.md
- [ ] Add frontmatter with description, argument support, and tools for refinement (Grep, Edit, Bash)
- [ ] Write command that implements dual-mode operation:
  - [ ] **Mode detection**: Check if `$ARGUMENTS` starts with `@` (refine mode) or not (create mode)
  - [ ] **CREATE mode:**
    - [ ] Check if `$ARGUMENTS` is provided, if not ask interactively
    - [ ] **FIRST: Check system date/time** (critical requirement)
    - [ ] Use extended thinking to analyze feature requirements
    - [ ] Think through user needs, edge cases, technical challenges
    - [ ] Extract feature name and create slug
    - [ ] Read PRD template
    - [ ] Create PRD file with ALL sections thoughtfully populated using ACTUAL current date
    - [ ] Include Version 1.0 and Refinement History section
    - [ ] Ask clarifying questions only if critical gaps exist
    - [ ] Provide summary of key considerations identified
  - [ ] **REFINE mode:**
    - [ ] Parse file reference and refinement request
    - [ ] Parse `--no-confirm` flag
    - [ ] Validate PRD file exists in `_claude/prd/`
    - [ ] Read and parse existing PRD (metadata, sections)
    - [ ] Check status (Draft/Approved/Superseded) and warn appropriately
    - [ ] Search for linked plans using grep
    - [ ] Create automatic backup (`.{filename}.backup`)
    - [ ] Use extended thinking to analyze existing PRD and requested changes
    - [ ] Generate refined PRD with updated metadata (version, date, history)
    - [ ] Generate diff summary
    - [ ] Request confirmation (unless --no-confirm)
    - [ ] Apply changes atomically
    - [ ] Confirm success with backup location and next steps
- [ ] Test CREATE mode with detailed multi-line feature description
- [ ] Test CREATE mode interactive (no arguments)
- [ ] Test REFINE mode with Draft PRD
- [ ] Test REFINE mode with Approved PRD (should warn)
- [ ] Test REFINE mode with Superseded PRD (should refuse)
- [ ] Test REFINE mode with linked plans (should detect and warn)
- [ ] Test --no-confirm flag
- [ ] Test backup creation and restoration

#### Deliverables
- [ ] commands/dr-prd.md (with dual-mode implementation)

#### Success Criteria
- [ ] **CREATE mode:**
  - [ ] Accepts multi-line prompts via `$ARGUMENTS`
  - [ ] Falls back to interactive mode if no arguments
  - [ ] Uses extended thinking for requirements analysis
  - [ ] Creates PRD with ALL sections meaningfully filled in
  - [ ] Includes Version and Refinement History sections
  - [ ] Filename follows naming convention with correct date
  - [ ] Identifies key considerations and risks
- [ ] **REFINE mode:**
  - [ ] Detects `@file` reference and switches to refine mode
  - [ ] Validates PRD file and shows helpful errors
  - [ ] Uses extended thinking for refinement analysis
  - [ ] Creates automatic backups
  - [ ] Detects and warns about linked plans
  - [ ] Respects PRD status (Draft/Approved/Superseded)
  - [ ] Shows clear diff summary
  - [ ] Requires confirmation (unless --no-confirm)
  - [ ] Updates version and refinement history
  - [ ] Provides clear success feedback
- [ ] Both modes work seamlessly from same command

### Phase 9: Slash Command - /dr-plan

**Estimated Time:** 8 hours (includes dual-mode create + refine functionality)

#### Tasks
- [ ] Create commands/dr-plan.md
- [ ] Add frontmatter with description, multi-line argument support, and dual-mode tools (Edit, Bash(cp:*))
- [ ] Support PRD references via `@path/to/prd.md` syntax
- [ ] Write command that implements dual-mode operation:
  - [ ] **Mode detection**: Check if `$ARGUMENTS` contains `@_claude/plans/` (plan file reference)
  - [ ] If yes → REFINE mode, if no → CREATE mode
  - [ ] **CREATE mode:**
  - [ ] Check if `$ARGUMENTS` is provided, if not ask interactively
  - [ ] **Parse for PRD file references** (`@path/to/prd.md` syntax)
  - [ ] If PRD referenced, verify file exists and read its contents
  - [ ] Use PRD content to inform plan creation (requirements, scope, etc.)
  - [ ] Parse for `--in-progress` flag
  - [ ] **FIRST: Check system date/time** (critical requirement)
  - [ ] **Scan ALL plan folders** (draft, in_progress, completed) using ls or find - MUST check all three
  - [ ] Collect all filenames from all three folders
  - [ ] Parse filenames to extract numbers (handle 001-xxx.md, 042-xxx.md formats)
  - [ ] Find the single highest number across all folders combined
  - [ ] Determine next plan number (increment highest + 1, start at 001 if none exist anywhere)
  - [ ] Format number with leading zeros (001-999) or as-is if over 999
  - [ ] Handle edge case: all plans in completed/, draft and in_progress empty
  - [ ] Use extended thinking to analyze implementation requirements
  - [ ] Break down into logical phases with realistic tasks
  - [ ] Identify dependencies and potential challenges
  - [ ] Estimate time for each phase
  - [ ] Optionally examine current codebase for context
  - [ ] Extract plan name and create slug
  - [ ] Determine destination folder (draft/ or in_progress/)
  - [ ] Read plan template
  - [ ] Create plan file with numbered format: `[number]-[slug].md` with ALL sections thoughtfully populated using ACTUAL current date
  - [ ] If PRD referenced, include path in plan metadata
  - [ ] Provide summary of key insights and considerations including plan number
  - [ ] **REFINE mode:**
    - [ ] Parse file reference (`@path/to/plan.md`)
    - [ ] Parse refinement request (text after file reference, before flags)
    - [ ] Parse `--no-confirm` flag
    - [ ] Validate plan file exists and is in a plan folder
    - [ ] Detect plan status (draft/in_progress/completed)
    - [ ] If completed: Refuse with message (historical records)
    - [ ] If in_progress + major changes: Warn user, suggest moving to draft
    - [ ] Read and analyze existing plan using extended thinking
    - [ ] Analyze refinement request using extended thinking
    - [ ] Create automatic backup (`.{filename}.backup`)
    - [ ] Generate refined plan (preserve number, update metadata, add refinement history)
    - [ ] Generate diff summary (additions, modifications, deletions, preserved)
    - [ ] Unless --no-confirm: Ask for confirmation (y/n/diff)
    - [ ] Apply changes atomically
    - [ ] Confirm success with backup location
- [ ] Test CREATE mode with detailed multi-line implementation context
- [ ] **Test with PRD reference** (e.g., `/dr-plan [context] @_claude/prd/feature.md`)
- [ ] Test PRD file validation (nonexistent file, invalid path)
- [ ] Test with --in-progress flag
- [ ] Test interactive mode (no arguments)
- [ ] Test numbering: verify correct number when existing plans present
- [ ] Test numbering: verify starts at 001 when no plans exist
- [ ] Test numbering edge case: all plans in completed/, new plan gets correct next number
- [ ] Test numbering edge case: plans spread across all three folders, finds highest correctly
- [ ] Test numbering: verify scans ALL three folders (not just draft/)
- [ ] Test REFINE mode with draft plan
- [ ] Test REFINE mode with in_progress plan + minor changes
- [ ] Test REFINE mode with in_progress plan + major changes (should warn)
- [ ] Test REFINE mode with completed plan (should refuse)
- [ ] Test --no-confirm flag in refine mode
- [ ] Test backup creation and restoration
- [ ] Test diff summary accuracy
- [ ] Test confirmation flow (y/n/diff options)

#### Deliverables
- [ ] commands/dr-plan.md (with dual-mode implementation)

#### Success Criteria
**CREATE mode:**
- [ ] Accepts multi-line prompts via `$ARGUMENTS`
- [ ] **Supports PRD references** via `@path/to/prd.md` syntax
- [ ] Reads and analyzes PRD content when referenced
- [ ] Includes PRD path in plan metadata section
- [ ] Validates PRD file exists before proceeding
- [ ] Falls back to interactive mode if no arguments
- [ ] Uses extended thinking for implementation breakdown
- [ ] **Automatically numbers plans sequentially** (001, 002, 003...)
- [ ] **Scans ALL three folders** (draft, in_progress, completed) to find highest existing number
- [ ] Finds highest number even if all plans are in completed/ folder
- [ ] Formats numbers with leading zeros (001-999)
- [ ] Handles 4+ digit numbers correctly (1000+)
- [ ] Creates plans in correct folder (respects --in-progress flag)
- [ ] Filename format: `[number]-[slug].md`
- [ ] Plan has ALL sections meaningfully filled with detailed phases and tasks
- [ ] Phases are logical, tasks are specific and actionable
- [ ] Time estimates are realistic
- [ ] Dependencies and risks identified
- [ ] Can optionally read codebase for current state analysis
- [ ] Clear warnings about draft vs in-progress status
- [ ] User feedback includes plan number

**REFINE mode:**
- [ ] Detects plan file reference and switches to refine mode
- [ ] Validates plan file exists and is in correct location
- [ ] Uses extended thinking for refinement analysis
- [ ] Creates automatic backups
- [ ] Respects plan status (Draft/In-Progress/Completed)
- [ ] Draft plans: all changes allowed
- [ ] In-progress plans: warns about major changes
- [ ] Completed plans: refuses to modify
- [ ] Shows clear diff summary
- [ ] Requires confirmation (unless --no-confirm)
- [ ] Preserves plan number and slug
- [ ] Updates refinement history
- [ ] Provides clear success feedback

**Both modes:**
- [ ] Work seamlessly from same command
- [ ] Automatic mode detection via `@_claude/plans/` reference

### Phase 10: Slash Command - /dr-move-plan

**Estimated Time:** 4 hours

#### Tasks
- [ ] Create commands/dr-refine-plan.md
- [ ] Add frontmatter with description and argument support
- [ ] Write command that instructs Claude to:
  - [ ] Parse arguments for file reference (`@path/to/plan.md`)
  - [ ] Parse refinement request (text after file reference, before flags)
  - [ ] Parse flags: `--no-confirm` (skip confirmation)
  - [ ] **Validate plan file exists** and is in a plan folder
  - [ ] **Detect plan status** (draft, in_progress, or completed)
  - [ ] **Read and analyze existing plan** using extended thinking
  - [ ] **Analyze refinement request** using extended thinking
  - [ ] **If plan is completed**: Refuse with message (historical records shouldn't change)
  - [ ] **If plan is in_progress** and major changes requested: Warn user, suggest moving to draft
  - [ ] **Create automatic backup**: `.{filename}.backup` before any changes
  - [ ] **Generate refined plan**:
    - Preserve plan number, slug, and core metadata
    - Apply requested changes intelligently
    - Maintain plan structure and format
    - Update "Last Refined" timestamp
    - Add refinement note to metadata
  - [ ] **Generate diff summary**:
    - List additions (new phases, tasks, sections)
    - List modifications (changed content)
    - List deletions (if any)
    - Show what's preserved
  - [ ] **Unless --no-confirm**: Ask for confirmation
    - Options: y (yes), n (no), diff (show detailed diff first)
    - If diff: show line-by-line changes, then ask again
  - [ ] **Apply changes**: Write refined plan to original file
  - [ ] **Confirm success**: Show what was changed and where backup is
- [ ] Test basic refinement (add content to plan)
- [ ] Test backup creation and restoration
- [ ] Test confirmation flow (y/n/diff options)
- [ ] Test --no-confirm flag
- [ ] Test with draft plan (should work smoothly)
- [ ] Test with in_progress plan + minor changes (should work)
- [ ] Test with in_progress plan + major changes (should warn)
- [ ] Test with completed plan (should refuse)
- [ ] Test diff summary accuracy
- [ ] Test file reference with autocomplete
- [ ] Test error handling (file not found, not a plan file, invalid path)

#### Deliverables
- [ ] commands/dr-refine-plan.md

#### Success Criteria
- [ ] Accepts file reference via `@path/to/plan.md` syntax
- [ ] Uses extended thinking to analyze existing plan and refinement request
- [ ] Creates automatic backup before any changes
- [ ] Generates clear, readable diff summary
- [ ] Requires confirmation before applying changes (unless --no-confirm)
- [ ] Preserves plan number and slug
- [ ] Status-aware: different behavior for draft/in_progress/completed
- [ ] Draft plans: allows all changes
- [ ] In-progress plans: warns about major changes
- [ ] Completed plans: refuses to modify
- [ ] Applies changes intelligently while preserving plan structure
- [ ] Clear user feedback showing what changed
- [ ] Handles errors gracefully with helpful messages
- [ ] Works with autocomplete for file paths
- [ ] Maintains plan quality equivalent to original creation

### Phase 10: Slash Command - /dr-move-plan

**Estimated Time:** 2.5 hours

#### Tasks
- [ ] Create commands/dr-move-plan.md
- [ ] Add frontmatter with description and argument handling (including file references)
- [ ] Write command logic:
  - [ ] Parse argument (could be plan number "001", plan name, or file reference "@path")
  - [ ] **If file reference**: Extract path, verify file exists and is valid plan file
  - [ ] **Otherwise search** for plan in all folders:
    - [ ] If number: Match files starting with that number (e.g., `001-*.md`)
    - [ ] If name: Search for files containing that name/slug (allow partial matches)
  - [ ] **Handle multiple matches**: Display list and ask user to clarify
  - [ ] **Handle no matches**: Show helpful error with all available plans
  - [ ] Validate destination folder
  - [ ] Move file **preserving the number prefix**
  - [ ] Confirm to user with plan number, name, and both locations
  - [ ] Handle errors gracefully (plan not found, already in destination, invalid file reference, etc.)
- [ ] Test moving by plan number (e.g., `/dr-move-plan 001 in-progress`)
- [ ] Test moving by plan name (e.g., `/dr-move-plan authentication in-progress`)
- [ ] **Test moving by file reference** (e.g., `/dr-move-plan @_claude/plans/draft/001-auth.md in-progress`)
- [ ] Test file reference with autocomplete
- [ ] Test moving between all folder combinations
- [ ] Test error handling (invalid plan, invalid destination, invalid file path)
- [ ] **Test ambiguous matches**: Verify displays all matches and asks for clarification
- [ ] **Test partial name matching**: Verify "auth" finds "authentication-system"
- [ ] **Test no match**: Verify shows helpful list of all available plans
- [ ] Test invalid file reference (file not in plan folders, file doesn't exist)

#### Deliverables
- [ ] commands/dr-move-plan.md

#### Success Criteria
- [ ] Accepts three argument types: plan number (001), plan name, or file reference (@path)
- [ ] **File reference support**: Can use `@_claude/plans/draft/001-plan.md` for exact file specification
- [ ] File reference works with autocomplete in terminal/editor
- [ ] Supports partial name matching (e.g., "auth" matches "authentication-system")
- [ ] Finds plans regardless of current location
- [ ] **Handles ambiguous matches**: Lists all matches, asks user to clarify, suggests file reference option
- [ ] **Handles no matches**: Shows all available plans with helpful guidance
- [ ] Moves files correctly **preserving number prefix**
- [ ] Confirmation shows plan number, name, source and destination
- [ ] Handles edge cases (plan not found, already in destination, ambiguous names, invalid file path)
- [ ] Clear, informative, user-friendly error and confirmation messages
- [ ] Never moves a plan without being certain which one the user meant

### Phase 11: Documentation and Testing

**Estimated Time:** 3 hours

#### Tasks
- [ ] Complete README.md with:
  - [ ] Installation instructions
  - [ ] Usage examples for each command
  - [ ] Directory structure explanation
  - [ ] Workflow guidelines
  - [ ] Troubleshooting
- [ ] Create CHANGELOG.md
- [ ] Test plugin end-to-end:
  - [ ] Fresh project initialization
  - [ ] Complete workflow (init → research → prd → plan)
  - [ ] Moving plans between stages
  - [ ] Re-running init on existing project
- [ ] Document any issues found
- [ ] Create example project using plugin

#### Deliverables
- [ ] Complete README.md
- [ ] CHANGELOG.md
- [ ] Tested plugin
- [ ] Example project

#### Success Criteria
- [ ] All commands work as specified
- [ ] Documentation is clear and complete
- [ ] Example project demonstrates workflow
- [ ] No critical bugs

### Phase 12: Distribution Preparation

**Estimated Time:** 2 hours

#### Tasks
- [ ] Create marketplace.json for distribution
- [ ] Add license file (MIT recommended)
- [ ] Create contributing guidelines
- [ ] Set up version tagging
- [ ] Test installation via marketplace
- [ ] Create announcement/introduction document

#### Deliverables
- [ ] .claude-plugin/marketplace.json
- [ ] LICENSE
- [ ] CONTRIBUTING.md
- [ ] Version 1.0.0 tag
- [ ] Distribution package

#### Success Criteria
- [ ] Plugin installs correctly via `/plugin install`
- [ ] All metadata is accurate
- [ ] License is appropriate
- [ ] Ready for public distribution

---

## Success Metrics

### Plugin Completeness
- [ ] All 5 slash commands implemented and working (dual-mode for PRD and plan)
- [ ] All 5 templates created and tested
- [ ] Plugin installs without errors
- [ ] Documentation is comprehensive

### Functionality
- [ ] `/dr-init` creates directory structure and CLAUDE.md
- [ ] `/dr-research` conducts research and creates multi-file documentation
- [ ] `/dr-prd` creates PRD from template OR refines existing PRD (dual-mode)
- [ ] `/dr-plan` creates implementation plan OR refines existing plan (dual-mode)
- [ ] `/dr-move-plan` moves plans between stages

### Refinement Capabilities (Dual-Mode)
- [ ] `/dr-prd` detects existing files and switches to refine mode
- [ ] PRD refinement uses extended thinking for analysis
- [ ] PRD refinement creates automatic backups
- [ ] PRD refinement shows diff summary before applying changes
- [ ] PRD refinement checks for linked plans and warns user
- [ ] PRD refinement respects status (Draft/Approved/Superseded)
- [ ] `/dr-plan` detects existing plan files and switches to refine mode
- [ ] Plan refinement uses extended thinking for analysis
- [ ] Plan refinement creates automatic backups
- [ ] Plan refinement shows diff summary before applying changes
- [ ] Plan refinement maintains plan number and structure
- [ ] Plan refinement respects status (Draft/In-Progress/Completed)
- [ ] Both commands support --no-confirm flag in refine mode

### Usability
- [ ] Commands have clear descriptions in `/help`
- [ ] User feedback is informative
- [ ] Error messages are helpful
- [ ] Workflow is intuitive

### Quality
- [ ] Language-agnostic (works for any tech stack)
- [ ] Templates are consistent and professional
- [ ] No hardcoded paths or assumptions
- [ ] Handles edge cases gracefully

### Distribution
- [ ] Plugin can be installed via marketplace
- [ ] README has clear instructions
- [ ] Version is tagged appropriately
- [ ] License is included

---

## Rollback Plan

1. Plugin is additive - can be uninstalled without affecting project files
2. `/dr-init` preserves existing content
3. Generated files can be deleted manually if needed
4. No destructive operations in any command

---

## Dependencies

- [ ] Claude Code v1.0+ installed
- [ ] Understanding of slash command syntax
- [ ] Access to plugin directory for development

---

## Future Enhancements (Post v1.0)

### v1.1 Considerations
- [ ] `/dr-archive-plan` - Archive old completed plans
- [ ] Status badges in plan files
- [ ] Plan template variations (small change, large feature, migration)

### v1.2 Considerations
- [ ] Hooks for automatic plan status updates
- [ ] MCP integration for external project management tools
- [ ] Custom template support per project
- [ ] Plan metrics and analytics

### v2.0 Considerations
- [ ] Skills for autonomous plan creation
- [ ] Agent for code review against plans
- [ ] Interactive plan refinement
- [ ] Plan dependency visualization

---

## Lessons from par-v2-migration

### What to Preserve
✅ Directory structure with clear separation
✅ Plan workflow with draft/in_progress/completed
✅ Strict rules about not executing draft plans
✅ Plan template with phases, checkboxes, time tracking
✅ Research documentation approach
✅ Sequential plan numbering for chronological tracking

### What to Generalize
🔄 Remove Elixir/Phoenix/Ash specific content
🔄 Remove specific framework constraints (Argon2, binary IDs, etc.)
🔄 Make commands work for any language
🔄 Remove agent definitions (backend-developer, graphql-architect)
🔄 Generalize error handling patterns

### What to Add
➕ PRD workflow (missing from original)
➕ PRD-Plan linking via `@file` references in `/dr-plan`
➕ Explicit research command with extended thinking
➕ Template management system
➕ Move-plan command for workflow management
➕ Better init/update mechanism
➕ Automatic sequential plan numbering (001, 002, etc.)
➕ Command namespacing with `dr-` prefix to prevent collisions
➕ File reference support for direct specification

---

## Notes

### Design Principles

1. **Explicit over Implicit**: Users invoke commands explicitly rather than autonomous behavior
2. **Prompt-Driven with Fallback**: Commands accept detailed multi-line prompts OR ask interactively
3. **Deep Thinking**: Commands explicitly use extended thinking for comprehensive analysis
4. **Comprehensive Output**: Produce thoughtful, complete documents - not empty templates
5. **Language-Agnostic**: Works for any programming language or framework
6. **Git-Friendly**: All files are markdown in version control
7. **Progressive Enhancement**: Start simple, add complexity as needed
8. **Idempotent Operations**: Safe to run commands multiple times
9. **Clear Feedback**: Users always know what happened
10. **Accurate Dates**: ALWAYS check system environment for current date/time before inserting into documents

### Technical Considerations

- All commands use relative paths from working directory
- Commands support multi-line prompts via `$ARGUMENTS`
- Commands fall back to interactive mode if no arguments provided
- **Extended Thinking**: Commands explicitly instruct Claude to use deep analysis and extended thinking
- Research leverages Claude's web search capabilities
- **Command Namespacing**: All commands use `dr-` prefix to prevent name collisions with project-specific commands
- **Plan Numbering**: Plans are automatically numbered sequentially (001-999 with leading zeros, 1000+ without) for chronological tracking. Numbers are determined by scanning ALL folders (draft, in_progress, completed) to find the highest existing number, then incrementing by 1
- **Ambiguity Handling**: `/dr-move-plan` asks for clarification when multiple plans match the search term - never guesses which plan the user meant
- **File Reference Support**: Commands support `@path/to/file` syntax for direct file references (especially useful with autocomplete)
- Plans support checkbox tracking for progress
- All output is markdown for maximum compatibility
- **Date/Time Handling**: ALWAYS check system environment for current date/time before inserting into any document - NEVER use hardcoded or guessed dates
- **Git Integration**: `.gitkeep` files added to all leaf directories so empty folders can be committed to version control
- Commands produce comprehensive, thoughtful output - not just empty templates

### Community Contribution

- Plugin is designed to be open-source
- Community can contribute additional templates
- Easy to fork and customize for specific needs
- Marketplace distribution enables wide adoption

---

**Created:** 2025-11-05
**Status:** Planning Complete - Ready for Implementation
**Next Phase:** Phase 2 - Plugin Foundation
