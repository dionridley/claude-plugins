# Changelog

All notable changes to the Project Management Plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.7.0] - 2026-04-18

### Changed

- **`/dr-prd` converted from command to Skill 2.0** (`skills/dr-prd/`) with substantive workflow improvements. Invocation is unchanged (`/dr-prd`); internals are fully rewritten.
  - **Clarifying-question phase in CREATE mode** — hybrid fixed core (problem, users, success metrics, feature type) plus adaptive follow-ups. Compresses automatically when the initial prompt is already rich. Designed to prevent generic, hallucinated, or shallow PRDs.
  - **Non-blocking nudge on thin answers** — when the user can't answer 2+ core questions clearly, the skill surfaces options (run `/dr-research`, brainstorm, share context, or proceed and flag assumptions) without blocking.
  - **Confidence checkpoint before drafting** — conversational, not numeric. Surfaces fuzzy areas and asks whether to clarify or flag.
  - **User-provided research/context only** — the skill does not proactively Glob `_claude/research/` or any other directory. Users reference material explicitly via `@path`.
  - **Problem → Hypothesis → Outcome framing** at the top of every PRD. Aligns with modern PRD practice (Cagan, Lenny).
  - **Adaptive template by feature type** — `user-facing`, `internal-tool`, `infra`, `ai-feature`, and `spike` each include/skip/replace sections appropriately. Feature type is inferred then confirmed with the user; falls back to asking if inference fails.
  - **Timeline & Milestones demoted** to a one-line `Release Strategy` section (delivery planning lives in `/dr-plan`, not the PRD).
  - **Top-level `Acceptance Criteria` section** — testable bullets that `/dr-plan` consumes directly as test-first tasks.
  - **AI-feature overlay** — for `ai-feature` type, adds `Model and Constraints`, `Prompt Spec`, `Eval Rubric` (replaces Acceptance Criteria for probabilistic outputs), `Performance Budgets`, and `Guardrails`. `/dr-plan` reads these to generate eval scaffolding, prompt-test, load-test, and red-team tasks.
  - **Open Questions format** — now `[ ] Question — owner: [name], needs by: [YYYY-MM-DD]` so the section stays actively useful rather than ritual.
  - **REFINE mode safety features preserved** — backup, diff preview, confirm gate (`AskUserQuestion`), status-aware warnings (Draft / Under Review / Approved / Superseded), bidirectional linked-plan detection. These remain the strongest parts of the original command.
  - **Old-template preservation on refine** — existing PRDs authored under the old template are refined without structural changes unless the user explicitly asks to migrate (e.g., "migrate this to the new template"). Migration maps old sections to new and flags inferred content in Open Questions.
  - **Cross-platform via native tools** — all filesystem operations use `Read`/`Write`/`Edit`/`Glob`/`Grep` instead of shell utilities. Works identically on Windows, macOS, and Linux.
  - **Progressive disclosure** — mode-specific logic in `references/create-mode.md` and `references/refine-mode.md`; shared logic in `references/template-variants.md` and `references/ai-feature-sections.md`.

### Added

- **`skills/dr-prd/`** — Full Skill 2.0 implementation with SKILL.md + 4 reference files + new base template
  - `SKILL.md` — mode detection and routing
  - `references/create-mode.md` — clarifying phase + drafting flow
  - `references/refine-mode.md` — validate + backup + diff + confirm + apply flow
  - `references/template-variants.md` — feature-type detection and section-inclusion rules
  - `references/ai-feature-sections.md` — AI/LLM overlay (model, prompt spec, eval rubric, performance budgets, guardrails)
  - `templates/prd-base.md` — new base template

### Removed

- **`commands/dr-prd.md`** — replaced by the new skill
- **`templates/prd-template.md`** (plugin root) — replaced by `skills/dr-prd/templates/prd-base.md` with a restructured section order and new Hypothesis / Acceptance Criteria / Release Strategy sections
- **Bash permissions** no longer required by `/dr-prd`: `Bash(ls:*)`, `Bash(cp:*)`, `Bash(mkdir:*)` — all replaced by native Claude Code tools (`Glob` for listing, `Read`+`Write` for backups, `Write` auto-creates parent directories)

## [1.6.1] - 2026-04-18

### Changed

- **`/dr-research` tool alignment** — `skills/dr-research/SKILL.md` now uses native Claude Code tools instead of shelling out. `Bash(mkdir:*)` removed from `allowed-tools` (Write creates parent directories automatically); `Glob` added for existence checks (deep-dive path verification and `_claude/research/` presence check). Phase 3 step renamed from "Create the directory" to "Determine the output path" to reflect that directory creation is implicit in the first Write.

### Removed

