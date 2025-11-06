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

### Option 1: Via Claude Code Plugin Manager (Coming Soon)

```bash
/plugin install project-management
```

### Option 2: Manual Installation

1. Clone or download this plugin to your Claude Code plugins directory:
   ```bash
   cd ~/.claude/plugins  # or your plugins directory
   git clone <repository-url> project-management
   ```

2. The plugin will be automatically available in Claude Code

3. Verify installation:
   ```bash
   /help
   ```
   You should see the `dr-*` commands listed.

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
   /dr-move-plan 001 in-progress
   ```
   or
   ```bash
   /dr-move-plan authentication in-progress
   ```

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

Creates detailed implementation plan OR refines existing plan using extended thinking. Supports dual-mode operation.

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
- **Creates automatic backup**: Saves `.{filename}.backup` before any changes
- **Shows diff summary**: Lists additions, modifications, and deletions
- **Requires confirmation**: Shows changes and asks y/n/diff (unless --no-confirm)
- **Status-aware behavior**:
  - **Draft plans**: All changes allowed
  - **In-progress plans**: Warns about major changes, suggests moving back to draft for restructuring
  - **Completed plans**: Refuses to modify (historical records)
- **Preserves metadata**: Keeps plan number, slug, and core structure
- **Adds refinement note**: Documents when and why plan was refined

**Plan Numbering:**
- Plans are numbered sequentially: 001, 002, 003, ..., 999, 1000, ...
- Numbers are preserved when moving between folders
- Finds highest number across ALL folders (handles case where all plans are in completed/)

### `/dr-move-plan`

Moves a plan between draft, in_progress, and completed folders while preserving the plan number.

**Usage:**
```bash
/dr-move-plan [plan-name-or-number-or-@file] [draft|in-progress|completed]
```

**Examples:**
```bash
# Move by plan number
/dr-move-plan 001 in-progress

# Move by partial name match
/dr-move-plan authentication in-progress

# Move using file reference (exact specification)
/dr-move-plan @_claude/plans/draft/001-authentication-system.md in-progress
```

**What it does:**
- **By number**: Finds plan with that number (e.g., "001")
- **By name**: Searches for plans containing that text (e.g., "auth" matches "authentication-system")
- **By file reference**: Uses exact file path via `@path` syntax (works with autocomplete)
- **Handles ambiguity**: If multiple plans match, lists all options and asks user to clarify
- **Helpful errors**: If no match found, shows all available plans
- Preserves plan number when moving
- Shows clear confirmation with plan number, name, and both locations

**Ambiguity handling:**
If you type `/dr-move-plan auth in-progress` and multiple plans match:
```
Multiple plans found matching "auth":
  1. 001-authentication-system.md (in draft/)
  2. 015-oauth-integration.md (in completed/)

Which plan did you mean? Please specify by:
  - Plan number: /dr-move-plan 001 in-progress
  - More specific name: /dr-move-plan authentication-system in-progress
  - File reference: /dr-move-plan @_claude/plans/draft/001-authentication-system.md in-progress
```

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
   /dr-move-plan [plan-name] in-progress
   ```
   - Moves plan to `in_progress/`
   - Now ready to implement
   - **IMPORTANT**: Only implement plans in `in_progress/`!

6. **Minor Adjustments During Implementation** (as needed)
   ```bash
   /dr-plan @_claude/plans/in_progress/[plan-file].md [minor changes]
   ```
   - Small adjustments only (dependencies, clarifications)
   - Major changes should move plan back to draft first

7. **Completion Phase**
   ```bash
   /dr-move-plan [plan-name] completed
   ```
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

# Start implementation
/dr-move-plan realtime-chat in-progress

# When done
/dr-move-plan realtime-chat completed
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
4. **File References in Move**: Use file references with autocomplete: `/dr-move-plan @_claude/plans/draft/001-plan.md in-progress`
5. **Partial Name Matching**: `/dr-move-plan auth in-progress` will find "authentication-system"
6. **Git Tracking**: All documentation is markdown - commit plans and PRDs to version control
7. **Idempotent Init**: Safe to run `/dr-init` multiple times - it won't duplicate files

## Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Test your changes with a real project
4. Submit a pull request

## License

MIT License - see LICENSE file for details

## Version History

- **1.0.0** - Initial release
  - Five core commands: init, research, prd, plan, move-plan
  - Dual-mode commands for PRDs and plans (create or refine)
  - Sequential plan numbering
  - Iterative refinement with extended thinking
  - PRD-plan linking via file references
  - Extended thinking integration
  - Language-agnostic design
  - Automatic backups and safety confirmations

## Support

For issues, questions, or suggestions:
- File an issue on the repository
- Check existing documentation in this README
- Review the CLAUDE.md file created by `/dr-init`

---

**Built for Claude Code** - Structured project management for modern software development
