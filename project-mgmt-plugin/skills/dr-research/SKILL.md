---
name: dr-research
description: Conducts deep research on a topic with web search, extended thinking, and structured documentation. Use when the user asks to research a topic, investigate a technology, compare approaches, or gather information for a decision. Produces organized markdown files with findings, visual diagrams, and actionable insights in _claude/research/.
disable-model-invocation: true
allowed-tools: WebSearch WebFetch Read Write Bash(mkdir:*)
effort: max
argument-hint: [detailed research prompt — can reference existing research path for deep dives]
---

# Deep Research

Conduct comprehensive research on a topic and produce structured, high-quality documentation.

## Phase 1: Understand & Plan

### Get the research prompt

Use `$ARGUMENTS` as the research prompt. If `$ARGUMENTS` is empty, ask the user interactively:

> What topic would you like me to research? Please provide as much detail as possible:
> - Core questions you want answered
> - Specific aspects to focus on
> - Context for why this research matters
> - Any constraints or requirements

### Detect deep-dive vs new research

Check whether the user is referencing existing research for a follow-up:

- **File/directory path provided** pointing to an existing `_claude/research/` directory → this is a **deep-dive follow-up**. Read the existing `index.md` and `findings.md` to understand prior coverage.
- **No file reference** → this is **new research**, regardless of whether the user uses phrases like "deep dive" or "go deeper." Those phrases describe desired depth, not a follow-up.
- A file reference that exists in `_claude/research/` is the key identifier. Language alone is not sufficient to trigger deep-dive mode.

### Plan the research

Read `${CLAUDE_SKILL_DIR}/references/research-methodology.md` for research types, strategies, and guidance.

1. Identify the **research type** from the user's prompt
2. Select the appropriate **research strategy** (funnel, adversarial, temporal, multi-stakeholder). If not clear, ask the user. If the user suggests a strategy in their prompt, follow their lead.
3. Determine the **output structure** — which files will be produced

### Present the plan and wait for approval

**For new research:**

```
## Research Plan: [Topic]

**Research Type:** [Type]

**Context:** [Reflect back the user's situation — what they're trying
to accomplish and why, so they can correct any misunderstandings]

**Key Questions:**
1. [Question]
2. [Question]
3. [Question]

**Research Strategy:** [Strategy name] — [Brief explanation of approach]

**Sources to Consult:**
- [Source category 1]
- [Source category 2]
- [Source category 3]

**Planned Output Structure:**
- `index.md` — Overview, key takeaways, visual concept map
- `findings.md` — [Description of what analysis will cover]
- `[topic-specific].md` — [Why this file is warranted]
- `resources.md` — Bibliography of all sources consulted
- `recommendations.md` — [Only if applicable, with brief rationale]

**Estimated Scope:** [Comprehensive/Focused — expected major findings]

Does this plan align with what you're looking for?
Would you like to adjust the focus, add questions, or change the output structure?
```

**For deep-dive follow-ups:**

```
## Deep Dive Plan: [Topic]

**Parent Research:** [Title] ([date])
**Location:** [path to parent research directory]

**Existing Coverage:**
- [file.md] covers [summary of what's already there]
- [file.md] covers [summary]
- Gap identified: [what the existing research flagged as unknown or shallow]

**Deep Dive Questions:**
1. [Question building on existing research]
2. [Question exploring the gap]
3. [Question going deeper]

**Planned Output:**
- `deep-dives/[slug]-[date]/index.md`
- `deep-dives/[slug]-[date]/findings.md`
- `deep-dives/[slug]-[date]/resources.md`
- [Additional files if warranted]

Shall I proceed, or would you like to adjust the focus?
```

**Wait for the user to approve or adjust before proceeding.**

## Phase 2: Research Execution

Execute the approved research plan.

### Research approach

- Follow the selected strategy pattern (funnel, adversarial, temporal, multi-stakeholder)
- Use **parallel WebSearch/WebFetch calls** for independent questions to improve efficiency. Don't over-fetch — parallel calls work best during the broad scan phase. Use sequential calls when later queries depend on earlier findings.
- **Progressive depth** — go deeper on findings central to the user's key questions. Stay surface-level on peripheral context.
- **Source triangulation** — cross-reference key findings across multiple sources where possible. Use best judgment on source reliability. Don't restrict source types — official docs, blog posts, forums, academic papers are all valid depending on the topic.

### Conflicting sources

When sources disagree: document both perspectives, analyze which seems more credible and why, and cite both inline with links so the user can evaluate. See `${CLAUDE_SKILL_DIR}/references/research-methodology.md` for the detailed approach and formatting.

