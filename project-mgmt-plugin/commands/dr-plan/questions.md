# Answer Plan Questions Interactively

This file contains instructions for resolving uncertain assumptions and open questions in a plan through interactive Q&A. It is invoked by `/dr-plan` when the user runs `/dr-plan @plan-file.md answer questions`.

**Context**: The plan file content has already been expanded into the conversation via the `@` reference. You have access to the full plan content.

---

## Instructions

### Phase 1: Identify the Plan

The plan content is available in the conversation context (expanded via `@` reference).

1. **Extract plan metadata:**
   - Plan name from the title
   - Status (Draft/In Progress/Completed)
   - Created date
   - File path (from the `@` reference)

2. **If you cannot find plan content in the context:**
   ```
   ❌ Plan content not found

   Usage: /dr-plan @_claude/plans/[folder]/[number]-[name].md answer questions
   ```
   - STOP - do not proceed

### Phase 2: Extract Unresolved Items

1. **Find uncertain assumptions:**
   - Search for `- [ ]` items in the "Assumptions Made" section
   - Especially those marked with `[?]`
   - These are assumptions that need verification

2. **Find blocking questions:**
   - Search for `[AWAITING]` status in "Open Questions & Decisions" section
   - These MUST be resolved before implementation

3. **Find non-blocking questions:**
   - Search for `[OPEN]` status in "Open Questions & Decisions" section
   - These can be resolved but don't block progress

4. **Categorize and count:**
   - Count uncertain assumptions
   - Count blocking questions
   - Count non-blocking questions
   - Total unresolved items

5. **If no unresolved items found:**
   ```
   ✅ No unresolved questions

   Plan #[number]: [Plan Name]
   Status: [Status]

   All assumptions are confirmed and all questions have been resolved.
   This plan is ready for implementation.

   Next steps:
     [If in draft/:] Move to in_progress: /dr-move-plan [number] in-progress
     [If in in_progress/:] Begin implementation following the phases
   ```
   - STOP - nothing to resolve

### Phase 3: Show Summary and Start Resolution

1. **Show summary of items to resolve:**
   ```
   📋 Plan Questions to Resolve

   Plan #[number]: [Plan Name]
   Status: [Status]

   Found [N] items needing resolution:

   Uncertain Assumptions ([count]):
     1. [assumption text] [?]
     2. [assumption text] [?]

   Blocking Questions ([count]) ⚠️  Must resolve before implementation:
     1. **[Topic]**: [question]
     2. **[Topic]**: [question]

   Non-Blocking Questions ([count]):
     1. **[Topic]**: [question]

   Starting interactive resolution...
   ```

### Phase 4: Resolve Blocking Questions First

**CRITICAL**: Blocking questions must be resolved before implementation can begin.

1. **For each blocking question (in order):**

   a. **Use AskUserQuestion tool:**
      - Question: The actual question from the plan
      - Header: The topic/title (max 12 chars)
      - Options: Extract from the plan's "Option A", "Option B" etc.
        - If no options in plan, create sensible options based on the question
        - Always include at least 2 options
      - multiSelect: false (decisions should be singular)

   b. **Wait for user response**

   c. **Document the decision:**
      - Mark as resolved: `- [x] **[Topic]** [DECIDED: YYYY-MM-DD]`
      - Add Decision line with what was chosen
      - Add Rationale based on user's explanation or the option description

2. **After all blocking questions resolved:**
   ```
   ✅ All blocking questions resolved ([count]/[count])

   Continuing to optional items...
   ```

### Phase 5: Offer to Resolve Non-Blocking Items

1. **Ask if user wants to resolve uncertain assumptions:**
   - If there are uncertain assumptions, ask:
   ```
   Would you like to verify uncertain assumptions? ([count] remaining)
   ```
   - Use AskUserQuestion with options:
     - "Yes, verify all"
     - "Skip for now"