- **`Bash(mkdir:*)` permission** from `/dr-research` — no longer needed

## [1.6.0] - 2026-04-18

### Changed

- **`/dr-init` converted from command to Skill 2.0** (`skills/dr-init/`) with workflow and safety improvements. Invocation is unchanged (`/dr-init`); internals are fully rewritten.
  - **Scope narrowed to plugin-managed content only** — `CLAUDE-template.md` no longer includes `## Project-Specific Commands` or `## Development Principles` sections. Those are software-project concerns handled by Claude Code's built-in `/init` (or by the user directly). State A's success message now suggests running `/init` alongside `/dr-init` for codebase-specific documentation
  - **Diff preview before CLAUDE.md updates** — State B now shows a unified diff of every outdated or missing section before any write, so users can see exactly what will change
  - **Git safety preflight** — before modifying an existing CLAUDE.md (State B or State C), the skill runs `git status --porcelain CLAUDE.md` and warns if there are uncommitted changes. The skill never runs git commands that modify state — commits are entirely the user's responsibility
  - **Simplified State C flow** — appending to an existing CLAUDE.md is now a clean separator-based append rather than a crude concatenation with manual-reorganization guidance. Since plugin sections no longer overlap with typical project content, no merge is needed
  - **Cross-platform via native tools** — all filesystem operations use `Read`/`Write`/`Edit`/`Glob` instead of shell utilities. The skill works identically on Windows, macOS, and Linux without relying on `mkdir`, `touch`, `ls`, `wc`, or `test`
  - **Progressive disclosure** — state-specific logic lives in `references/state-a-fresh.md`, `references/state-b-update.md`, `references/state-c-uninitialized.md`. Section versioning scheme documented in `references/section-versioning.md` for maintainers and future contributors

### Removed

- **`commands/dr-init.md`** — Replaced by the new skill
- **`templates/CLAUDE-template.md`** (plugin root) — Moved to `skills/dr-init/templates/CLAUDE-template.md` since it is specific to the init skill
- **`## Project-Specific Commands` section** from `CLAUDE-template.md` — Out of scope for this plugin; users should run `/init` for codebase-specific documentation
- **`## Development Principles` section** from `CLAUDE-template.md` — Generic software advice not tied to plugin features
- **Bash permissions** no longer required by `/dr-init`: `Bash(mkdir:*)`, `Bash(ls:*)`, `Bash(touch:*)`, `Bash(wc:*)`, `Bash(test:*)` — all replaced by native Claude Code tools. The only Bash permission retained is `Bash(git status:*)` for the safety preflight

### Added

- **`skills/dr-init/`** — Full Skill 2.0 implementation with SKILL.md + 4 reference files + templates/

## [1.5.0] - 2026-04-14

### Changed

- **`/dr-research` converted from command to Skill 2.0** (`skills/dr-research/`) with major workflow and output improvements. Invocation is unchanged (`/dr-research [prompt]` or `/dr-research` for interactive mode); internals are fully rewritten.
  - **Interactive research plan approval** — Before any research runs, Claude presents a structured plan (research type, context, key questions, strategy, sources, planned output files) and waits for user approval or adjustments
  - **Research strategy selection** — Claude now picks an appropriate strategy (funnel, adversarial, temporal, multi-stakeholder) based on the research type, and asks the user when unclear
  - **Deep-dive follow-up support** — Reference an existing research path (e.g., `/dr-research follow-up questions _claude/research/existing-topic-2026-01-15/`) to create a `deep-dives/` subfolder within the original research, with back-links to the parent index and an updated "Deep Dives" section on the parent
  - **Adaptive output structure** — Output files are proposed in the plan and adapted to the research type. `recommendations.md` is only created when the research genuinely supports actionable next steps; topic-specific files (`comparison.md`, `implementation-guide.md`, `architecture.md`, etc.) are created when warranted rather than always
  - **Confidence flags only on uncertain findings** — High-confidence findings are not marked; only findings based on limited sources, single blog posts, or potentially outdated information get a callout. Conflicting sources are documented inline with direct links so the reader can evaluate competing claims
  - **Mermaid diagram support** — Encouraged for workflows, architecture, system interactions, comparisons, and dense technical concepts. Includes a reference file with 7 diagram patterns (flowchart, sequence, mindmap, block, quadrant, ER, gantt)
  - **Circuit breaker pattern** — Claude works to completion without interrupting for normal complexity (contradictions, diverse viewpoints). It stops and asks the user only if the research premise is wrong or a discovery fundamentally changes the direction, capped at 1-2 interruptions maximum
  - **Completion summary improvements** — Now includes a "What surprised me" insight, suggested deep-dive topics (only when genuinely warranted), and contextual follow-up actions tailored to the research type
  - **`index.md` written last** — Ensures the overview accurately reflects everything that was produced; encourages a visual concept map via Mermaid
  - **Extended thinking via `effort: max`** in skill frontmatter instead of inline instructions

