# Changelog

All notable changes to the Project Management Plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2026-01-07

### Removed

- **`/dr-move-plan` command** - Removed to reduce token usage. Plans can now be moved using standard `mv` commands or Claude will move files automatically when contextually appropriate
- **`/dr-summary` reference** - Removed stale command reference from plugin.json (file never existed)

### Changed

- **Plan movement guidance** - Other commands now suggest using `mv` command directly or trusting Claude to move files automatically

## [1.1.0] - 2026-01-06

### Removed

- **frontend-design skill** - Moved to engineering-tools plugin for better organization

## [1.0.5] - 2025-12-23

### Added

- **Branch commit message generation** in `/dr-plan summary` mode
  - Generates a copyable commit message alongside the PR summary
  - Includes a short imperative title (3-6 words) and up to 5 bullet points
  - Format uses `*` prefix for bullets, past tense for completed actions
  - Displayed in separate code fence for easy copying
  - Useful for squash/merge commits when closing PRs

## [1.0.4] - 2025-12-22

### Changed

- **Replaced bash find/grep with Glob tool** in `/dr-plan` and `/dr-move-plan` commands
  - Plan number scanning now uses native `Glob` tool instead of `find | grep`
  - Plan search operations use `Glob` with filtering in Claude's reasoning
  - Eliminates user approval prompts for file search operations
  - Faster execution by using Claude's native file matching capabilities

### Fixed

- **Reduced permission prompts** - Users no longer need to approve bash commands for routine plan searches
  - `/dr-plan` CREATE mode: No longer prompts for `find` command when determining next plan number
  - `/dr-move-plan`: No longer prompts for `find` or `ls` commands when searching for plans

## [1.0.3] - 2025-12-21

### Added

- **SUMMARY mode** for `/dr-plan` - Generate GitHub-ready PR descriptions from plans
  - Creative formatting with tables, emojis, mermaid diagrams, and collapsible sections
  - Copyable markdown output wrapped in code fence (using tildes for nested code blocks)
  - Usage: `/dr-plan @plan-file.md summary`

- **QUESTION RESOLUTION mode** for `/dr-plan` - Interactive Q&A for plan refinement
  - Guides users through resolving uncertain assumptions and open questions
  - Prioritizes blocking questions marked `[AWAITING]`
  - Updates plan file with decisions marked `[DECIDED: date]`
  - Creates backup before making changes
  - Usage: `/dr-plan @plan-file.md answer questions`

### Changed

- **Modular command architecture** - Mode-specific logic extracted to `commands/dr-plan/` subfolder
  - `commands/dr-plan/summary.md` - SUMMARY mode instructions
  - `commands/dr-plan/questions.md` - QUESTION RESOLUTION mode instructions
  - Main `dr-plan.md` handles mode detection and delegates to subfolder files

## [1.0.0] - 2025-11-24

### Added

#### Core Commands
- `/dr-init` - Initialize project with standard directory structure and CLAUDE.md
  - Smart state detection (fresh, initialized, uninitialized)
  - Creates `_claude/` directory with docs, plans, prd, resources, research subdirectories
  - Creates `.gitkeep` files for git tracking of empty directories
  - Generates CLAUDE.md with project management guidelines
  - Handles existing CLAUDE.md files with append/show/cancel options
  - Idempotent - safe to run multiple times

- `/dr-research` - Conduct deep research with extended thinking
  - Accepts detailed multi-line research prompts
  - Interactive mode when no arguments provided
  - Creates timestamped research directory
  - Generates interconnected markdown files (index, findings, resources, recommendations)
  - Uses web search and extended thinking for comprehensive analysis

- `/dr-prd` - Create or refine Product Requirements Documents
  - **CREATE mode**: Generates comprehensive PRD from detailed feature description
  - **REFINE mode**: Intelligently updates existing PRD with `@file` reference
  - Extended thinking for requirements analysis
  - All sections thoughtfully populated (not just placeholders)
  - Automatic backup before refinement
  - Diff summary and confirmation before applying changes
  - Status-aware (Draft, Under Review, Approved, Superseded)
  - Detects and warns about linked plans

- `/dr-plan` - Create or refine implementation plans
  - **CREATE mode**: Generates detailed phase-based implementation plan
  - **REFINE mode**: Updates existing plan with extended thinking analysis
  - **QUESTION RESOLUTION mode**: Interactive resolution of blocking questions and assumptions
  - Sequential plan numbering (001, 002, etc.) across all folders
  - PRD linking via `@path/to/prd.md` syntax
  - `--in-progress` flag to create directly in in_progress folder
  - Automatic backup before refinement
  - Status-aware behavior (draft allows all changes, in-progress warns on major changes, completed refuses)
  - Assumptions and Open Questions sections for collaborative decision-making

- `/dr-move-plan` - Move plans between lifecycle stages
  - Three input methods: plan number, plan name (partial match), file reference
  - Searches all folders (draft, in_progress, completed)
  - Handles ambiguous matches with user clarification
  - Preserves plan number when moving
  - Clear confirmation with source and destination

#### Templates
- `CLAUDE-template.md` - Project guidelines template with plan workflow rules
- `plan-template.md` - Implementation plan template with phases, tasks, and tracking
- `prd-template.md` - PRD template with all critical sections
- `research-index-template.md` - Research documentation index template
- `research-findings-template.md` - Research findings template
- `research-resources-template.md` - Research resources and links template
- `research-recommendations-template.md` - Actionable recommendations template

#### Features
- **Extended Thinking Integration** - All commands use deep analysis for comprehensive output
- **Multi-line Prompt Support** - Commands accept detailed 10-15 line prompts for best results
- **PRD-Plan Linking** - Plans can reference PRDs via `@path` syntax
- **Automatic Backups** - Refinement creates `.backup` files before changes
- **Diff Summaries** - See what changed before confirming refinements
- **Confirmation Prompts** - Safety confirmations with `--no-confirm` override option
- **Sequential Numbering** - Plans auto-numbered across all folders
- **Collaborative Decision-Making** - Assumptions and Open Questions tracking in plans
- **Language Agnostic** - Works with any programming language or framework
- **Git-Friendly** - All output is markdown with `.gitkeep` for empty directories

#### Documentation
- Comprehensive README with installation, usage, and examples
- Troubleshooting guide for common issues
- Workflow guidelines and best practices
- Detailed command documentation with examples

### Technical Details
- Plugin manifest (`plugin.json`) with metadata
- Command files use frontmatter for description and allowed-tools
- All paths use forward slashes for cross-platform compatibility
- Date handling uses system environment (never hardcoded)

---

## [Unreleased]

### Planned for Future Releases

#### v1.1 Considerations
- `/dr-archive-plan` - Archive old completed plans
- Status badges in plan files
- Plan template variations (small change, large feature, migration)

#### v1.2 Considerations
- Hooks for automatic plan status updates
- MCP integration for external project management tools
- Custom template support per project
- Plan metrics and analytics

#### v2.0 Considerations
- Skills for autonomous plan creation
- Agent for code review against plans
- Interactive plan refinement
- Plan dependency visualization
