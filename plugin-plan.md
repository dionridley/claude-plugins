# Project Management Plugin - Implementation Plan

## Executive Summary

This plan outlines the creation of a reusable Claude Code plugin that brings structured project management practices to any codebase. The plugin provides standardized directory structures, research workflows, PRD creation, and implementation planning with consistent templates - all language and framework agnostic.

**Key Goals:**
- Create reusable project structure (`_claude/` with docs, plans, prd, resources, research)
- Provide explicit workflows for research, PRD writing, and plan creation
- Use templates to ensure consistency across projects
- Enable easy initialization and updates via `/init-project-structure` command

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

**Decision:** Create `/init-project-structure` slash command that:
1. Creates directory structure if missing
2. Generates or updates CLAUDE.md with standardized instructions
3. Preserves existing content while adding/updating plugin sections
4. Can be re-run as plugin evolves to gain new features

### Decision 2: Skills vs Slash Commands for Core Features

**Research Workflow:**
- **Decision:** Slash command (`/research`)
- **Rationale:** Users explicitly decide when to conduct research, provide research topic as argument, want control over when research happens
- **Usage:** `/research [topic]` creates `_claude/research/[topic-slug]/` with structured markdown files

**PRD Creation:**
- **Decision:** Slash command (`/prd`)
- **Rationale:** Explicit document creation, user provides feature name, manual trigger point
- **Usage:** `/prd [feature-name]` creates `_claude/prd/[feature-name].md` from template

**Plan Creation:**
- **Decision:** Slash command (`/plan`)
- **Rationale:** Explicit document creation, uses consistent template, user decides when planning happens
- **Usage:** `/plan [plan-name]` creates `_claude/plans/draft/[plan-name].md` with full template structure

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
- `/plan` creates in `draft/` by default
- `/plan --in-progress` creates in `in_progress/` directly
- CLAUDE.md includes strict rules about not executing draft plans

---

## Plugin Structure

### Directory Layout

```
project-mgmt-plugin/
├── .claude-plugin/
│   └── plugin.json                          # Plugin manifest
├── commands/
│   ├── init-project-structure.md            # Initialize/update project structure
│   ├── research.md                          # Conduct structured research
│   ├── prd.md                               # Create PRD from template
│   ├── plan.md                              # Create implementation plan
│   └── move-plan.md                         # Move plan between stages
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
    "./commands/init-project-structure.md",
    "./commands/research.md",
    "./commands/prd.md",
    "./commands/plan.md",
    "./commands/move-plan.md"
  ]
}
```

---

## Component Specifications

### 1. Slash Command: `/init-project-structure`

**Purpose:** Initialize or update project with standard directory structure and CLAUDE.md

**Location:** `commands/init-project-structure.md`

**Frontmatter:**
```yaml
---
description: Initialize or update project with standard directory structure
allowed-tools: Bash(mkdir:*), Bash(ls:*), Read, Write, Edit
---
```

**Behavior:**
1. Check if `_claude/` directory exists
2. Create directory structure:
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
3. Check if CLAUDE.md exists
4. If missing: Create from template
5. If exists: Check for plugin-managed section and update it
6. Create .gitkeep files in empty directories

**CLAUDE.md Template Content:**
- Welcome message
- Directory structure explanation
- Commands available from plugin
- Plan workflow (draft → in_progress → completed)
- Strict rules about not executing draft plans
- Section for project-specific commands (placeholder)
- Development principles (general, not language-specific)

**User Feedback:**
```
✅ Project structure initialized!

Created:
  _claude/docs/
  _claude/plans/draft/
  _claude/plans/in_progress/
  _claude/plans/completed/
  _claude/prd/
  _claude/resources/
  _claude/research/

Updated: CLAUDE.md
  - Added project management structure instructions
  - Available commands: /research, /prd, /plan, /move-plan

Next steps:
  1. Review CLAUDE.md and customize for your project
  2. Add project-specific commands (build, test, lint, etc.)
  3. Create your first PRD: /prd [feature-name]
  4. Or conduct research: /research [topic]
```

### 2. Slash Command: `/research`

**Purpose:** Conduct structured research and document findings in organized markdown files

**Location:** `commands/research.md`

**Frontmatter:**
```yaml
---
description: Conduct research on a topic and document findings
argument-hint: [topic]
allowed-tools: WebSearch, WebFetch, Read, Write, Bash(mkdir:*)
---
```

**Usage:** `/research [topic]`