2. **If user chooses to verify:**
   - For each uncertain assumption, ask:
     - "Is this assumption correct: [assumption text]?"
     - Options: "Yes, correct" / "No, needs change" / "Skip"
   - If correct: Mark as `[x]`
   - If needs change: Ask what the correct assumption should be, update text
   - If skip: Leave as is

3. **Ask if user wants to resolve non-blocking questions:**
   - If there are non-blocking questions, ask:
   ```
   Would you like to resolve non-blocking questions? ([count] remaining)
   ```
   - Use AskUserQuestion with options:
     - "Yes, resolve all"
     - "Skip for now"

4. **If user chooses to resolve:**
   - Follow same process as Phase 4 for each question
   - Mark as `[DECIDED: YYYY-MM-DD]` when resolved

### Phase 6: Update Plan File

1. **Create backup first:**
   - Copy current plan to `.{filename}.backup` in same directory

2. **Apply all updates:**
   - Update each resolved assumption (change `[ ]` to `[x]`)
   - Update each resolved question with:
     - Checkbox: `[x]` instead of `[ ]`
     - Status: `[DECIDED: YYYY-MM-DD]` instead of `[AWAITING]` or `[OPEN]`
     - Add `> **Decision:** [chosen option]`
     - Add `> **Rationale:** [brief explanation]`

3. **Add to Refinement History:**
   ```markdown
   - [YYYY-MM-DD]: Resolved [N] questions via interactive Q&A
   ```

4. **Update Refinements count in metadata**

### Phase 7: Confirm Success

1. **Show completion summary:**
   ```
   ✅ Questions resolved successfully

   Plan #[number]: [Plan Name]
   Location: _claude/plans/[folder]/[filename].md

   Resolution Summary:
     - Assumptions verified: [X] of [Y]
     - Blocking questions resolved: [X] of [Y] ✓
     - Non-blocking questions resolved: [X] of [Y]

   [If all blocking resolved:]
   ✓ All blocking questions resolved - ready for implementation

   [If some items skipped:]
   ℹ️  [N] items skipped - can resolve later with:
     /dr-plan @_claude/plans/[folder]/[filename].md answer questions

   Decisions made:
     - **[Topic 1]**: [Brief decision]
     - **[Topic 2]**: [Brief decision]

   Backup saved: _claude/plans/[folder]/.[filename].backup

   Next steps:
     [If in draft/ and all blocking resolved:]
     1. Move to in_progress: /dr-move-plan [number] in-progress
     [If in in_progress/:]
     1. Continue implementation following the phases
     [If items remain:]
     1. Resolve remaining items later: /dr-plan @plan answer questions
   ```

---

## Checkbox and Status Conventions

**Assumption Markers:**
- `[x]` - Confirmed assumption (verified from codebase, PRD, or user)
- `[ ]` - Unverified assumption
- `[?]` - Uncertain assumption that needs verification (placed after the assumption text)

**Question Status Labels:**
- `[AWAITING]` - Blocking question waiting for user decision
- `[OPEN]` - Non-blocking question that can be resolved during implementation
- `[DECIDED: YYYY-MM-DD]` - Question resolved with date recorded

**Resolved Question Format:**
```markdown
- [x] **[Topic]** [DECIDED: YYYY-MM-DD]
  [Original question text]
  > **Decision:** [What was decided]
  > **Rationale:** [Why this decision was made]
```

**Example Lifecycle:**
```markdown
# Before resolution:
- [ ] **Auth Method** [AWAITING]
  Which authentication method should we use?
  - Option A: JWT tokens
  - Option B: Session cookies

# After resolution:
- [x] **Auth Method** [DECIDED: 2025-01-15]
  Which authentication method should we use?
  > **Decision:** JWT tokens
  > **Rationale:** Better for API-first design and mobile app support
```

---

## Execute Now

Analyze the plan content in the conversation context, identify unresolved items, and guide the user through interactive resolution.
