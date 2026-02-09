# Project Management Plugin for Claude Code

A structured project management system for Claude Code that provides organized workflows for research, requirements documentation (PRDs), and implementation planning.

## Features

- **Organized Directory Structure**: Standardized `_claude/` folder with subdirectories for plans, PRDs, research, and documentation
- **Research Workflow**: Deep research with extended thinking, organized into multi-file documentation
- **PRD Creation**: Comprehensive Product Requirements Documents with all critical sections
- **Implementation Planning**: Detailed, phase-based plans with sequential numbering and tracking
- **Iterative Plan Refinement**: Refine existing plans with extended thinking while preserving structure and number
- **Plan Lifecycle**: Move plans through draft → in-progress → completed stages
- **Language Agnostic**: Works with any programming language or framework
- **Git-Friendly**: All files are markdown for easy version control

## Installation

See the [marketplace README](../README.md#installation) for installation instructions.

## Quick Start

1. **Initialize your project** with the standard directory structure:
   ```bash
   /dr-init
   ```

2. **Conduct research** on a topic (10-15 line prompts recommended):
   ```bash
   /dr-research I need to research authentication best practices for Node.js applications.
   Focus on JWT vs session-based auth, password hashing algorithms, rate limiting,
   and protection against common attacks like CSRF, XSS, and session fixation.
   Include comparison of popular libraries and their security track records.
   Target audience: production web applications handling sensitive user data.
   ```

3. **Create a PRD** for your feature:
   ```bash
   /dr-prd User authentication system supporting email/password login and OAuth providers.
   Need password reset, email verification, session management, and MFA support.
   Must handle 10,000+ concurrent users with sub-100ms response times.
   Security is critical - must meet OWASP Top 10 requirements.
   Integration with existing Express.js backend and React frontend.
   ```

4. **Create an implementation plan** (optionally reference the PRD):
   ```bash
   /dr-plan Implement the authentication system as specified in the PRD.
   @_claude/prd/user-authentication-system.md
   Start with core email/password flow, then add OAuth in phase 2.
   Backend is Express.js with PostgreSQL. We have 3 weeks for initial release.
   ```

5. **Refine the plan** if needed (optional but recommended):
   ```bash
   /dr-plan @_claude/plans/draft/001-authentication-system.md Add more detail to Phase 3 about password hashing and add security best practices section
   ```

6. **Move plan to in-progress** when ready to implement:
   ```bash
   mv _claude/plans/draft/001-authentication-system.md _claude/plans/in_progress/
   ```
   Or simply ask Claude to move it - Claude will handle the move automatically when appropriate.

## Commands

### `/dr-init`

Initializes or updates project with standard directory structure and CLAUDE.md file.

**Usage:**
```bash
/dr-init
```

**What it does:**
- Creates `_claude/` directory with subdirectories:
  - `docs/` - Technical documentation
  - `plans/draft/` - Plans being reviewed
  - `plans/in_progress/` - Plans actively being implemented
  - `plans/completed/` - Completed plans
  - `prd/` - Product Requirements Documents
  - `resources/` - User-provided reference materials
  - `research/` - Research documentation
- Creates `.gitkeep` files in all leaf directories for git tracking
- Generates or updates `CLAUDE.md` with project management guidelines
- **Idempotent** - safe to run multiple times

### `/dr-research`

Conducts deep research with extended thinking and creates comprehensive multi-file documentation.

**Usage:**
```bash
/dr-research [detailed 10-15 line research prompt]
```

**Interactive mode** (if no arguments provided):
```bash
/dr-research
```
Claude will ask for research details.

**Example:**
```bash
/dr-research Research microservices architecture patterns for e-commerce platforms.
Need to understand service decomposition strategies, inter-service communication
(REST vs gRPC vs message queues), data consistency patterns (saga, 2PC),
service discovery, API gateway patterns, and deployment orchestration.
Focus on handling 100k+ daily transactions with high reliability.
Include case studies from major platforms and common pitfalls to avoid.
```

**What it does:**
- Uses extended thinking to deeply analyze the research request
- Creates timestamped directory in `_claude/research/`
- Generates multiple interconnected markdown files:
  - `index.md` - Overview and navigation
  - `findings.md` - Core research findings
  - `resources.md` - Links and references
  - `recommendations.md` - Actionable recommendations
- Cross-links all files for easy navigation
- Provides summary of key insights

### `/dr-prd`

Creates a comprehensive Product Requirements Document with extended thinking.

**Usage:**
```bash
/dr-prd [detailed 10-15 line feature description]
```

**Interactive mode:**
```bash
/dr-prd
```

**Example:**
```bash
/dr-prd Real-time notification system for social media application.
Needs to support push notifications, in-app notifications, and email digests.
Must handle notification preferences, delivery tracking, and read receipts.
Should support 1M+ users with <500ms delivery latency.
Integration with iOS/Android apps and web client.
Admin dashboard for monitoring delivery rates and user engagement.
Need templates for common notification types and A/B testing capability.
```

**What it does:**
- Uses extended thinking to analyze requirements and edge cases
- Creates PRD in `_claude/prd/` with comprehensive sections:
  - Executive Summary
  - Problem Statement
  - Goals and Non-Goals
  - User Stories
  - Functional Requirements
  - Non-Functional Requirements
  - Technical Considerations
  - Success Metrics
  - Open Questions
  - Timeline
- Thoughtfully fills ALL sections (not just placeholders)
- Identifies risks and dependencies

### `/dr-plan`

Creates detailed implementation plan OR refines existing plan using extended thinking. Supports multi-mode operation.

**CREATE Mode - Usage:**
```bash
/dr-plan [detailed implementation context - 10-15 lines]
```

**With PRD reference:**
```bash
/dr-plan [context] @_claude/prd/feature-name.md
```

**Create directly in in-progress:**
```bash
/dr-plan [context] --in-progress
```

**Interactive mode:**
```bash
/dr-plan
```

**REFINE Mode - Usage:**
```bash
/dr-plan @plan-file [refinement request]
```

**With flag to skip confirmation:**
```bash
/dr-plan @plan-file [refinement request] --no-confirm
```

**SUMMARY Mode - Generate PR Summary:**
```bash
/dr-plan @plan-file summary
```

**QUESTION RESOLUTION Mode - Answer Plan Questions:**
```bash
/dr-plan @plan-file answer questions
```

**CREATE Example:**
```bash
/dr-plan Implement real-time notification system as specified in PRD.
@_claude/prd/notification-system.md
Start with core WebSocket infrastructure and basic push notifications.
Backend is Node.js with Redis for pub/sub. Frontend uses React with Socket.io.
Phase 1: WebSocket server and connection management (1 week)
Phase 2: Push notification integration for iOS/Android (1 week)
Phase 3: Email digests and admin dashboard (1 week)
Must support 10k concurrent WebSocket connections initially.
```

**REFINE Examples:**
```bash
# Add OAuth support to existing auth plan
/dr-plan @_claude/plans/draft/001-authentication-system.md Add OAuth 2.0 support with Google and GitHub providers

# Add more detail to specific phase
/dr-plan @_claude/plans/draft/001-auth.md Add detailed code examples and security best practices to Phase 3

# Minor adjustment to in-progress plan
/dr-plan @_claude/plans/in_progress/003-database-migration.md Add Redis dependency to requirements section

# Skip confirmation for quick updates
/dr-plan @_claude/plans/draft/002-api.md Fix typo in Phase 2 tasks --no-confirm
```

**SUMMARY Examples:**
```bash
# Generate PR summary for a completed plan
/dr-plan @_claude/plans/completed/001-authentication-system.md summary

# Generate PR summary for in-progress work
/dr-plan @_claude/plans/in_progress/003-database-migration.md summary
```

**QUESTION RESOLUTION Examples:**
```bash
# Resolve open questions in a draft plan
/dr-plan @_claude/plans/draft/001-authentication-system.md answer questions

# Can also use "resolve questions"
/dr-plan @_claude/plans/draft/002-api.md resolve questions
```

**CREATE Mode - What it does:**
- Automatically assigns sequential plan number (001, 002, 003, etc.)
- Scans ALL plan folders (draft, in_progress, completed) to find the highest existing number
- If PRD referenced via `@path`, reads and analyzes it
- Uses extended thinking to break down implementation
- Creates plan file: `_claude/plans/draft/XXX-plan-name.md` (or `in_progress/` if flag used)
- Includes metadata with creation date, status, and PRD reference
- Comprehensive sections:
  - Executive Summary
  - Current State
  - Success Criteria
  - Implementation Plan (phases with tasks, time estimates, test verification)
  - Rollback Plan
  - Dependencies
  - Success Metrics

**REFINE Mode - What it does:**
- **Reads existing plan**: Analyzes current structure, phases, and content
- **Uses extended thinking**: Deeply analyzes both the existing plan and your refinement request
- **Generates refined plan**: Applies requested changes intelligently while preserving what works
- **Shows diff summary**: Lists additions, modifications, and deletions
- **Requires confirmation**: Shows changes and asks y/n/diff (unless --no-confirm)
- **Status-aware behavior**:
  - **Draft plans**: All changes allowed
  - **In-progress plans**: Warns about major changes, suggests moving back to draft for restructuring
  - **Completed plans**: Refuses to modify (historical records)
- **Preserves metadata**: Keeps plan number, slug, and core structure
- **Adds refinement note**: Documents when and why plan was refined

**SUMMARY Mode - What it does:**
- **Generates PR summary**: Creates a GitHub-ready Pull Request description from the plan
- **Creative formatting**: Uses tables, emojis, mermaid diagrams, and collapsible sections as appropriate
- **Copyable output**: Wraps output in a code fence so you can copy the raw markdown
- **Comprehensive content**: Includes summary, changes, test plan, and notes for reviewers

**QUESTION RESOLUTION Mode - What it does:**
- **Interactive Q&A**: Guides you through resolving uncertain assumptions and open questions
- **Prioritizes blocking questions**: Resolves questions marked `[AWAITING]` first
- **Updates plan file**: Marks resolved questions with `[DECIDED: date]` and documents decisions
- **Tracks progress**: Shows summary of resolved vs. remaining items

**Plan Numbering:**
- Plans are numbered sequentially: 001, 002, 003, ..., 999, 1000, ...
- Numbers are preserved when moving between folders
- Finds highest number across ALL folders (handles case where all plans are in completed/)

## Moving Plans Between Stages

Plans can be moved between `draft/`, `in_progress/`, and `completed/` folders using:

1. **Manual `mv` command:**
   ```bash
   mv _claude/plans/draft/001-plan-name.md _claude/plans/in_progress/
   mv _claude/plans/in_progress/001-plan-name.md _claude/plans/completed/
   ```

2. **Ask Claude:** Simply tell Claude to move a plan and it will handle it automatically (e.g., "move plan 001 to in-progress").

## Directory Structure

After running `/dr-init`, your project will have:

```
your-project/
├── _claude/
│   ├── docs/              # Technical documentation
│   │   └── .gitkeep
│   ├── plans/
│   │   ├── draft/         # Plans being reviewed (DO NOT IMPLEMENT)
│   │   │   └── .gitkeep
│   │   ├── in_progress/   # Plans being implemented (ONLY THESE)
│   │   │   └── .gitkeep
│   │   └── completed/     # Finished plans (archive)
│   │       └── .gitkeep
│   ├── prd/               # Product Requirements Documents
│   │   └── .gitkeep
│   ├── resources/         # User-provided materials
│   │   └── .gitkeep
│   └── research/          # Research documentation
│       └── .gitkeep
└── CLAUDE.md              # Project management guidelines
```

## Workflow

### Typical Development Flow

1. **Research Phase**
   ```bash
   /dr-research [detailed research prompt about technology/approach]
   ```
   - Creates research documentation
   - Helps make informed decisions
   - Documents alternatives considered

2. **Requirements Phase**
   ```bash
   /dr-prd [detailed feature description]
   ```
   - Creates comprehensive PRD
   - Defines scope and requirements
   - Identifies success metrics

3. **Planning Phase**
   ```bash
   /dr-plan [implementation context] @_claude/prd/feature.md
   ```
   - Creates plan in `draft/` folder
   - Automatically numbered (001, 002, etc.)
   - Review and refine the plan

4. **Refinement Phase** (optional but recommended)
   ```bash
   /dr-plan @_claude/plans/draft/[plan-file].md [refinement request]
   ```
   - Add missing details or requirements
   - Enhance specific phases
   - Adjust time estimates
   - Add security considerations
   - Can be repeated multiple times

5. **Implementation Phase**
   ```bash
   mv _claude/plans/draft/[plan-file].md _claude/plans/in_progress/
   ```
   Or ask Claude to move the plan for you.
   - Moves plan to `in_progress/`
   - Now ready to implement
   - **IMPORTANT**: Only implement plans in `in_progress/`!

6. **Minor Adjustments During Implementation** (as needed)
   ```bash
   /dr-plan @_claude/plans/in_progress/[plan-file].md [minor changes]
   ```
   - Small adjustments only (dependencies, clarifications)
   - Major changes should move plan back to draft first

7. **PR Summary** (before or after completion)
   ```bash
   /dr-plan @_claude/plans/in_progress/[plan-file].md summary
   ```
   - Generates GitHub-ready PR description
   - Includes summary, changes, test plan, and notes
   - Output is copyable markdown

8. **Completion Phase**
   ```bash
   mv _claude/plans/in_progress/[plan-file].md _claude/plans/completed/
   ```
   Or ask Claude to move the plan to completed.
   - Archives completed plan
   - Documents lessons learned
   - Updates success metrics

## Important Rules

### Plan Execution Rules

⚠️ **CRITICAL**:
- **NEVER** implement tasks from plans in `draft/` folder
- **ONLY** work on plans in `in_progress/` folder
- Draft plans are for review and refinement only
- If you need to work on a draft plan, move it to `in_progress/` first

### Prompt Best Practices

All commands support detailed multi-line prompts for best results:

✅ **GOOD** - Detailed, 10-15 line prompts:
```bash
/dr-research GraphQL federation architecture for microservices.
Need to understand schema composition, service boundaries, entity resolution,
query planning optimization, and error handling patterns.
Focus on Apollo Federation 2.5+ features and best practices.
Consider performance implications for 100+ requests/second.
Include tooling recommendations for development and production.
```

❌ **NOT IDEAL** - Single line without context:
```bash
/dr-research GraphQL federation
```

The more context you provide, the better Claude can analyze and produce comprehensive, thoughtful output using extended thinking.

## Examples

### Example 1: New Feature Development

```bash
# Research the technology
/dr-research WebSocket scaling patterns for Node.js applications handling
10k+ concurrent connections. Need to understand cluster mode, Redis pub/sub,
sticky sessions, connection management, heartbeat strategies, and
horizontal scaling approaches. Include AWS and Docker deployment patterns.

# Create requirements document
/dr-prd Real-time chat feature for customer support platform.
Support 1-on-1 conversations and group chats up to 50 participants.
Need typing indicators, read receipts, file attachments up to 10MB,
message history with search, and offline message queuing.
Must integrate with existing user auth and notification systems.

# Create implementation plan referencing the PRD
/dr-plan Implement real-time chat as specified in PRD.
@_claude/prd/realtime-chat-feature.md
Use Socket.io with Redis adapter for multi-server support.
Existing stack: Express.js backend, React frontend, PostgreSQL database.
Have 4 weeks for initial release with basic features.

# Review and refine if needed
/dr-plan @_claude/plans/draft/001-realtime-chat.md Add detailed error handling strategy and enhance security phase

# Start implementation (move to in_progress)
mv _claude/plans/draft/001-realtime-chat.md _claude/plans/in_progress/

# When done (move to completed)
mv _claude/plans/in_progress/001-realtime-chat.md _claude/plans/completed/
```

### Example 2: Technical Spike / Investigation

```bash
# Research without PRD or plan
/dr-research Database migration from PostgreSQL to MongoDB for our analytics service.
Need to understand data modeling differences, query pattern translation,
performance characteristics for time-series data, backup and restore procedures,
migration tooling options, and potential gotchas. We handle 100GB of analytics
data with 1M+ events per day. Focus on minimal downtime migration strategies.

# Review research, then create plan if moving forward
/dr-plan Migrate analytics service database from PostgreSQL to MongoDB based on
research findings. Start with data model redesign, then migrate historical data,
then switch over production traffic with blue-green deployment strategy.
```

## Tips

1. **Use Extended Thinking**: Commands automatically use extended thinking when you provide detailed prompts
2. **Reference Files**: Use `@path/to/file` syntax to reference PRDs or other files in your prompts
3. **Plan Numbering**: Plans auto-number sequentially - no need to manage numbers manually
4. **Moving Plans**: Use `mv` command or ask Claude to move plans between draft/in_progress/completed folders
5. **Git Tracking**: All documentation is markdown - commit plans and PRDs to version control
6. **Idempotent Init**: Safe to run `/dr-init` multiple times - it won't duplicate files
7. **PR Summaries**: Use `/dr-plan @plan.md summary` to generate copy-paste ready PR descriptions
8. **Resolve Questions Early**: Use `/dr-plan @plan.md answer questions` to resolve blocking questions before implementation

## Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Test your changes with a real project
4. Submit a pull request

## License

MIT License - see LICENSE file for details

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and release notes.

## Troubleshooting

### Commands not appearing in `/help`

**Problem**: After installation, the `dr-*` commands don't show up.

**Solutions**:
1. Verify plugin is in correct directory: `~/.claude/plugins/project-management/`
2. Check that `plugin.json` exists in `.claude-plugin/` subdirectory
3. Restart Claude Code session
4. Run `/plugin list` to verify plugin is recognized

### `/dr-init` says "already initialized" but structure is incomplete

**Problem**: Running init reports success but some folders are missing.

**Solution**: This is expected - init only reports what it created. Check the actual directories:
```bash
ls -la _claude/plans/
```
If folders are truly missing, the command should have created them. Try running `/dr-init` again.

### Plan numbering skipped a number

**Problem**: Expected plan 003 but got 004.

**Explanation**: Plan numbers are determined by scanning all three folders (draft, in_progress, completed). If a plan 003 exists anywhere, even in completed/, the next plan will be 004. This is intentional to maintain chronological ordering.

### Can't find a plan file to move

**Problem**: Not sure where a plan file is located.

**Solutions**:
1. List all plans: `ls _claude/plans/*/`
2. Search by name: `ls _claude/plans/*/*.md | grep -i "auth"`
3. Ask Claude to find and move the plan for you

### PRD or Plan refinement not working

**Problem**: Refine mode doesn't detect my file.

**Solutions**:
1. Ensure path starts with `@`: `/dr-prd @_claude/prd/my-feature.md [changes]`
2. For plans, path must include `_claude/plans/`: `/dr-plan @_claude/plans/draft/001-plan.md [changes]`
3. File must exist before refinement

### Extended thinking not being used

**Problem**: Output seems shallow or template-like.

**Solutions**:
1. Provide more detailed prompts (10-15 lines recommended)
2. Include specific context about your project, constraints, and requirements
3. Ask specific questions rather than generic requests

### Git not tracking empty directories

**Problem**: After cloning, `_claude/` structure is incomplete.

**Explanation**: This is a git limitation. The `.gitkeep` files ensure directories are tracked. If missing:
```bash
/dr-init
```
This will recreate any missing directories and `.gitkeep` files.

### CLAUDE.md was modified unexpectedly

**Problem**: CLAUDE.md contents changed after running commands.

**Clarification**: The plugin NEVER modifies CLAUDE.md after initial creation (State A) or user-approved append (State C). If changes occurred:
1. Check git history to see what changed
2. A different tool or user may have modified it
3. The plugin only creates/modifies CLAUDE.md during `/dr-init` with user consent

## Support

For issues, questions, or suggestions:
- File an issue on the repository
- Check existing documentation in this README
- Review the CLAUDE.md file created by `/dr-init`

---

**Built for Claude Code** - Structured project management for modern software development