**Behavior:**
1. **CRITICAL: Check current date/time** using system environment before any document creation
2. Take topic from `$ARGUMENTS`
3. Create slug from topic (e.g., "OAuth Implementation" → "oauth-implementation")
4. Create timestamp using actual current date (YYYY-MM-DD format)
5. Create directory: `_claude/research/[slug]-[timestamp]/`
6. Conduct research using WebSearch, WebFetch as needed
7. Structure findings into multiple markdown files:
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

**User Feedback:**
```
✅ Research completed: [topic]

Created: _claude/research/[slug]-[timestamp]/
  - index.md (overview and navigation)
  - findings.md (main research)
  - resources.md (links and references)
  - recommendations.md (next steps)

Summary: [2-3 sentence summary of findings]

Next steps:
  - Review findings in _claude/research/[slug]-[timestamp]/
  - Create PRD if ready: /prd [feature-name]
  - Create implementation plan: /plan [plan-name]
```

### 3. Slash Command: `/prd`

**Purpose:** Create Product Requirement Document from template

**Location:** `commands/prd.md`

**Frontmatter:**
```yaml
---
description: Create a Product Requirement Document for a feature
argument-hint: [feature-name]
allowed-tools: Read, Write
---
```

**Usage:** `/prd [feature-name]`

**Behavior:**
1. **CRITICAL: Check current date** using system environment before document creation
2. Take feature name from `$ARGUMENTS`
3. Create slug from name
4. Read PRD template from plugin
5. Create file: `_claude/prd/[feature-slug].md`
6. Populate template with feature name and **current date** in date fields
7. Guide user through filling in sections

**PRD Template Structure:**
```markdown
# PRD: [Feature Name]

**Status:** Draft
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
```

**User Feedback:**
```
✅ PRD created: _claude/prd/[feature-slug].md

Template includes:
  - Problem statement and goals
  - User stories with acceptance criteria
  - Functional and non-functional requirements
  - Technical considerations
  - Timeline and risks

Next steps:
  1. Fill in PRD sections with details
  2. Review with stakeholders
  3. Create implementation plan: /plan [plan-name]
```

### 4. Slash Command: `/plan`

**Purpose:** Create implementation plan from standardized template

**Location:** `commands/plan.md`

**Frontmatter:**
```yaml
---
description: Create an implementation plan for a feature or change
argument-hint: [plan-name] [--in-progress]
allowed-tools: Read, Write
---
```

**Usage:**
- `/plan [plan-name]` - Creates in draft/
- `/plan [plan-name] --in-progress` - Creates in in_progress/

**Behavior:**
1. **CRITICAL: Check current date** using system environment before document creation
2. Parse arguments for plan name and optional flag
3. Create slug from plan name
4. Determine destination folder (draft vs in_progress)
5. Read plan template from plugin
6. Create file: `_claude/plans/[folder]/[plan-slug].md`
7. Populate with template structure, placeholders, and **current date** in date fields

**Plan Template Structure:**
(Based on analyzed format from par-v2-migration)

```markdown
# [Plan Name]

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

## Implementation Notes

**Actual Time Tracking:**
- Phase 1: [Estimated: X hours] (Actual: Y hours)
- Phase 2: [Estimated: X hours] (Actual: Y hours)

**Key Decisions:**
- [Document important decisions made during implementation]

**Lessons Learned:**
- [Document what worked well and what didn't]
```

**User Feedback:**
```
✅ Plan created: _claude/plans/[folder]/[plan-slug].md

Location: _claude/plans/draft/[plan-slug].md
Status: Draft (not ready for implementation)

Template includes:
  - Executive summary and current state
  - Success criteria with checkboxes
  - Phase-based implementation structure
  - Test verification sections
  - Rollback plan and dependencies
  - Success metrics tracking

Next steps:
  1. Fill in plan details for each phase
  2. Review and refine the plan
  3. When ready: /move-plan [plan-name] in-progress
  4. IMPORTANT: Only implement plans in 'in_progress' folder!
```

### 5. Slash Command: `/move-plan`

**Purpose:** Move plan between draft, in_progress, and completed folders

**Location:** `commands/move-plan.md`

**Frontmatter:**
```yaml
---
description: Move a plan between draft, in_progress, and completed
argument-hint: [plan-name] [draft|in-progress|completed]
allowed-tools: Bash(mv:*), Bash(ls:*), Read
---
```

**Usage:** `/move-plan [plan-name] [destination]`

