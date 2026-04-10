---
name: dynamic-questioning
description: "Generates dynamic MCQ options for AskUserQuestion based on PRD context. Use when asking users questions during Project OS phases, when the phase file says to 'ask' or 'validate' with user, or when you need to present choices. Reads project-brief.yaml and product-context.yaml to propose relevant options. Always includes 'Other' for free-form input."
---

# Dynamic Questioning

This skill teaches you how to generate relevant MCQ options from PRD context instead of relying on hardcoded options. Use this pattern throughout Project OS phases.

## Core Principle

**Never ask a naked question.** Always present the user with concrete options derived from what you already know about their project.

## When to Use This Pattern

- Any time you call `AskUserQuestion`
- When a phase file says to "ask the user" or "validate with user"
- When you have PRD context loaded (project-brief.yaml, product-context.yaml, user-stories.yaml)

## The Three-Step Process

### Step 1: Read Context First

Before asking any question, read the relevant memory files to understand what's already known:

```
Read memory/project-brief.yaml     # Goals, scope, constraints
Read memory/product-context.yaml   # Features, requirements, users
Read memory/user-stories.yaml      # User stories, personas
```

From this context, extract:
- Specific features mentioned (FTR-XXX)
- User types/roles mentioned
- Constraints and dependencies
- Domain-specific terminology

### Step 2: Derive Options from Context

Based on what you found, generate 2-4 relevant MCQ options:

| Context Source | What to Derive |
|----------------|----------------|
| `features` list | Component options, entity options, integration options |
| `requirements.functional` | Feature validation options |
| `users` section | Actor/role options |
| `constraints` | Deployment options, infrastructure options |
| `dependencies` | Integration point options |

**Option structure:**
```
{
  "label": "Clear, short label",
  "description": "What this means in plain language"
}
```

### Step 3: Always Include "Other"

Every AskUserQuestion must include an "Other" option as the escape hatch:

```
{"label": "Other", "description": "I have something different in mind"}
```

This ensures the user is never trapped by your options.

## Question Types by Section

### Actors & Roles
**Read from:** `product-context.yaml` → `users` section

**Derive options from:**
- User types mentioned (e.g., "admin", "customer", "viewer")
- Stakeholder roles from Phase 01

**Example:**
```json
AskUserQuestion({
  "questions": [
    {
      "header": "User roles",
      "question": "Based on the PRD, I see [X] and [Y] mentioned. Who else interacts with the system?",
      "multiSelect": true,
      "options": [
        {"label": "[Role from PRD]", "description": "[Context from PRD]"},
        {"label": "[Role from PRD]", "description": "[Context from PRD]"},
        {"label": "Admin/manager", "description": "Internal team managing the system"},
        {"label": "Other", "description": "I have other user types to add"}
      ]
    }
  ]
})
```

### Features & Components
**Read from:** `product-context.yaml` → `features` and `requirements.functional`

**Derive options from:**
- Feature names (FTR-XXX)
- Feature descriptions
- Priority (P0/P1/P2)

**Example:**
```json
AskUserQuestion({
  "questions": [
    {
      "header": "Feature scope",
      "question": "I've identified these core features from the PRD: [FTR-001], [FTR-002], [FTR-003]. Which are essential for the first release?",
      "multiSelect": true,
      "options": [
        {"label": "[FTR-001 name] (Recommended)", "description": "[Why it's essential based on PRD context]"},
        {"label": "[FTR-002 name]", "description": "[Context]"},
        {"label": "All three for MVP", "description": "Full feature set from day one"},
        {"label": "Other", "description": "I want to adjust the priorities"}
      ]
    }
  ]
})
```

### Integrations
**Read from:** `project-brief.yaml` → `dependencies` and `product-context.yaml` → `requirements`

**Derive options from:**
- External dependencies (DEP-XXX)
- Third-party services mentioned
- Data sources/destinations

**Example:**
```json
AskUserQuestion({
  "questions": [
    {
      "header": "Integrations",
      "question": "The PRD mentions [DEP-001] for [purpose]. Are there other systems this needs to connect to?",
      "multiSelect": true,
      "options": [
        {"label": "[DEP-001 name]", "description": "[Already in PRD]"},
        {"label": "Payment processor", "description": "Handle transactions"},
        {"label": "Email service", "description": "Send notifications"},
        {"label": "None additional", "description": "The PRD covers all integrations"},
        {"label": "Other", "description": "I have other integrations to add"}
      ]
    }
  ]
})
```

### Data Entities
**Read from:** `product-context.yaml` → `features` and `user-stories.yaml`

**Derive options from:**
- Core nouns in feature descriptions (User, Order, Document, etc.)
- Entities referenced in user stories

**Example:**
```json
AskUserQuestion({
  "questions": [
    {
      "header": "Core entities",
      "question": "From the user stories, I see the system revolves around [Entity A], [Entity B], and [Entity C]. Does that match your mental model?",
      "multiSelect": false,
      "options": [
        {"label": "Yes, that's right", "description": "Those are the main things"},
        {"label": "Missing something", "description": "There's another key entity"},
        {"label": "Something is wrong", "description": "One of those isn't central"},
        {"label": "Other", "description": "Let me clarify"}
      ]
    }
  ]
})
```

## Validation Questions

After presenting options, verify:

1. **Did I read context first?** — Options should reference actual PRD content
2. **Are options specific?** — Not "Option A" but actual feature/role names
3. **Is "Other" included?** — User must have escape hatch
4. **Is there a recommendation?** — Add "(Recommended)" to your suggested option when appropriate

## Anti-Patterns

❌ **Don't use placeholder text:**
```
"question": "What are your core entities: [Entity A], [Entity B], [Entity C]?"
```
If you don't know the entities, read the PRD first.

❌ **Don't offer options without context:**
```
{"label": "Option 1", "description": "Description"}
```
Options should be meaningful, not generic.

❌ **Don't skip the "Other" option:**
Users need to be able to provide input you didn't anticipate.

## Reference

For option templates by question type, see `references/option-templates.md`.