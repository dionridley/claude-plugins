# Output Formats Reference

This document guides how to structure research output files. Read this during Phase 3 (synthesis) when writing the research documentation.

## General Principles

- **Prescribe quality, not rigid structure.** Every finding must have supporting evidence. Every claim must be traceable to a source. But the exact section headings and organization should adapt to the topic.
- **Write for a developer who will act on this research.** Be specific, include code examples when relevant, and make the research actionable — not academic.
- **Use Mermaid diagrams where they clarify.** Workflows, architecture, system interactions, comparisons, and dense technical concepts benefit from visual representation. See `mermaid-patterns.md` for patterns. Don't force diagrams where they don't add value.
- **Flag uncertainty, not certainty.** Only call out findings based on limited sources, single blog posts, or potentially outdated information. High-confidence findings don't need markers — the reader assumes confidence by default.

## File Writing Order

Write files in this order:
1. `findings.md` — core analysis (written first because everything else depends on it)
2. Topic-specific files — comparison matrices, implementation guides, etc. (if warranted)
3. `resources.md` — bibliography (written after research is complete)
4. `recommendations.md` — actionable next steps (only if warranted)
5. `index.md` — overview and navigation (written last so it reflects everything produced)

## index.md

**Purpose:** The landing page for the research. Gives the reader a quick understanding of what was researched, what was found, and where to find details.

**Written last** so it accurately reflects everything that was produced.