**Examples:**
- `/move-plan authentication in-progress` - Move to active work
- `/move-plan authentication completed` - Mark as done
- `/move-plan authentication draft` - Move back to draft

**Behavior:**
1. Search for plan file in all three folders
2. Confirm source and destination
3. Move file to new location
4. Update any internal status markers if present
5. Confirm move to user

**User Feedback:**
```
✅ Plan moved: authentication

From: _claude/plans/draft/authentication.md
To:   _claude/plans/in_progress/authentication.md

Status: Ready for implementation

You can now begin working on this plan.
```

---

## Templates

### Template: CLAUDE.md.template

```markdown
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

This project follows a structured approach to planning, documentation, and implementation.

```
_claude/
├── docs/           # Technical documentation and architecture
├── plans/          # Implementation plans
│   ├── draft/      # Plans being developed or refined
│   ├── in_progress/ # Plans currently being implemented
│   └── completed/  # Finished and archived plans
├── prd/            # Product Requirement Documents
├── resources/      # Reference materials and external docs
└── research/       # Structured research output
```

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

1. **Create Plan**: `/plan [plan-name]` creates in `draft/`
2. **Review and Refine**: Edit plan with all necessary details
3. **Move to Active**: `/move-plan [plan-name] in-progress` when ready to implement
4. **Implement**: Work through plan phases systematically
5. **Complete**: `/move-plan [plan-name] completed` when finished

## Available Commands

This project uses the **project-management** plugin which provides:

- `/init-project-structure` - Initialize or update project structure
- `/research [topic]` - Conduct structured research on a topic
- `/prd [feature-name]` - Create a Product Requirement Document
- `/plan [plan-name]` - Create an implementation plan
- `/move-plan [plan-name] [stage]` - Move plan between stages

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

### Phase 2: Plugin Foundation

**Estimated Time:** 2 hours

#### Tasks
- [ ] Create plugin directory structure
- [ ] Write plugin.json manifest
- [ ] Create README.md with plugin documentation
- [ ] Set up templates/ directory
- [ ] Create .gitignore for plugin

#### Deliverables
- [ ] project-mgmt-plugin/ with proper structure
- [ ] plugin.json with metadata
- [ ] README.md with installation and usage instructions

#### Success Criteria
- [ ] Plugin directory follows Claude Code standards
- [ ] Manifest is valid JSON with required fields
- [ ] Documentation is clear and comprehensive

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

### Phase 6: Slash Command - /init-project-structure

**Estimated Time:** 3 hours

#### Tasks
- [ ] Create commands/init-project-structure.md
- [ ] Add frontmatter with description and allowed-tools
- [ ] Write command logic:
  - [ ] Check for existing _claude/ directory
  - [ ] Create directory structure if missing
  - [ ] Read CLAUDE.md.template
  - [ ] Create or update CLAUDE.md
  - [ ] Create .gitkeep files
  - [ ] Provide user feedback
- [ ] Test on new project
- [ ] Test on existing project with CLAUDE.md

#### Deliverables
- [ ] commands/init-project-structure.md

#### Success Criteria
- [ ] Creates directory structure correctly
- [ ] Generates CLAUDE.md from template
- [ ] Idempotent (safe to run multiple times)
- [ ] Preserves existing CLAUDE.md content
- [ ] Clear user feedback

### Phase 7: Slash Command - /research

**Estimated Time:** 3 hours

#### Tasks
- [ ] Create commands/research.md
- [ ] Add frontmatter with description and argument handling
- [ ] Write command logic:
  - [ ] **FIRST: Check system date/time** (critical requirement)
  - [ ] Parse topic argument
  - [ ] Create slug and timestamp using ACTUAL current date
  - [ ] Create research directory
  - [ ] Conduct research using WebSearch/WebFetch
  - [ ] Structure findings into multiple files
  - [ ] Create cross-references between files
  - [ ] Generate index.md with navigation and current date
- [ ] Test with sample research topic

#### Deliverables
- [ ] commands/research.md

#### Success Criteria
- [ ] Creates proper directory structure
- [ ] Generates multiple linked markdown files
- [ ] Research is thorough and well-organized
- [ ] Index provides clear navigation
- [ ] Works with various topic types

### Phase 8: Slash Command - /prd

**Estimated Time:** 2 hours

#### Tasks
- [ ] Create commands/prd.md
- [ ] Add frontmatter with description and argument handling
- [ ] Write command logic:
  - [ ] **FIRST: Check system date/time** (critical requirement)
  - [ ] Parse feature name argument
  - [ ] Create slug
  - [ ] Read PRD template
  - [ ] Create PRD file with populated template using ACTUAL current date
  - [ ] Guide user through sections
- [ ] Test with sample feature

#### Deliverables
- [ ] commands/prd.md

#### Success Criteria
- [ ] Creates PRD with proper template
- [ ] Filename follows naming convention
- [ ] Template sections are clear
- [ ] User guidance is helpful

### Phase 9: Slash Command - /plan

**Estimated Time:** 2.5 hours

#### Tasks
- [ ] Create commands/plan.md
- [ ] Add frontmatter with description and argument handling
- [ ] Write command logic:
  - [ ] **FIRST: Check system date/time** (critical requirement)
  - [ ] Parse plan name and optional flag
  - [ ] Determine destination folder
  - [ ] Read plan template
  - [ ] Create plan file with populated template using ACTUAL current date
  - [ ] Provide appropriate user feedback based on destination
- [ ] Test creating draft plan
- [ ] Test creating in-progress plan

#### Deliverables
- [ ] commands/plan.md

#### Success Criteria
- [ ] Creates plans in correct folder
- [ ] Template is fully populated
- [ ] Respects --in-progress flag
- [ ] Clear warnings about draft vs in-progress

### Phase 10: Slash Command - /move-plan

**Estimated Time:** 2 hours

#### Tasks
- [ ] Create commands/move-plan.md
- [ ] Add frontmatter with description and argument handling
- [ ] Write command logic:
  - [ ] Search for plan in all folders
  - [ ] Validate destination
  - [ ] Move file
  - [ ] Confirm to user
  - [ ] Handle errors gracefully
- [ ] Test moving between all folder combinations

#### Deliverables
- [ ] commands/move-plan.md

#### Success Criteria
- [ ] Finds plans regardless of current location
- [ ] Moves files correctly
- [ ] Handles edge cases (plan not found, already in destination)
- [ ] Clear confirmation messages

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
- [ ] All 5 slash commands implemented and working
- [ ] All 5 templates created and tested
- [ ] Plugin installs without errors
- [ ] Documentation is comprehensive

### Functionality
- [ ] `/init-project-structure` creates directory structure and CLAUDE.md
- [ ] `/research` conducts research and creates multi-file documentation
- [ ] `/prd` creates PRD from template
- [ ] `/plan` creates implementation plan in correct folder
- [ ] `/move-plan` moves plans between stages

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
2. `/init-project-structure` preserves existing content
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
- [ ] `/archive-plan` - Archive old completed plans
- [ ] `/link-plan-prd` - Create references between plans and PRDs
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

### What to Generalize
🔄 Remove Elixir/Phoenix/Ash specific content
🔄 Remove specific framework constraints (Argon2, binary IDs, etc.)
🔄 Make commands work for any language
🔄 Remove agent definitions (backend-developer, graphql-architect)
🔄 Generalize error handling patterns

### What to Add
➕ PRD workflow (missing from original)
➕ Explicit research command
➕ Template management system
➕ Move-plan command for workflow management
➕ Better init/update mechanism

---

## Notes

### Design Principles

1. **Explicit over Implicit**: Users invoke commands explicitly rather than autonomous behavior
2. **Template-Driven**: Consistency through templates, not repeated prompting
3. **Language-Agnostic**: Works for any programming language or framework
4. **Git-Friendly**: All files are markdown in version control
5. **Progressive Enhancement**: Start simple, add complexity as needed
6. **Idempotent Operations**: Safe to run commands multiple times
7. **Clear Feedback**: Users always know what happened
8. **Accurate Dates**: ALWAYS check system environment for current date/time before inserting into documents

### Technical Considerations

- All commands use relative paths from working directory
- Templates use simple variable substitution (no complex templating engine)
- Research leverages Claude's web search capabilities
- Plans support checkbox tracking for progress
- All output is markdown for maximum compatibility
- **Date/Time Handling**: ALWAYS check system environment for current date/time before inserting into any document - NEVER use hardcoded or guessed dates

### Community Contribution

- Plugin is designed to be open-source
- Community can contribute additional templates
- Easy to fork and customize for specific needs
- Marketplace distribution enables wide adoption

---

**Created:** 2025-11-05
**Status:** Planning Complete - Ready for Implementation
**Next Phase:** Phase 2 - Plugin Foundation
