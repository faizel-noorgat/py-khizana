---
name: generate-user-stories
description: Generate user stories by spawning persona agents that write from each user's perspective. Use after generate-personas skill completes, or when you have approved personas and need user stories. Triggers on "create user stories", "write stories for these personas", or "generate requirements from personas".
---

# Generate User Stories from Personas

Spawn agents that roleplay each persona and write user stories that help the user discover features they want.

## Purpose

User stories are **feature discovery tools**, not just documentation. They help the user:
- See possibilities they hadn't considered
- Get excited about features ("oh yes, I want that!")
- Spark new ideas ("that makes me think of...")
- Prioritize what matters most

The goal is to get the user's creative juices flowing and help them identify what features to build.

## Process

### Step 1: Load Personas

Read personas from `$ARGUMENTS`. This may be:
- A file path (e.g., `memory/product-context.yaml` containing `user_personas`)
- Direct YAML personas passed as arguments
- Reference to personas from prior generate-personas skill output

**Error handling:**
- If personas file does not exist, ask the user to run `/generate-personas` first or provide personas directly
- If personas list is empty, invoke `/generate-personas` automatically before proceeding
- If a persona is missing required fields (name, goals), skip it and warn the user

Once loaded, extract each persona's:
- `id` — Used to link stories (US-001 → P-001)
- `name` — Used in story format ("As a [name]")
- `goals` — What stories should help achieve
- `pain_points` — What stories should address
- `technical_level` — Influences acceptance criteria detail level
- `is_primary` — Primary personas get more stories (4-5 vs 2-3)

### Step 2: Spawn Persona Agents

For each persona, spawn an Agent using the Agent tool. Each agent takes on that persona's identity and writes stories from their perspective.

**Agent prompt structure:**

Build the prompt by filling in persona details:

```
You are {persona.name}, a {persona.role}.

Your situation: {persona.description}

Your goals:
{for each goal: - {goal}}

Your pain points:
{for each pain_point: - {pain_point}}

Your technical level: {persona.technical_level}

You are evaluating a system being built to help you. Write 3-5 user stories
from YOUR perspective — features YOU would genuinely want.

CRITICAL FORMAT REQUIREMENT:
Each story MUST use this EXACT format on a single line:
"As {your persona name}, I want to [specific action], So that [specific benefit]."

Examples of CORRECT format:
- "As Book Collector, I want to see which books belong to a series and their reading order, So that I can spot missing volumes and read them in sequence."
- "As Thesis Writer, I want to search my PDF highlights from all papers, So that I can find quotes I highlighted months ago."

Examples of WRONG format (DO NOT USE):
- "As a Book Collector, I want..." (wrong: "a" before name)
- Multi-line story format (wrong: must be single line)
- Missing "So that" clause (wrong: must include benefit)

For each story, also provide:
- 2-3 acceptance criteria in Given/When/Then format
- A suggested feature name (short, like "Series Tracker" or "Highlight Search")

Write stories that would make YOU excited — features you'd actually use.
```

**Spawn agents in parallel:**

```json
Agent({
  "subagent_type": "general-purpose",
  "description": "Write stories as [persona name]",
  "prompt": "[full prompt with persona details filled in]"
})
```

Spawn all persona agents at once in a single message with multiple Agent tool calls.

**Error handling for agent failures:**
- If an agent fails to return valid YAML, retry once with simpler instructions
- If still failing, generate stories for that persona manually based on their goals and pain points
- If an agent times out, proceed with stories from other personas and note the gap

### Step 3: Collect and Extract Features

Each agent returns user stories written from that persona's perspective. Collect all outputs and:

1. **Validate format** — Each story must follow: "As {name}, I want to {action}, So that {benefit}." Fix any that don't match.

2. **Extract feature names** — Each story should suggest a feature name. Create a consolidated feature list.

3. **Assign IDs** sequentially:
- P-001's stories: US-001, US-002...
- P-002's stories: US-003, US-004...
- Feature IDs: FTR-001, FTR-002...

**Output structure:**

Write to **`memory/user-stories.yaml`**:
```yaml
user_stories:
  - id: "US-001"
    persona_id: "P-001"
    title: "Series Tracker"
    story: "As Book Collector, I want to see which books belong to a series and their reading order, So that I can spot missing volumes and read them in sequence."
    acceptance_criteria:
      - "Given I have books from 'The Stormlight Archive', when I view the series, then I see books 1-5 in order with gaps highlighted"
      - "Given a book is part of a series, when I view its details, then I see 'Book 3 of 5'"
    priority: "P0"
    status: "PENDING"
    notes: ""
```

Write to **`memory/product-context.yaml`** (features only):
```yaml
features_discovered:
  - id: "FTR-001"
    name: "Series Tracker"
    description: "See book series order and spot missing volumes"
    discovery_status: "WANTED"
    priority_hint: "HIGH"
    requested_by: ["P-001"]
    user_stories: ["US-001"]
```

### Step 4: Present for Feature Discovery

Display stories grouped by persona, but frame them as **feature suggestions** to spark the user's thinking.

**First, show a quick summary:**

```
I've identified [N] potential features from [M] user perspectives:

| Feature | Who Wants It | Priority Hint |
|---------|--------------|---------------|
| Series Tracker | Book Collector | HIGH (primary user) |
| Duplicate Finder | Book Collector, Thesis Writer | HIGH (multiple users) |
| Semantic Search | All personas | HIGH (core feature) |
| Version Control | Corporate Worker | MEDIUM |
| Annotation Search | Thesis Writer | MEDIUM |
```