### Removed

- **`commands/dr-research.md`** — Replaced by the new skill
- **`templates/research-*-template.md`** — The 4 research template files are replaced by flexible reference guides in `skills/dr-research/references/` (research-methodology, output-formats, mermaid-patterns) plus annotated exemplars in `skills/dr-research/examples/`

## [1.4.1] - 2026-02-22

### Fixed

- **PR validation before update** — strengthened instructions so Claude reliably runs `gh pr view` before any `gh pr edit`, with explicit CRITICAL/REQUIRED markers to prevent skipping the validation step

## [1.4.0] - 2026-02-22

### Added

- **GitHub PR auto-update** in `/dr-plan summary` mode
  - Optionally pass a GitHub PR URL: `/dr-plan @plan.md summary https://github.com/org/repo/pull/123`
  - Automatically updates the PR title (set to the commit message title) and description (set to the generated PR summary) via `gh pr edit`
  - Validates PR is open before updating — blocks update on merged/closed PRs with clear error message
  - Prompts for confirmation if the PR already has an existing description before overwriting
  - Falls back to display-only mode if the `gh` command fails
  - When no PR URL is provided, existing copy-paste behavior is unchanged

### Changed

- **Allowed-tools updated** in `/dr-plan` — added `Bash(gh pr edit:*)` and `Bash(gh pr view:*)` scoped to PR operations only
- **`summary.md` restructured** with conditional output paths (Path A: update PR directly, Path B: display for manual copy)

## [1.3.1] - 2026-02-09

### Changed

- **Date handling standardized** across all commands (`/dr-init`, `/dr-research`, `/dr-prd`, `/dr-plan`) to use conversation context instead of ambiguous "system environment" for cross-platform reliability
- **User confirmation prompts** in `/dr-init`, `/dr-prd`, and `/dr-plan` now use structured AskUserQuestion with labeled options instead of text-based `[y/n/diff]` prompts
- **Directory existence checks** added to `/dr-research`, `/dr-prd`, and `/dr-plan` CREATE modes — commands now suggest running `/dr-init` if `_claude/` directories don't exist
- **Allowed-tools trimmed** in `/dr-init` — removed unused `Bash(find:*)`, `Bash(cat:*)`, `Bash(sed:*)`, `Bash(echo:*)`
- **Allowed-tools updated** in `/dr-prd` — replaced `Bash(grep:*)` with `Bash(mkdir:*)` (search uses Grep tool instead)
- **Allowed-tools updated** in `/dr-plan` — added `Bash(mkdir:*)`

### Fixed

- **CLAUDE-template.md** — removed stale `/dr-move-plan` references (command was removed in v1.2.0), bumped plan-management-workflow section version to v2
- **dr-plan.md** — fixed duplicate step 4 numbering in Phase 4, fixed Important Notes numbering gap (7,9,10 → 7,8,9)
- **Template cleanup** — removed redundant `{{PLACEHOLDER}}` variables from `plan-template.md`, `prd-template.md`, and `research-index-template.md` that duplicated human-readable instructions

## [1.3.0] - 2026-02-08

### Added

- **Template section versioning** for `CLAUDE-template.md`
  - Sections now have version markers (`<!-- section: name v1 -->`) to track content changes
  - `/dr-init` detects outdated or missing sections in existing projects and offers to update them
  - Two-tier verification: Tier 1 checks section existence, Tier 2 checks content is current

- **CLAUDE.md verification in `/dr-init`** (STATE B)
  - Re-running `/dr-init` on an already-initialized project now verifies CLAUDE.md sections
  - Shows categorized results: current, outdated, or missing
  - Offers Update (apply changes), Show (display for manual copy), or Skip options
  - Replaces the previous "never modify CLAUDE.md" behavior

### Changed

- **Task Completion Protocol** - Claude now proactively checks plan checkboxes after completing each phase instead of waiting for user confirmation
  - Work through one phase at a time
  - Update the plan file immediately after completing a phase
  - Report what was completed and what phase is next

### Removed

- **Plan backup files** - Removed automatic `.backup` file creation during plan refinement and question resolution
  - Backup creation via `Bash(cp:*)` removed from `/dr-plan` REFINE mode
  - Backup creation removed from `/dr-plan` QUESTION RESOLUTION mode
  - Backup references removed from success/cancellation messages
  - `Bash(cp:*)` removed from `/dr-plan` allowed-tools
  - Troubleshooting section about backup files removed from README

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
