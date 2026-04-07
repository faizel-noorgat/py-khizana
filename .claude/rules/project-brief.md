---
paths:
  - "memory/project-brief.yaml"
---

# Project Brief Completion Rules

This file defines how to complete each section in `project-brief.yaml` during Phase 01 (PRD Discovery).

## Section Completion Order

Sections are filled incrementally during discovery conversation. Order follows natural conversation flow, not file order.

---

## raw_vision

**When:** Section 1 - Capture Free-Form Vision

**How to complete:**
- Capture user's description **verbatim** — do not restructure, summarize, or interpret
- Preserve their language, phrasing, and stream-of-consciousness structure
- This becomes source material for later restructuring

**Example:**
```yaml
raw_vision: |
  I want to build a tool for tracking my reading. I have so many books
  scattered across different apps and shelves. Sometimes I forget what
  I've read or where I put things. I want one place to see everything
  and maybe get suggestions for what to read next.
```

---

## overview

**When:** Section 2 - Understand the Problem

**How to complete:**
- Write a **one-sentence** summary: "[Problem description] → [Proposed solution]"
- Must be grounded in what user said, not AI assumptions
- Avoid vague terms (see Anti-Pattern Checklist below)

**Example:**
```yaml
overview: "Users with scattered book collections need one place to track reading history and discover what to read next."
```

---

## current_landscape

**When:** Section 2 - Understand the Problem (follow-up)

**How to complete:**
- Document what exists today: incumbent tools, manual processes, workarounds, or "nothing"
- Ask: "What do you do now to solve this problem?"
- Capture alternatives user has tried and why they fall short

**Example:**
```yaml
current_landscape: "Manual tracking in notes app, spreadsheet, or memory. Goodreads exists but feels bloated. No single tool fits workflow."
```

---

## goals

**When:** Section 2 - Understand the Problem

**How to complete:**
- Extract 2-5 goals from user's vision and problem discussion
- Each goal must be **achievable** and **measurable** where possible
- ID format: `G-001`, `G-002`, etc.

**Example:**
```yaml
goals:
  - id: "G-001"
    description: "Consolidate all book tracking in one place"
  - id: "G-002"
    description: "Reduce time spent searching for book info"
  - id: "G-003"
    description: "Provide personalized reading suggestions"
```

---

## scope

**When:** Section 7 - Define Scope Boundaries

**How to complete:**

### in_scope
- List what this effort covers
- Must align with P0 features (see product-context.yaml)
- Be specific: "X and Y" not "everything related to X"

### out_of_scope
- List what we're **explicitly not building**
- Capture adjacent problems deferred
- Ask: "What might a user ask for that we'd say 'no' to?"

**Example:**
```yaml
scope:
  in_scope:
    - "Personal book library tracking"
    - "Reading history and progress"
    - "Book discovery suggestions based on library"
  out_of_scope:
    - "Social features (friends, reviews, sharing)"
    - "E-book reader functionality"
    - "Purchase integration"
```

---

## success_criteria

**When:** Section 8 - Define Success Criteria

**How to complete:**
- Separate **LAUNCH** criteria (what must be true to ship) from **OUTCOME** criteria (what proves it's working)
- Each criterion must be **testable** — can answer "did we hit this?"
- ID format: `SC-001`, `SC-002`, etc.
- Type: `LAUNCH` or `OUTCOME`

**Example:**
```yaml
success_criteria:
  - id: "SC-001"
    type: "LAUNCH"
    description: "User can add, edit, and view all their books end-to-end"
  - id: "SC-002"
    type: "LAUNCH"
    description: "Suggestion engine produces at least 5 recommendations per library"
  - id: "SC-003"
    type: "OUTCOME"
    description: "User logs in at least once per week to track reading"
```

---

## constraints

**When:** Section 9 - Identify Constraints and Dependencies

**How to complete:**
- Document hard limits: deadlines, budget caps, team size, tech requirements
- Each constraint must have a **specific value** (date, number, tech name)
- Ask: "What would force this project to fail if we ignored it?"
- ID format: `C-001`, `C-002`, etc.

**Example:**
```yaml
constraints:
  - id: "C-001"
    description: "Must launch by end of Q2 2025"
  - id: "C-002"
    description: "Solo developer, max 10 hours/week"
  - id: "C-003"
    description: "No paid third-party services"
```

---

## dependencies

**When:** Section 9 - Identify Constraints and Dependencies

**How to complete:**
- Document external systems, teams, APIs, data sources this effort depends on
- Each dependency must have an **owner** (who controls it?)
- Ask: "What's outside our control that could block us?"
- ID format: `DEP-001`, `DEP-002`, etc.

**Example:**
```yaml
dependencies:
  - id: "DEP-001"
    description: "Google Books API for book metadata"
  - id: "DEP-002"
    description: "User's existing book data (may need import feature)"
```

---

## timeline

**When:** Section 9 - Identify Constraints and Dependencies (if deadline exists)

**How to complete:**
- Only fill if user has a specific timeline
- `start_date` and `target_completion` must be dates
- Milestones optional but recommended for longer efforts
- ID format: `M-001`, `M-002`, etc.

**Example:**
```yaml
timeline:
  start_date: "2025-04-01"
  target_completion: "2025-06-30"
  milestones:
    - id: "M-001"
      description: "Core tracking complete"
      target_date: "2025-05-15"
    - id: "M-002"
      description: "Suggestion engine integrated"
      target_date: "2025-06-15"
```

---

## notes

**When:** Any time during discovery

**How to complete:**
- Free-form capture space for anything not fitting structured fields
- Use for partial thoughts, follow-up reminders, user anecdotes
- Does not need structure — raw capture

---

## Anti-Pattern Checklist

Before finalizing any section, verify:

| ❌ Avoid | ✓ Use instead |
|----------|---------------|
| "fast" | "response time under 200ms" |
| "easy to use" | "user completes task in under 3 clicks" |
| "user-friendly" | "user satisfaction score above 4.0" |
| "intuitive" | "new user completes core flow without help" |
| "scalable" | "handles 10,000 concurrent users" |
| "secure" | "all data encrypted at rest with AES-256" |
| "reliable" | "99.9% uptime" |

**Validation questions:**
- Can a developer implement this without clarification?
- Can a tester write a pass/fail test?
- If you replaced the vague term with a number, would implementation change?

If any field fails, rephrase before documenting.