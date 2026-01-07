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

*No skills yet - add skill directories to `skills/` folder*

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
