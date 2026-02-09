---
description: Conduct deep research on a topic with extended thinking
argument-hint: [detailed research prompt - can be multi-line]
allowed-tools: WebSearch, WebFetch, Read, Write, Bash(mkdir:*)
---

# Conduct Deep Research with Extended Thinking

This command conducts comprehensive research on a topic using extended thinking and documents findings in organized markdown files.

## Instructions for Claude

You are executing the `/dr-research` command to conduct deep research and create structured documentation.

### Phase 1: Gather Research Topic

**CRITICAL**: The command arguments are provided in the `<command-args>` tag at the top of this message.

1. **Check the `<command-args>` tag for arguments:**

   - If `<command-args>` contains text (not empty): Use it as the research prompt
   - If `<command-args>` is empty: Ask the user interactively for a detailed research prompt

2. **Interactive prompt (if needed):**
   - Ask: "What topic would you like me to research? Please provide as much detail as possible, including:"
   - " - Core questions you want answered"
   - " - Specific aspects to focus on"
   - " - Context for why this research matters"
   - " - Any constraints or requirements"
   - Wait for user response before proceeding

### Phase 2: Plan Research Approach

1. **CRITICAL: Get current date**

   - Use the current date from the conversation context (available in the system prompt)
   - Format as YYYY-MM-DD
   - NEVER use hardcoded or assumed dates

2. **Use extended thinking to analyze the research request:**

   - What are the key questions that need to be answered?
   - What different angles should be explored?
   - What are the most important aspects to research?
   - What sources would be most valuable?
   - How should the findings be organized?
   - What would make this research most useful?

3. **Extract core topic:**

   - Identify the main topic from the research prompt
   - Create a slug from the topic (lowercase, hyphens for spaces, remove special chars)
   - Example: "OAuth 2.1 Implementation Patterns" → "oauth-2.1-implementation-patterns"

4. **Create directory name:**
   - Combine slug with actual current date: `[slug]-[YYYY-MM-DD]`
   - Example: `oauth-2.1-implementation-patterns-2025-11-09`

### Phase 3: Conduct Research

1. **Create research directory:**

   If `_claude/research/` does not already exist, inform the user they should run `/dr-init` to set up the full project structure, but proceed with creating the directory anyway.

   ```bash
   mkdir -p _claude/research/[slug]-[date]
   ```

2. **Conduct thorough research:**

   - Use WebSearch to find current information
   - Use WebFetch to read detailed documentation, articles, and resources
   - Explore multiple perspectives and approaches
   - Look for best practices, common pitfalls, and real-world examples
   - Gather specific technical details, code examples, and case studies
   - Consider trade-offs, alternatives, and edge cases

3. **Take comprehensive notes:**
   - Document all important findings
   - Capture key insights and patterns
   - Note sources and references
   - Identify actionable recommendations

### Phase 4: Structure Documentation

1. **Create multiple interconnected markdown files:**

   **index.md** - Use `${CLAUDE_PLUGIN_ROOT}/templates/research-index-template.md` as a guide:

   - Executive summary of research and key insights
   - Navigation links to all other research files
   - Key takeaways (top 3-5 insights)
   - If additional deep-dive files were created, add links to them in the Structure section

   **findings.md** - Use `${CLAUDE_PLUGIN_ROOT}/templates/research-findings-template.md` as a guide:

   - Comprehensive analysis of the topic
   - Technical details and explanations
   - Comparison of approaches or solutions
   - Best practices and patterns
   - Common pitfalls and challenges
   - Code examples or diagrams where relevant

   **resources.md** - Use `${CLAUDE_PLUGIN_ROOT}/templates/research-resources-template.md` as a guide:

   - Links to documentation
   - Articles and blog posts
   - Case studies
   - Tools and libraries
   - Community resources
   - Each resource with brief annotation explaining its value

   **recommendations.md** - Use `${CLAUDE_PLUGIN_ROOT}/templates/research-recommendations-template.md` as a guide:

   - Actionable next steps
   - Recommended approaches
   - Implementation priorities
   - Things to avoid
   - Further research needed

2. **Create additional files as needed:**

   - If the topic is complex, create additional files for deep dives
   - Name them descriptively (e.g., `security-considerations.md`, `implementation-examples.md`)
   - Link to them from index.md

3. **Use markdown links for navigation:**
   - Link between files using relative paths: `[Text](./filename.md)`
   - Create a cohesive set of documents that work together

### Phase 5: Confirm Completion

1. **Show success message:**

   ```
   ✅ Research completed: [topic name]

   Created: _claude/research/[slug]-[date]/
     - index.md (overview and navigation)
     - findings.md (comprehensive research findings)
     - resources.md (links and references)
     - recommendations.md (actionable next steps)
     [- additional-file.md (topic-specific deep dive)]

   Key Findings:
     - [3-5 bullet points with most important insights]

   Next steps:
     - Review detailed findings in _claude/research/[slug]-[date]/
     - Create PRD if ready: /dr-prd [feature description]
     - Create implementation plan: /dr-plan [plan description]
   ```

## Important Notes

1. **Always use the current date from conversation context** - Never use hardcoded dates
2. **Use extended thinking** - Take time to deeply analyze the research request before starting
3. **Be comprehensive** - Research thoroughly and document everything important
4. **Create multiple files** - Don't put everything in one file; organize logically
5. **Add cross-references** - Link between files for easy navigation
6. **Provide value** - Make the research actionable and useful, not just informational
7. **Use templates as guides** - Read templates from `${CLAUDE_PLUGIN_ROOT}/templates/` for structure

## Execute Now

Follow the instructions above to conduct comprehensive research and create well-organized documentation.
