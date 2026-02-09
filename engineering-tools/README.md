# Engineering Tools Plugin

Skills and commands to assist with software engineering and coding tasks.

## Installation

See the [marketplace README](../README.md#installation) for installation instructions.

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

#### react-19

Provides guidance on React 19.x features and APIs that may not be in LLM training data. Automatically triggered when working with:

- React 19+ features and new hooks
- Form handling with Actions
- Server Components and Server Actions
- New React 19 patterns and migrations

**Key features:**
- New hooks: useActionState, useFormStatus, useOptimistic, use()
- React 19.2 additions: useEffectEvent, Activity component
- Form Actions replacing manual submit handlers
- Breaking changes: ref as prop, Context as provider
- Decision tree for choosing the right API
- Common pitfalls and migration patterns

**Example prompts:**
- "How do I handle form submissions in React 19?"
- "What's the difference between useFormStatus and useActionState?"
- "Show me how to use the Activity component for state preservation"
- "How do I migrate from forwardRef to the new ref prop pattern?"

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
