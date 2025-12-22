# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code plugin marketplace repository containing reusable plugins that extend Claude Code's capabilities. The primary plugin is `project-mgmt-plugin` which provides structured project management workflows.

## Repository Structure

```
claude-plugins/
├── .claude-plugin/
│   └── marketplace.json       # Marketplace catalog configuration
├── .claude/
│   └── skills/
│       └── skill-creator/     # Skill for creating new skills
├── project-mgmt-plugin/       # Main project management plugin
│   ├── .claude-plugin/
│   │   └── plugin.json        # Plugin manifest
│   ├── commands/              # Slash commands (.md files)
│   │   ├── dr-init.md
│   │   ├── dr-research.md
│   │   ├── dr-prd.md
│   │   ├── dr-plan.md         # Multi-mode: CREATE, REFINE, SUMMARY, QUESTION RESOLUTION
│   │   ├── dr-plan/           # Subfolder for mode-specific logic
│   │   │   ├── summary.md
│   │   │   └── questions.md
│   │   └── dr-move-plan.md
│   ├── skills/
│   │   └── frontend-design/   # UI/frontend design skill
│   └── templates/             # Markdown templates for generated files
```

## Plugin Architecture

### Command Structure

Commands are markdown files with YAML frontmatter:

```markdown
---
description: Short description for /help
argument-hint: [arg1] [arg2] [--flag]
allowed-tools: Read, Write, Edit, Bash(specific:*)
---

# Command Title

Instructions for Claude to execute...
```

Key patterns:
- `<command-args>` tag contains user arguments at runtime
- `${CLAUDE_PLUGIN_ROOT}` resolves to the plugin's root directory
- `@file.md` references auto-expand file content into context (path disappears from args)
- Workaround: `@file.md keyword` keeps "keyword" in args while expanding file

### Multi-Mode Commands

For commands with multiple modes (like dr-plan), use subfolder delegation:

```
commands/
├── dr-plan.md           # Mode detection + routing
└── dr-plan/
    ├── summary.md       # SUMMARY mode logic
    └── questions.md     # QUESTION RESOLUTION mode logic
```

The main command detects mode from args and reads the appropriate subfolder file.

### Skills vs Commands

| Aspect | Slash Command | Skill |
|--------|---------------|-------|
| Trigger | Manual `/command` | Auto-discovered by Claude |
| Structure | Single .md file | Directory with SKILL.md |
| Arguments | Supports `<command-args>` | Context-driven |
| Use when | User wants explicit control | Claude should autonomously use |

### Skill Structure

```
skill-name/
├── SKILL.md              # Required: frontmatter + instructions
├── scripts/              # Optional: executable code
├── references/           # Optional: documentation loaded as needed
└── assets/               # Optional: templates, images for output
```

SKILL.md frontmatter must have `name` and `description` fields. The description is the primary trigger mechanism.

## Development Workflow

### Testing Commands

Commands are tested by using them in a project with the plugin installed. There's no automated test suite - validation is manual.

### Markdown Output for Copying

When generating content users need to copy (like PR summaries), use tildes for the outer fence to allow backticks inside:

````markdown
~~~markdown
## Summary
Content with `code` and:
```bash
commands
```
~~~
````

### Template Variables

Templates in `templates/` use placeholder patterns that commands fill in:
- `[YYYY-MM-DD]` - Current date
- `[Plan Name]` - Generated from context
- `${CLAUDE_PLUGIN_ROOT}` - Plugin root path (runtime)

## Plugin Manifest (plugin.json)

Required fields:
- `name`: kebab-case identifier
- `version`: semver string
- `description`: What the plugin does
- `commands`: Array of relative paths to command files
- `skills`: Array of relative paths to skill directories

## Version Management

When releasing a new version for any plugin:

1. **Check both version files first:**
   - `.claude-plugin/marketplace.json` (parent marketplace catalog)
   - `<plugin-folder>/.claude-plugin/plugin.json` (plugin manifest)

2. **Determine the new version:**
   - Find the higher of the two current versions
   - Increment appropriately (patch/minor/major based on changes)

3. **Update all three locations to the same version:**
   - Update `version` field in plugin.json
   - Update `version` field in the plugin's entry in marketplace.json
   - Add new entry to CHANGELOG.md

4. **CHANGELOG.md format** (follows [Keep a Changelog](https://keepachangelog.com/)):
   - Add new version section at top (below Unreleased if present)
   - Format: `## [X.Y.Z] - YYYY-MM-DD`
   - Categorize changes: Added, Changed, Deprecated, Removed, Fixed, Security
   - Use bullet points with brief descriptions

This ensures version consistency across the marketplace catalog, plugin manifest, and changelog.

## Key Files

### Repository-Level
- `.claude-plugin/marketplace.json` - Marketplace catalog listing all plugins with versions
- `.claude/skills/skill-creator/SKILL.md` - Guide for creating new skills

### Per-Plugin Files (each plugin folder contains these)
Each plugin is a self-contained folder (e.g., `project-mgmt-plugin/`) with its own:
- `.claude-plugin/plugin.json` - Plugin manifest with version, commands, skills
- `README.md` - Documentation for that specific plugin
- `CHANGELOG.md` - Version history for that specific plugin only

**Important**: When making changes to a plugin, update that plugin's own CHANGELOG.md - not a different plugin's changelog. Each plugin maintains independent version history.

### Current Plugins
- `project-mgmt-plugin/` - Project management with research, PRDs, and implementation plans
