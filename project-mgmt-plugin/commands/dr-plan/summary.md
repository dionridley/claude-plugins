# Generate PR Summary from Plan

This file contains instructions for generating a Pull Request summary from a plan. It is invoked by `/dr-plan` when the user runs `/dr-plan @plan-file.md summary`.

**Context**: The plan file content has already been expanded into the conversation via the `@` reference. You have access to the full plan content.

---

## Instructions

### Step 1: Analyze the Plan Content

The plan content is available in the conversation context (expanded via `@` reference). Analyze it to understand:

- **Title/Name**: The plan heading
- **Executive Summary**: What is being implemented and why
- **Phases**: The implementation phases and their tasks
- **Success Criteria**: What defines completion
- **Key Decisions**: Any decisions made during planning
- **Completed Tasks**: Items marked with `[x]`
- **Dependencies**: What this work relies on
- **Breaking Changes**: Anything that might affect existing functionality

### Step 2: Generate the PR Summary

**Your goal:** Create a compelling, reviewer-friendly PR summary that clearly communicates what was done and why.

**Creative freedom:** You have flexibility in how you present the information. Consider using:
- Emojis to highlight sections or categories
- Tables to organize complex information
- Mermaid diagrams for architecture or flow changes
- Collapsible sections (`<details>`) for lengthy details
- Whatever formatting best conveys the changes

**Core content to include (in whatever format works best):**

1. **Summary/Overview**
   - What is the main purpose of this PR?
   - What problem does it solve?
   - High-level description of changes

2. **What Changed**
   - Key modifications organized logically (by feature, component, or phase)
   - New files, modified files, deleted files if significant
   - API changes, schema changes, configuration changes

3. **How to Test**
   - Clear instructions for verifying the changes work
   - Test commands, manual test steps, or verification criteria

4. **Important Notes for Reviewers**
   - Breaking changes (highlight prominently!)
   - Dependencies or prerequisites
   - Known limitations or follow-up work needed
   - Related issues, PRDs, or other PRs

**Quality guidelines:**
- Be concise but comprehensive
- Optimize for scannability - reviewers are busy
- Lead with the most important information
- Make testing instructions actionable
- Don't just copy/paste from the plan - synthesize and improve

### Step 3: Display the Summary

**CRITICAL OUTPUT FORMAT**: The PR summary MUST be displayed inside a single code fence so the user can copy the raw markdown. Follow this exact structure:

1. First, show the introduction (outside any code fence):
   ```
   📋 PR Summary Generated

   Plan: [Plan Name]
   Source: [file path]

   ─────────────────────────────────────────────
   📄 COPY THE MARKDOWN BELOW FOR YOUR PR
   ─────────────────────────────────────────────
   ```

2. Then output the ENTIRE PR summary wrapped in ONE code fence using TILDES (`~~~markdown`). This allows backtick code blocks inside without conflict:

   ````
   ~~~markdown
   ## 🎯 Summary

   Your summary text here...

   ## 📝 Changes

   - Change 1
   - Change 2

   ## ✅ Test Plan

   Run the tests:
   ```bash
   mix test
   ```

   ## ⚠️ Notes

   **Breaking Changes:**
   - Item here
   ~~~
   ````

   **IMPORTANT**:
   - Use `~~~markdown` (TILDES) for the outer fence, NOT backticks
   - This allows ``` code blocks inside without breaking the outer fence
   - The ENTIRE summary goes inside this ONE `~~~markdown` block
   - ALL content must be inside - headers, lists, tables, code blocks, EVERYTHING
   - The user will see `##`, `**`, `-`, ``` characters literally, which is what they need to copy

3. After the code fence closes, show the tips:
   ```
   ─────────────────────────────────────────────

   Tips:
     - Copy the content inside the code block above
     - Paste directly into GitHub's PR description

   Related commands:
     - Refine plan: /dr-plan @[plan-file] [changes]
   ```

---

## Example Formats (for inspiration)

### Simple Format
```markdown
## 🎯 Summary
Brief description of what this PR accomplishes.

## 📝 Changes
- Change 1
- Change 2

## ✅ Test Plan
How to verify this works.
```

### Detailed Format with Table
```markdown
## Summary
Description of the feature/fix.

## Changes Overview
| Category | Files | Description |
|----------|-------|-------------|
| API | 3 | New auth endpoints |
| UI | 5 | Login form components |
| Tests | 4 | Auth flow coverage |

## Test Plan
1. Step one
2. Step two
```

### Format with Diagram
```markdown
## Summary
Architecture change to support X.

## Architecture
` ` `mermaid
graph LR
    A[Client] --> B[API Gateway]
    B --> C[Auth Service]
    C --> D[Database]
` ` `

## Changes
...
```

### Format with Collapsible Sections
```markdown
## Summary
Major refactor of the data layer.

<details>
<summary>📁 Files Changed (15)</summary>

- src/data/models.ts
- src/data/repository.ts
...
</details>

## Breaking Changes ⚠️
...
```

---

## Execute Now

Generate a creative and effective PR summary from the plan content in the conversation context.

**REMINDER**: Output the PR summary inside a `~~~markdown` code fence (using TILDES) so the user sees raw markdown syntax they can copy. This allows backtick code blocks inside.