**Should include:**
- Research question and scope (what was and wasn't researched)
- 1-2 paragraph overview summarizing the key insight
- Key takeaways (top 3-5 bullet points — the most important things the reader should know)
- Visual concept map — encouraged when it helps show how findings relate to each other. Use a Mermaid mindmap, flowchart, or other diagram type that fits. Not required for every research output — use judgment on whether it adds value.
- File navigation with brief descriptions of what each file contains
- Deep Dives section (only in parent research when deep-dive follow-ups exist)

**Example structure:**

```markdown
# Research: [Topic]

**Date:** [YYYY-MM-DD]
**Research Question:** [The specific question or area investigated]

## Overview

[1-2 paragraphs summarizing the most important insight from this research.
Not a list of everything found — the single most valuable thing the reader
should take away.]

## Key Takeaways

1. **[Takeaway 1]** — [Brief explanation of why this matters]
2. **[Takeaway 2]** — [Brief explanation]
3. **[Takeaway 3]** — [Brief explanation]

## Concept Overview

[Mermaid diagram showing how the key concepts/findings relate — when it adds value]

## Research Files

- **[Findings](./findings.md)** — [Brief description of what's covered]
- **[Topic-Specific File](./topic-file.md)** — [Why this file exists]
- **[Resources](./resources.md)** — Bibliography of all sources consulted
- **[Recommendations](./recommendations.md)** — Actionable next steps

## Deep Dives

- **[Deep Dive Topic](./deep-dives/slug-date/)** — [Date] — [Why this follow-up was done]
```

## findings.md

**Purpose:** The core analytical document. This is where the real value of the research lives — thorough analysis with evidence, not just a summary of what was found.

**Should include:**
- Executive summary (2-3 sentences, the essence of findings)
- Detailed findings organized by topic — use whatever structure fits the research naturally. Don't force a rigid "Finding 1, Finding 2" pattern if a different organization makes more sense.
- For each finding: what was learned, why it matters, and evidence supporting it
- Mermaid diagrams where they help explain workflows, architecture, interactions, or comparisons
- Cross-cutting themes — patterns that span multiple findings
- Gaps and limitations — what we still don't know, what wasn't researched, areas of uncertainty

**Quality standards:**
- Every substantive claim should be traceable to a source (inline link or reference to resources.md)
- Findings should include analysis, not just facts — "what does this mean for us?"
- Include code examples when they make a concept concrete
- Use tables for structured comparisons (feature matrices, compatibility tables, etc.)

**Confidence flags (use sparingly):**

Only flag findings where the reader should be skeptical. Don't mark high-confidence findings.

Format for uncertain findings — use a blockquote callout:

```markdown
> **Note:** This finding is based on a single blog post from 2024. 
> The library has had two major releases since then. Verify before 
> relying on this for implementation decisions.
```

**Conflicting sources — cite inline:**

When sources disagree, document both sides and cite inline so the reader can evaluate:

```markdown
> **Conflicting information:** [Official docs (2026-03)](https://...) 
> state that runtime compilation was removed in v3.0, while 
> [this blog post (2025-08)](https://...) describes runtime compilation 
> as the recommended approach. The blog post appears to reference v2.x. 
> The official docs are likely current, but verify which version you're 
> targeting.
```

**Example structure:**

```markdown
# Research Findings: [Topic]

**Date:** [YYYY-MM-DD]

[← Back to Index](./index.md)

## Executive Summary

[2-3 sentences capturing the essence of what was found]

## [Topic Area 1]

[Analysis, evidence, code examples, diagrams as appropriate]

## [Topic Area 2]

[Analysis organized in whatever way fits this topic naturally]

## [Topic Area N]

[...]

## Cross-Cutting Themes

1. **[Theme]:** [How this pattern appears across multiple findings]
2. **[Theme]:** [...]

## Gaps and Limitations

- [What we don't know and why it might matter]
- [Limitations of the sources available]
- [Areas that would benefit from deeper investigation]

## Related Documents

- [Index](./index.md) — Research overview
- [Resources](./resources.md) — All sources consulted
```

## resources.md

**Purpose:** A bibliography of all sources consulted during research. A single reference point for where information came from.

**Always created.** Even flexible research needs a source list.

**Use minimal grouping** for readability — enough structure to be useful, not so much that it feels forced. Group by type when natural:

```markdown
# Research Resources: [Topic]

**Date:** [YYYY-MM-DD]

[← Back to Index](./index.md)

## Documentation

- [Resource title](URL) — Brief annotation of what this covers and why it's relevant

## Articles & Blog Posts

- [Resource title](URL) — Author/publication — Brief annotation
- [Resource title](URL) — Brief annotation

## Repositories & Libraries

- [Repository name](URL) — Stars, language — Brief annotation

## Community Discussions

- [Thread title](URL) — Platform — Brief annotation of key insight

## Related Documents

- [Index](./index.md) — Research overview
- [Findings](./findings.md) — Core research findings
```

**Guidelines:**
- Every entry should have a brief annotation explaining its relevance — not just a bare URL
- Don't force categories. If all sources are docs and blog posts, just use those two groups.
- This is a bibliography, not a curated reading list. Include everything consulted, not just the "best" sources.

## recommendations.md

**Purpose:** Actionable next steps when the research clearly supports them.

**Conditional — only create this file when:**
- The research is decision-oriented (technology evaluation, architecture decision)
- The research is implementation-focused and there are clear steps to take
- The findings naturally lead to "here's what you should do"

**Do NOT create this file when:**
- The research is purely exploratory ("what options exist?")
- The research is knowledge-gathering without a clear action to take
- Recommendations would feel forced or premature

**Should include:**
- Executive summary of top recommendations
- Prioritized next steps with rationale (why this order?)
- What to avoid — anti-patterns, common pitfalls
- Implementation phases (when the research supports phased rollout)
- Risks and mitigation strategies
- Questions for further investigation

**Example structure:**

```markdown
# Research Recommendations: [Topic]

**Date:** [YYYY-MM-DD]

[← Back to Index](./index.md)

## Summary

[Brief summary of the top recommendation and why it matters]

## Recommended Approach

[What to do and why — the primary recommendation with rationale]

## Next Steps

### Priority 1: [Action]
**Why:** [Rationale for this being first]
**What:** [Specific actions]

### Priority 2: [Action]
**Why:** [Rationale]
**What:** [Specific actions]

## What to Avoid

- **[Anti-pattern]:** [Why to avoid and what to do instead]
- **[Common pitfall]:** [How to prevent]

## Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| [Risk] | [High/Med/Low] | [How to address] |

## Open Questions

- [Question that would provide additional clarity if investigated]
- [Question for the team to discuss]

## Related Documents

- [Index](./index.md) — Research overview
- [Findings](./findings.md) — Core research findings
- [Resources](./resources.md) — All sources consulted
```

## Topic-Specific Files

**Purpose:** Additional files created when the research warrants dedicated deep dives on specific aspects.

**These are proposed in the research plan** and approved by the user during Phase 1. They can also be suggested by the user when adjusting the plan.

**Common examples:**
- `comparison.md` — Side-by-side evaluation of multiple options (technology evaluations)
- `implementation-guide.md` — Step-by-step integration guide (implementation research)
- `architecture.md` — System design analysis with diagrams (architecture decisions)
- `decision-record.md` — ADR-style document capturing the decision and rationale
- `[technology-name]-guide.md` — Deep dive on a specific technology within the broader research

**Guidelines:**
- Name files descriptively — the filename should make its contents obvious
- Each file should be self-contained enough to be useful on its own
- Include navigation links back to index.md and to related files
- Use whatever internal structure fits the content — these files are the most flexible

## Deep-Dive Output Structure

When research is a follow-up deep dive on existing research:

**Directory structure:**
```
_claude/research/[parent-slug]-[date]/
├── index.md                              (updated with deep-dive link)
├── findings.md
├── resources.md
└── deep-dives/
    └── [deep-dive-slug]-[date]/
        ├── index.md
        ├── findings.md
        ├── resources.md
        └── [additional files as warranted]
```

**Deep-dive index.md additions:**
- Link back to the parent research index: `[← Back to [Parent Title]](../../index.md)`
- Reference the parent research title and date
- Explain why this deep dive was done (what gap or question it addresses)
- Don't rehash the parent research — focus on what's new

**Deep-dive navigation pattern:**
All files within a deep dive should include a link back to the parent research in their Related Documents section:

```markdown
## Related Documents

- [← Parent Research: [Title]](../../index.md) — Original research overview
- [Index](./index.md) — This deep dive's overview
- [Findings](./findings.md) — Deep dive findings
```

**Parent index.md update:**
- Add or update a "Deep Dives" section
- Link to the deep-dive's `index.md` file directly — not the directory. Directory links break in markdown viewers.
  - Correct: `[Deep Dive Title](./deep-dives/slug-date/index.md)`
  - Wrong: `[Deep Dive Title](./deep-dives/slug-date/)`
- Include date and brief description of what the deep dive covers