**Then use AskUserQuestion to spark ideation:**

```json
AskUserQuestion({
  "questions": [
    {
      "header": "Excitement",
      "question": "Which of these features excite you? Select all that make you think 'yes, I want that!'",
      "multiSelect": true,
      "options": [
        {"label": "Series Tracker", "description": "See book series order, spot missing volumes"},
        {"label": "Duplicate Finder", "description": "Identify same file downloaded multiple times"},
        {"label": "Semantic Search", "description": "Find files by describing what you remember"},
        {"label": "Something else sparked an idea", "description": "A story made me think of a different feature I want"}
      ]
    }
  ]
})
```

**If user selects "Something else sparked an idea":**
- Ask them to describe the feature they're thinking of
- Add it to the feature list
- This is the CORE VALUE — helping them discover features they hadn't articulated

**After initial selection:**
- Show the full stories for selected features
- Ask if any acceptance criteria need adjustment
- Allow adding new features or modifying existing ones

### Step 5: Iterate and Add User Ideas

Based on user feedback:
- Mark features as "wanted", "maybe", or "not needed"
- Add new features the user thought of during review
- Refine acceptance criteria if needed
- Re-prioritize based on user enthusiasm

This step is where the real discovery happens — the user often realizes they want things they hadn't considered.

### Step 6: Document

After user confirms their feature selections, save to two files:

**1. `memory/user-stories.yaml` — All user stories:**

```yaml
user_stories:
  - id: "US-001"
    persona_id: "P-001"
    title: "Series order viewing"
    story: "As Book Collector, I want to see which books belong to a series and their reading order, So that I can spot missing volumes and read them in sequence."
    acceptance_criteria:
      - "Given I have books from a series, when I view series view, then I see all volumes in order with gaps highlighted"
      - "Given a book is in a series, when I view its details, then I see 'Book X of Y'"
    priority: "P0"
    status: "PENDING"
    notes: ""
```

**2. `memory/product-context.yaml` — Features only:**

```yaml
features_discovered:
  - id: "FTR-001"
    name: "Series Tracker"
    description: "See book series order and spot missing volumes"
    discovery_status: "WANTED"
    priority_hint: "HIGH"
    requested_by: ["P-001"]
    user_stories: ["US-001"]

features:
  - id: "FTR-001"
    description: "Series Tracker - See book series order and spot missing volumes"
    priority: "P0"
    status: "PENDING"
    requested_by: ["P-001"]
    user_stories: ["US-001"]
```

**Priority conversion rule:**
| discovery_status | priority_hint | features priority |
|------------------|---------------|-------------------|
| WANTED | HIGH | P0 |
| WANTED | MEDIUM | P1 |
| WANTED | LOW or none | P2 |
| MAYBE | (any) | Not added to features |
| NOT_NEEDED | (any) | Not added to features |

**Note:** MAYBE features stay in `features_discovered` for future review but are not added to the active `features` list.

Announce: "Identified [N] features. [X] marked as WANTED (added to features), [Y] as MAYBE (archived for review). Ready for next phase."

## Example

**Input personas:**

```yaml
user_personas:
  - id: "P-001"
    name: "Book Collector"
    role: "Hobbyist with 5,000+ ebook library"
    goals:
      - See proper book covers and metadata
      - Find any book by plot description
      - Spot missing volumes in series
    pain_points:
      - Generic filenames like "download_3847.epub"
      - Can't tell which books in a series I own
      - Duplicates from re-downloading
    technical_level: "MEDIUM"
    is_primary: true
```

**Agent output (correct format):**

```yaml
user_stories:
  - id: "US-001"
    persona: "P-001"
    story: "As Book Collector, I want to see which books belong to a series and their reading order, So that I can spot missing volumes and read them in sequence."
    feature: "Series Tracker"
    acceptance_criteria:
      - "Given I have books from 'The Stormlight Archive', when I view the series, then I see books 1-5 in order with books 3-5 marked as missing"
      - "Given a book is part of a series, when I view its details, then I see 'Book 2 of 5' and can click to see the full series"
    priority: "HIGH"

  - id: "US-002"
    persona: "P-001"
    story: "As Book Collector, I want to find a book by describing its plot, So that I can locate books even when I forget the title."
    feature: "Semantic Search"
    acceptance_criteria:
      - "Given I search for 'wizard school with scar', when results appear, then Harry Potter books are shown"
      - "Given I search by plot description, when I view results, then I see a relevance score for each match"
    priority: "HIGH"
```

**Feature summary presented to user:**

```
I've identified 2 potential features from Book Collector:

| Feature | Description | Priority |
|---------|-------------|----------|
| Series Tracker | See series order, spot missing volumes | HIGH |
| Semantic Search | Find books by plot description | HIGH |
```

## Notes

- **Purpose is feature discovery** — Stories help users identify what they want to build, not just document requirements
- **Strict format required** — "As {Name}, I want to {action}, So that {benefit}." (single line, no "a" before name)
- **Feature names matter** — Each story should suggest a short feature name for the summary table
- **User ideas are gold** — If a story sparks a new idea, capture it — that's the real value
- **Primary personas get more stories** — 4-5 stories vs 2-3 for secondary personas
- **Multiple personas wanting same feature** = higher priority hint
- **Technical level affects criteria detail** — LOW = simple visual criteria, HIGH = detailed expectations