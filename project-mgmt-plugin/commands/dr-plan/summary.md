# Generate PR Summary from Plan

This file contains instructions for generating a Pull Request summary from a plan. It is invoked by `/dr-plan` when the user runs `/dr-plan @plan-file.md summary` or `/dr-plan @plan-file.md summary <github-pr-url>`.

**Context**: The plan file content has already been expanded into the conversation via the `@` reference. You have access to the full plan content.

---

## Instructions

### Step 1: Detect GitHub PR URL

Check the `<command-args>` for a GitHub PR URL. The args will contain "summary" and may optionally contain a GitHub PR URL.

- Look for a URL matching the pattern: `https://github.com/<owner>/<repo>/pull/<number>`
- If found, store it — you will use it in Step 5 to update the PR directly on GitHub
- If not found, the summary will be displayed for the user to copy manually (standard behavior)

### Step 2: Analyze the Plan Content

The plan content is available in the conversation context (expanded via `@` reference). Analyze it to understand:

- **Title/Name**: The plan heading
- **Executive Summary**: What is being implemented and why
- **Phases**: The implementation phases and their tasks
- **Success Criteria**: What defines completion
- **Key Decisions**: Any decisions made during planning
- **Completed Tasks**: Items marked with `[x]`
- **Dependencies**: What this work relies on
- **Breaking Changes**: Anything that might affect existing functionality

### Step 3: Generate the PR Summary

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

### Step 4: Generate Branch Commit Message

1. **Analyze the plan to create the commit message:**
   - **Title**: A short, imperative phrase summarizing the overall work (3-6 words)
   - **Bullets**: Up to 5 specific changes/accomplishments from the implementation

2. **Commit message format rules:**
   - Title: Imperative mood, no period (e.g., "Add user authentication", "Fix event loading")
   - Bullets: Use `*` prefix, past tense, specific actions completed
   - Maximum 5 bullet points - prioritize the most significant changes
   - No periods at end of bullets
   - Each bullet should be a complete, standalone description

**Commit Message Examples:**

Title examples:
- "Add tools for testing"
- "Create, Edit and Delete Event functionality added"
- "Frontend app cleanup"
- "Set up account pages"
- "Stripe webhook fixes"
- "Design exploration"

Bullet examples:
- "Research on Apollo Client v4 and Codegen monorepo strategy"
- "Update dockerfile to support new elixir image"
- "Fixed eventBySlug call to properly only need slug"
- "Documented Ash framework learnings and made them part of claude.md"
- "Created database migrations for Users, Events and UserEvents"

### Step 5: Output Results

This step depends on whether a GitHub PR URL was detected in Step 1.

---

#### Path A: GitHub PR URL Detected — Update PR

If a GitHub PR URL was found in Step 1, you MUST validate the PR before making any changes.

**1. Validate the PR (REQUIRED — do this BEFORE generating any content):**

**CRITICAL**: You MUST run this command FIRST, before proceeding to steps 2-5. Do NOT skip this step.

Use the Bash tool to fetch the PR state and current body:

```
gh pr view <PR_URL> --json state,body,title
```

Then inspect the JSON output:

**Check the PR state:**
- Parse the `state` field from the JSON response
- If the `state` is NOT `OPEN` (e.g., `MERGED`, `CLOSED`), STOP immediately. Do NOT update the PR. Show:
  ```
  ❌ PR is not open (current state: <state>)

  The PR at <PR_URL> is <state> and cannot be updated.
  ```
  Then fall back to Path B to display the summary for manual use. Do NOT proceed with any `gh pr edit` commands.

**Check for existing description:**
- Parse the `body` field from the JSON response
- If the `body` field is non-empty (contains content beyond whitespace), warn the user before overwriting:
  ```
  ⚠️  This PR already has a description. Updating will replace it.
  ```
  Then use AskUserQuestion to ask: "The PR already has an existing description. Do you want to replace it with the generated summary?" with options:
  - **Replace** — Overwrite the existing PR title and description
  - **Cancel** — Keep the existing PR unchanged

  If the user chooses **Cancel**, show:
  ```
  ❌ PR update cancelled — existing description preserved.
  ```
  Then fall back to Path B to display the summary for manual use. Do NOT proceed with any `gh pr edit` commands.

- If the `body` is empty or whitespace-only, proceed without prompting.

**2. Update the PR on GitHub:**

Use the Bash tool to update the PR title and body via the GitHub CLI. Run the two commands:

- **Title**: Set to the commit message title (the first line ONLY — do NOT include the bullet points)
- **Body**: Set to the full PR summary markdown generated in Step 3

Use a heredoc for the body to safely handle special characters, backticks, and multi-line content:

```
gh pr edit <PR_URL> --title "<commit message title>" --body "$(cat <<'PRBODY'
<full PR summary markdown>
PRBODY
)"
```

**IMPORTANT**: The `--title` must be ONLY the commit message title line (e.g., "Add user authentication"). Do NOT include the bullet points in the title.

**3. Show confirmation and commit message:**

```
✅ PR Updated Successfully

PR: <PR_URL>
Title set to: <commit message title>
Description: Updated with generated summary

─────────────────────────────────────────────
📝 BRANCH COMMIT MESSAGE (for your merge/squash commit)
─────────────────────────────────────────────
```

Then output the full commit message (title AND all bullets) in a code fence using tildes so the user can review and copy it:

````
~~~
<Commit Message Title>
* <Item 1 message>
* <Item 2 message>
* <Item 3 message>
~~~
````

**4. Show tips:**

```
─────────────────────────────────────────────

Tips:
  - The PR title and description have been updated on GitHub
  - Copy the commit message above for your merge/squash commit
  - Review the PR on GitHub to verify the formatting looks correct

Related commands:
  - Refine plan: /dr-plan @[plan-file] [changes]
```

**5. If the `gh` command fails:**

Show the error and fall back to Path B (display-only mode) so the user can still copy the content manually. Prefix with:

```
⚠️  Failed to update PR: <error message>
Falling back to display mode — copy the content below manually.
```

Then continue with Path B below.

---

#### Path B: No PR URL — Display for Manual Copy

If no GitHub PR URL was found in Step 1, display the PR summary and commit message for the user to copy.

**1. Show introduction:**

```
📋 PR Summary Generated

Plan: [Plan Name]
Source: [file path]

─────────────────────────────────────────────
📄 COPY THE MARKDOWN BELOW FOR YOUR PR
─────────────────────────────────────────────
```

**2. Display the PR summary** inside a tilde code fence (`~~~markdown`) so the user can copy raw markdown. This allows backtick code blocks inside without conflict:

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

**3. Display the commit message:**

```
─────────────────────────────────────────────
📝 COPY THE COMMIT MESSAGE BELOW
─────────────────────────────────────────────
```

Then output the commit message in a tilde code fence:

````
~~~
<Commit Message Title>
* <Item 1 message>
* <Item 2 message>
* <Item 3 message>
~~~
````

**4. Show tips:**

```
─────────────────────────────────────────────

Tips:
  - Copy the PR summary content and paste into GitHub's PR description
  - Copy the commit message for your merge/squash commit

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

**REMINDERS**:
- First check for a GitHub PR URL in the command args
- If a PR URL is present: update the PR title and description via `gh pr edit`, then display the full commit message for the user to review and copy
- If no PR URL: output the PR summary inside a `~~~markdown` code fence (using TILDES) so the user sees raw markdown syntax they can copy, followed by the commit message in a separate `~~~` code fence
- The PR title (when updating GitHub) is ONLY the first line of the commit message — not the bullets
- If the `gh` command fails, fall back to display-only mode