### Circuit breaker (use sparingly)

Work to completion without interrupting the user in most cases. Contradictions, diverse viewpoints, and unexpected complexity should be documented, not used as reasons to stop.

**Stop and ask the user only if:**
1. The research premise is wrong — the technology doesn't exist, the approach isn't viable, or the question is based on a misunderstanding
2. Something truly impactful emerges that changes the entire direction of the research

Maximum 1-2 interruptions in exceptional cases. Zero in most research runs. The natural checkpoint, if needed, is after the broad scan before deep dives begin.

## Phase 3: Synthesis & Output

Read `${CLAUDE_SKILL_DIR}/references/output-formats.md` for detailed guidance on what each file should contain, quality standards, and example structures. For annotated examples of what great output looks like, see `${CLAUDE_SKILL_DIR}/examples/exemplar-index.md` and `${CLAUDE_SKILL_DIR}/examples/exemplar-findings.md`.

### Create the directory

- **New research:** `_claude/research/[slug]-[date]/`
  - Create the slug from the topic: lowercase, hyphens for spaces, remove special characters
  - Use the current date from the conversation context (YYYY-MM-DD)
  - If `_claude/research/` doesn't exist, inform the user they can run `/dr-init` for full project setup, but proceed with creating the directory
- **Deep dive:** `_claude/research/[parent-slug]-[date]/deep-dives/[deep-dive-slug]-[date]/`

### Write the files

Follow the file writing order from output-formats.md:

1. **findings.md** — Core analysis. Organize by topic in whatever structure fits naturally. Include Mermaid diagrams where they clarify workflows, architecture, system interactions, or comparisons — reference `${CLAUDE_SKILL_DIR}/references/mermaid-patterns.md` for diagram types and syntax. Flag uncertain findings with a brief note; don't mark high-confidence findings.

2. **Topic-specific files** — Only if proposed in the plan and approved by the user. Name descriptively (e.g., `comparison.md`, `implementation-guide.md`, `architecture.md`).

3. **resources.md** — Bibliography of all sources consulted. Use minimal grouping (Documentation, Articles, Repositories, etc.). Every entry gets a brief annotation. Don't force categories that don't fit.

4. **recommendations.md** — Only create when the research clearly supports actionable next steps. Skip for exploratory or knowledge-gathering research.

5. **index.md** — Written last so it reflects everything produced. Include research question and scope, key takeaways, file navigation. Encourage a visual concept map (Mermaid mindmap or flowchart) when it helps show how findings relate — but don't force it if it doesn't add value.

### Deep-dive additions

When this is a deep-dive follow-up:
- Write all output files into the `deep-dives/[slug]-[date]/` subdirectory
- The deep-dive `index.md` must link back to the parent research: `[← Back to [Parent Title]](../../index.md)`
- All deep-dive files should include a link to the parent research in their Related Documents section
- Update the parent research's `index.md` — add or update a "Deep Dives" section linking to the deep dive's `index.md` file (not the directory), with date and brief description

## Phase 4: Summary

After all files are written, present a summary to the user in the conversation.

### Completion summary

```
Research completed: [Topic]

Created: _claude/research/[slug]-[date]/
  - index.md (overview and navigation)
  - findings.md ([brief description of what's covered])
  - [additional files with descriptions]
  - resources.md (bibliography — N sources)

Key Findings:
  - [Finding 1 — the most important insight]
  - [Finding 2]
  - [Finding 3]

What surprised me:
  - [An unexpected or particularly impactful finding the user
    might not have anticipated]
```

### Deep-dive suggestions

If the research genuinely uncovered areas worth further investigation, suggest specific topics:

```
Areas worth deeper investigation:
  - [Topic] — [Why it's worth exploring further]
  - [Topic] — [What the research found that warrants a follow-up]
```

Only include when there are real gaps or promising threads. Don't manufacture suggestions.

### Follow-up actions

Suggest the most appropriate next steps for this specific research (pick 1-3):
- Deep dive: "Investigate further with `/dr-research [aspect] [path to this research]`"
- Decision: "This research supports a decision — consider documenting it as an ADR"
- PRD: "Ready to define the product? `/dr-prd [topic]`"
- Plan: "Ready to implement? `/dr-plan [topic]`"

Tailor to the research type and findings — don't always suggest the same actions.

### Deep-dive summary additions

When this was a deep-dive follow-up, also include:
- The parent research title and confirmation that its index.md was updated
- Why this deep dive was done in addition to the original research (if applicable)
- Focus on what this deep dive specifically contributes — don't rehash the parent research
