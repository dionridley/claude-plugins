# Mermaid Diagram Patterns for Research

This document provides diagram patterns to use in research output. Reference this when you determine a diagram would help clarify a concept.

## When to Use Diagrams

**Strong candidates for diagrams:**
- Workflows and processes that have multiple steps or decision points
- Architecture and system design — how components connect and interact
- Dense technical concepts that are hard to explain in prose alone
- Comparisons between multiple options on two or more dimensions
- Sequences of interactions between systems or actors
- Data models and entity relationships

**Not worth diagramming:**
- Simple lists or single-dimension comparisons (use a table instead)
- Concepts that are already clear from a short paragraph
- Trivial flows with only 2-3 linear steps

Use your judgment. A diagram should save the reader time understanding a concept, not add decoration.

## Diagram Types and When to Use Each

### Flowchart — Decision Processes and Data Flow

Best for: Auth flows, request lifecycles, decision trees, build pipelines, any process with branching logic.

```mermaid
flowchart LR
    A[Client Request] --> B{Authenticated?}
    B -->|Yes| C[Process Request]
    B -->|No| D[Return 401]
    C --> E{Authorized?}
    E -->|Yes| F[Return Data]
    E -->|No| G[Return 403]
```

Use `LR` (left-to-right) for horizontal flows, `TD` (top-down) for vertical. Horizontal works better for linear pipelines; vertical works better for decision trees.

### Sequence Diagram — System Interactions

Best for: API call sequences, auth handshakes, webhook flows, any interaction between 2+ systems over time.

```mermaid
sequenceDiagram
    participant U as User
    participant A as App
    participant S as External Service

    U->>A: Initiate action
    A->>S: API request
    S-->>A: Response
    A-->>U: Show result

    Note over A,S: Async callback
    S->>A: Webhook notification
    A->>A: Process event
```

Use solid arrows (`->>`) for requests, dashed arrows (`-->>`) for responses. Add `Note over` for context.

### Mindmap — Concept Relationships and Topic Overview

Best for: Research topic decomposition, showing how findings relate, concept maps in index.md.

```mermaid
mindmap
  root((Topic))
    Subtopic A
      Key Finding
      Related Detail
    Subtopic B
      Key Finding
      Trade-off Noted
    Subtopic C
      Key Finding
```

Good for the visual concept overview in index.md — shows the reader the shape of the research at a glance.

### Block Diagram — Architecture Overview

Best for: System architecture, deployment topology, component relationships, infrastructure layouts.

```mermaid
block-beta
    columns 3
    Frontend["Frontend\nReact App"]:1
    API["API Gateway"]:1
    Auth["Auth Service"]:1
    space:1
    Backend["Backend\nPhoenix/Ash"]:1
    space:1
    space:1
    DB[("Database\nPostgres")]:1
    Cache[("Cache\nRedis")]:1

    Frontend --> API
    API --> Backend
    Backend --> DB
    Backend --> Cache
    Auth --> Backend
```

### Quadrant Chart — Two-Axis Comparison

Best for: Comparing options on two dimensions (complexity vs capability, cost vs performance, effort vs impact).

```mermaid
quadrantChart
    title Technology Comparison
    x-axis Low Complexity --> High Complexity
    y-axis Low Capability --> High Capability
    quadrant-1 "Ideal Choice"
    quadrant-2 "Power but Complex"
    quadrant-3 "Simple but Limited"
    quadrant-4 "Avoid"
    Option A: [0.3, 0.8]
    Option B: [0.7, 0.9]
    Option C: [0.2, 0.4]
```

Useful in technology evaluations and landscape surveys to visually position options.

### Entity Relationship Diagram — Data Models

Best for: Database schemas, API entity relationships, data model documentation.

```mermaid
erDiagram
    USER ||--o{ SUBSCRIPTION : has
    USER {
        string id PK
        string email
        string stripe_customer_id
    }
    SUBSCRIPTION ||--|{ INVOICE : generates
    SUBSCRIPTION {
        string id PK
        string status
        date current_period_end
    }
    INVOICE {
        string id PK
        int amount
        string status
    }
```

### Gantt Chart — Implementation Timelines

Best for: Phased implementation plans, project timelines in recommendations.

```mermaid
gantt
    title Implementation Phases
    dateFormat YYYY-MM-DD
    section Phase 1
        Setup & Configuration    :a1, 2026-01-01, 5d
        Core Integration         :a2, after a1, 10d
    section Phase 2
        Advanced Features        :b1, after a2, 10d
        Testing                  :b2, after b1, 5d
    section Phase 3
        Production Deploy        :c1, after b2, 3d
```

Use in recommendations.md when suggesting a phased approach.

## Formatting Tips

- Keep diagrams focused — if it's getting complex, split into multiple smaller diagrams
- Add labels and descriptions on arrows/connections when the relationship isn't obvious
- Use consistent naming across diagrams in the same research (same component names)
- Test that the Mermaid syntax is valid — broken diagrams are worse than no diagrams
- Prefer horizontal (`LR`) flowcharts for processes and vertical (`TD`) for hierarchies
