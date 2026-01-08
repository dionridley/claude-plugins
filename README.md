# Claude Code Plugin Marketplace

A collection of plugins that extend Claude Code's capabilities with specialized workflows and tools.

## Available Plugins

| Plugin | Description |
|--------|-------------|
| [Project Management](./project-mgmt-plugin/README.md) | Structured project management with research, PRDs, and implementation plans |
| [Engineering Tools](./engineering-tools/README.md) | Skills and commands to assist with software engineering and coding tasks |

## Installation

### Option 1: Via Claude Code UI (Recommended)

1. Open Claude Code and type `/plugins` to open the plugin manager
2. Select **Add plugin...**
3. Enter the marketplace URL: `dionridley/claude-plugins`
4. Select which plugins to enable
5. Enable **Auto-update** when prompted to receive new features automatically

### Option 2: Via Command Line

#### Installing the Marketplace

Install the entire marketplace to get access to all plugins:

```bash
claude plugins add dionridley/claude-plugins
```

After installation, enable auto-update to receive new features and bug fixes automatically:

```bash
claude plugins update dionridley/claude-plugins --auto-update
```

With auto-update enabled, plugins will be kept up to date each time Claude Code starts.

#### Installing Individual Plugins

If you only want specific plugins, you can install them individually by specifying the plugin name:

```bash
# Install only the project management plugin
claude plugins add dionridley/claude-plugins --plugin project-management

# Install only the engineering tools plugin
claude plugins add dionridley/claude-plugins --plugin engineering-tools
```

## About Marketplaces

A marketplace is a curated collection of plugins maintained in a single repository. When you add a marketplace:

- **All plugins are available**: Every plugin in the marketplace becomes available to Claude Code
- **Single source**: Updates to any plugin come through the same repository
- **Version coordination**: Plugin versions are managed together in the marketplace catalog

### Marketplace vs Individual Plugins

| Approach | Pros | Cons |
|----------|------|------|
| **Full Marketplace** | Get all plugins at once; simpler updates | May include plugins you don't need |
| **Individual Plugin** | Only install what you use | Must add each plugin separately |

For most users, installing the full marketplace with auto-update enabled is the simplest approach.

## Verifying Installation

After installation, verify the plugins are available:

```bash
# List installed plugins
claude plugins list

# Check available commands
/help
```

For the project management plugin, you should see commands like `/dr-init`, `/dr-research`, `/dr-prd`, and `/dr-plan`.

## Updating Plugins

If auto-update is not enabled, manually update plugins:

```bash
claude plugins update dionridley/claude-plugins
```

## Removing Plugins

To remove the marketplace and all its plugins:

```bash
claude plugins remove dionridley/claude-plugins
```

## Contributing

Contributions are welcome! See [CLAUDE.md](./CLAUDE.md) for development guidelines.

## License

MIT
