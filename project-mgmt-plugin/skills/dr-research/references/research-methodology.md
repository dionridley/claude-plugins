# Research Methodology Reference

This document guides how to plan and execute research. Read this during Phase 1 (planning) to select the right approach.

## Research Types

Identify which type best matches the user's request. This determines strategy selection and output structure.

| Type | Signals in User Prompt | Core Question |
|------|----------------------|---------------|
| **Technology Evaluation** | "should we use," "compare," "X vs Y," "evaluate" | Which option is the best fit? |
| **Implementation Guide** | "how to," "integrate," "set up," "implement" | How do we build this? |
| **Landscape Survey** | "what options exist," "what libraries," "overview of" | What's out there? |
| **Architecture Decision** | "how should we design," "what pattern," "structure for" | What's the right design? |
| **Best Practices** | "best practices," "patterns," "how should we approach" | What's the right way to do this? |
| **Investigation** | "why is," "what's causing," "troubleshoot," "debug" | What's going on and how do we fix it? |

If the type isn't clear from the prompt, ask the user. Multiple types can overlap — pick the primary one and note secondary aspects in the research plan.

## Research Strategies

Select the strategy that best fits the research type. The user may also suggest a strategy in their prompt — follow their lead if they do.

### Funnel Strategy (Default)

**How it works:** Start broad to map the landscape, then progressively narrow into the areas that matter most.

**Flow:** Broad scan → Identify key areas → Deep dives on high-value findings → Synthesis

**Best for:** Implementation guides, landscape surveys, best practices — most research defaults to this.

**When to use:** When you need to understand the full picture before knowing where to focus depth.

### Adversarial Strategy

**How it works:** Research the case FOR and AGAINST separately, then arbitrate between them.

**Flow:** Bull case research → Bear case research → Compare evidence → Balanced assessment

**Best for:** Technology evaluations, "should we use X?" decisions, controversial or polarizing topics.

**When to use:** When the user needs to make a choice between options and wants an honest assessment of trade-offs, not a sales pitch for one option.

### Temporal Strategy

**How it works:** Research how something evolved over time to understand where it's heading.

**Flow:** Historical context → Current state → Emerging trends → Future trajectory

**Best for:** "State of X" research, understanding why something is the way it is, predicting where a technology or approach is heading.

**When to use:** When history and trajectory matter as much as current state — e.g., researching a framework that's undergone major version changes.

### Multi-Stakeholder Strategy

**How it works:** Research the topic from multiple distinct perspectives, then synthesize into a unified assessment.

**Flow:** Perspective A research → Perspective B research → Perspective C research → Unified synthesis

**Best for:** Architecture decisions with multiple concerns (performance, security, developer experience), organizational decisions, topics where different roles have different priorities.

**When to use:** When the right answer depends on who you ask — e.g., "how should we handle auth?" has different answers from a security, UX, and backend perspective.

### Strategy-Type Pairing Guide

| Research Type | Primary Strategy | Alternative |
|---------------|-----------------|-------------|
| Technology Evaluation | Adversarial | Funnel |
| Implementation Guide | Funnel | Temporal |
| Landscape Survey | Funnel | — |
| Architecture Decision | Multi-Stakeholder | Adversarial |
| Best Practices | Funnel | Temporal |
| Investigation | Funnel | — |

If unsure which strategy to use, ask the user. Present the options briefly and let them choose.

## Source Quality and Triangulation

### Approach

Don't restrict research to specific source types. The best source depends on the topic:

- A mature framework will have excellent official docs — lean on them
- A brand-new library might only have blog posts and GitHub issues — that's fine
- A niche technique might live in academic papers or conference talks
- Community discussions (Stack Overflow, GitHub Discussions, forums) capture real-world experience that docs miss

### Triangulation

For key findings that will influence decisions:
- Try to confirm across multiple independent sources
- If only one source covers a finding, that's OK — but flag it with a confidence note in the output (see confidence flags in output-formats.md)
- Don't over-research well-established facts just to hit a source count

### Conflicting Sources

When sources disagree:
1. Document both perspectives — don't silently pick a winner
2. Offer your analysis of which seems more credible and why (recency, authority, specificity)
3. Cite both sources inline with links so the user can evaluate the competing claims directly
4. Example format in findings:

> **Conflicting information:** [Source A (date)](url) states X, while [Source B (date)](url) describes Y. Source A appears more current and references the latest version, but verify against your specific version.

### What NOT to Do

- Don't dismiss sources just because they're blog posts or forum answers
- Don't manufacture consensus by ignoring the dissenting source
- Don't over-fetch sources trying to reach a magic number — quality over quantity
- Don't chase down every tangential reference — stay focused on the key questions

## Circuit Breaker

During research execution, you should work to completion without interrupting the user. Contradictions, diverse viewpoints, unexpected complexity, and interesting tangents should all be documented in the output, not used as reasons to stop.

**Stop and ask the user only under these two conditions:**

1. **The research premise is wrong.** The broad scan reveals that the user's question is based on a misunderstanding, the technology doesn't exist, or the approach they're asking about is fundamentally not viable. Example: User asks to research "integrating Library X with Framework Y" but Library X was abandoned and replaced by Library Z two years ago.

2. **A discovery changes everything.** During deep dives, you find something so impactful that it would change the entire direction of the research. Example: User asks to research self-hosting solution options, and you discover their cloud provider just launched a managed service that eliminates the need entirely.

**Timing:**
- The natural checkpoint is after the broad scan, before deep dives begin. One pause maximum at this point.
- The second condition is an emergency brake during deep dives — no scheduled check, just permission to interrupt if genuinely warranted.
- **Goal: 0 interruptions in most research runs, 1-2 maximum in exceptional cases.**

## Parallel Research

Use parallel WebSearch and WebFetch calls where it makes sense for efficiency:

- **Good for parallel:** Independent questions that don't depend on each other (e.g., searching for library docs AND community blog posts simultaneously)
- **Bad for parallel:** Queries where the next search depends on what you find first (e.g., identifying the right library THEN reading its docs)
- Don't over-fetch without knowing what's needed next — parallel calls work best for the broad scan phase, less so for progressive deep dives

## Deep-Dive Follow-Up Detection

### How to Detect

- **Strong signal (deep dive):** User provides a file or directory path to existing research in `_claude/research/`. This is the key identifier. Load the existing research context.
- **Weak signal (new research):** User uses phrases like "deep dive" or "go further into" without referencing an existing research path. Treat as new research with thorough depth.
- **Combined signal (deep dive):** User references an existing research path AND uses follow-up language. This is a deep-dive follow-up.

Language alone ("deep dive," "go deeper") is NOT sufficient to trigger deep-dive mode. The file/directory reference must exist.

### Deep-Dive Planning

When a deep dive is detected:
1. Read the existing research (index.md and findings.md at minimum) to understand what's already covered
2. Identify gaps, shallow areas, or specific aspects the user wants to explore further
3. Present the deep-dive plan showing existing coverage and what new ground will be covered
4. Deep dives can be just as comprehensive as original research — don't artificially limit scope
