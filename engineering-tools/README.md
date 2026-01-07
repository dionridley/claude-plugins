# Engineering Tools Plugin

Skills and commands to assist with software engineering and coding tasks.

## Installation

Add this plugin to your Claude Code configuration:

```bash
claude mcp add-plugin https://github.com/dionridley/claude-plugins engineering-tools
```

## Features

### Commands

*No commands yet - add `.md` files to `commands/` folder*

### Skills

#### frontend-design

Creates distinctive, production-grade frontend interfaces with high design quality. This skill is automatically triggered when you ask Claude to build:

- Web components or pages
- Landing pages or dashboards
- React/Vue/HTML components
- UI styling or beautification

**Key features:**
- Bold aesthetic direction (minimalist, maximalist, retro-futuristic, etc.)
- Production-grade, functional code
- Distinctive typography and color choices
- Motion and micro-interactions
- Avoids generic "AI slop" aesthetics

**Example prompts:**
- "Build me a landing page for a coffee subscription service"
- "Create a dashboard for monitoring server health"
- "Design a React component for a pricing table"

## Development

### Adding a Command

1. Create a new `.md` file in `commands/`
2. Add YAML frontmatter with `description` and `allowed-tools`
3. Register in `plugin.json` under `commands` array

### Adding a Skill

1. Create a new directory in `skills/`
2. Add `SKILL.md` with frontmatter (`name`, `description`)
3. Register in `plugin.json` under `skills` array

## License

MIT
