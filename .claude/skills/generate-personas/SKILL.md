---
name: generate-personas
description: Generate user personas from a project description. Use when you have a project vision and need to identify who the users are, or when asked to "create personas", "define user types", "who are my users", or "identify target users". Outputs structured personas for downstream use by generate-user-stories skill.
---

# Generate User Personas

Take the project description and generate 10-15 distinct user personas who would use this system.

## Process

### Step 1: Analyze the Project Description

Read the project description from `$ARGUMENTS`. This may be:
- A file path (e.g., `memory/project-brief.yaml` containing a `raw_vision` field)
- A direct text description passed as arguments
- Content from conversation context if no arguments provided

**Error handling:**
- If input file does not exist or cannot be read, ask the user to provide a project description
- If input is empty or too vague (under 20 words), ask clarifying questions about what the system does and who it's for before proceeding

Once you have usable input, extract from the description:

1. **Core functionality** - What does the system do?
2. **Target domain** - What context/industry is this for?
3. **Problem space** - What pain points does it address?
4. **User types implied** - Who is mentioned or implied as using this?
5. **Edge scenarios** - What unusual use cases might exist?

### Step 2: Generate Personas

Create 10-15 distinct personas. Each persona represents a different type of user with genuinely different needs, goals, and pain points.

**Output format:**

```yaml
personas:
  - id: "P-001"
    name: "[Memorable descriptive name, e.g., 'Digital Hoarder']"
    role: "[Their role/title, e.g., 'Home User with Disorganized Files']"
    description: |
      [2-3 sentences about who this person is and their context.
      Make them feel real - give them a situation, not just a label.]
    goals:
      - "[What they want to achieve with this system]"
      - "[Secondary goal]"
    pain_points:
      - "[Current frustration they face that this system addresses]"
      - "[Another pain point]"
    technical_level: "LOW | MEDIUM | HIGH"
    is_primary: true | false  # Is this the main target user?
```

**Persona generation rules:**

- Include at least one **primary** user (the main target, `is_primary: true`)
- Include at least one **edge case** user (unusual but valid use case)
- Vary **technical levels** — don't make everyone technical
- Give each persona a **memorable name**, not generic labels like "User A"
- Make personas **distinct** — each should have different goals/pain points
- Use **domain language** from the project description

### Step 3: Present for Review

Display all generated personas in a clear, readable format. Then use `AskUserQuestion` to get user confirmation:

```json
AskUserQuestion({
  "questions": [
    {
      "header": "Personas",
      "question": "I've identified [N] user personas based on your description. Do these capture the right users for your system?",
      "multiSelect": false,
      "options": [
        {"label": "Yes, these look right", "description": "The personas accurately represent who will use this"},
        {"label": "Missing a user type", "description": "There's another type of user I need to add"},
        {"label": "Something is wrong", "description": "One of these personas doesn't fit my vision"},
        {"label": "Too many personas", "description": "Simplify to fewer user types"}
      ]
    }
  ]
})
```

### Step 4: Iterate Based on Feedback

If user indicates issues:
- **Missing user type**: Ask for details, add new persona
- **Something wrong**: Ask which persona, get specifics, revise or remove
- **Too many**: Identify least important, propose consolidation

**Loop escape:** Track rejection count. If user rejects personas 3+ times without reaching "Yes", ask:

```json
AskUserQuestion({
  "questions": [
    {
      "header": "Restart",
      "question": "We've tried a few iterations. Would you like to take a different approach?",
      "multiSelect": false,
      "options": [
        {"label": "Start fresh", "description": "Discard current personas and generate new ones from scratch"},
        {"label": "Describe users yourself", "description": "You provide the user types, I'll flesh them out"},
        {"label": "Continue refining", "description": "Keep working with the current personas"}
      ]
    }
  ]
})
```

Re-present after changes until user confirms.

### Step 5: Document

After user approval, save personas to `memory/product-context.yaml`:

```yaml
user_personas:
  - id: "P-001"
    name: "[Persona name]"
    role: "[Role]"
    description: |
      [Full description]
    goals:
      - "[Goal 1]"
      - "[Goal 2]"
    pain_points:
      - "[Pain point 1]"
    technical_level: "MEDIUM"
    is_primary: true
  # ... additional personas
```

Announce completion: "Personas saved to memory/product-context.yaml. Ready for /generate-user-stories."

## Example

**Input:**
> ai file organization and search app that works on my computer and network. the app will first look for all pdfs, use some python library for identifying if its an ebook, if ebook then move to ebook library, connect to google books api and download all the metadata and save to the pdf or to a database, rename the pdf and organize and categorize, then create an embedding based on the book name location and description metadata...

**Output:**

```yaml
personas:
  - id: "P-001"
    name: "Digital Hoarder"
    role: "Home User with Disorganized Files"
    description: |
      Has 10+ years of accumulated files across multiple computers and drives.
      Knows the files exist somewhere but can't find them when needed.
      Has tried organizing manually but gave up after a few attempts.
    goals:
      - Find any document within seconds using natural language
      - Have files automatically organized without manual effort
      - Stop duplicating files across different locations
    pain_points:
      - Wastes hours searching for documents they know they have
      - Duplicate files everywhere with no way to identify them
      - No consistent naming or folder structure across drives
    technical_level: "MEDIUM"
    is_primary: true

  - id: "P-002"
    name: "Small Business Owner"
    role: "Solo Entrepreneur"
    description: |
      Runs a small business with invoices, contracts, bank statements, and
      client documents scattered across folders. Needs quick access during
      tax season and for contract renewals.
    goals:
      - Quickly find financial documents for tax season
      - Separate business from personal files automatically
      - Track contracts and their expiry dates
    pain_points:
      - Tax season is a nightmare of finding scattered documents
      - Missed contract renewals due to poor tracking
      - Can't tell which version of a document is the latest
    technical_level: "LOW"
    is_primary: false

  - id: "P-003"
    name: "Academic Researcher"
    role: "Graduate Student / Researcher"
    description: |
      Collects academic papers, research notes, and citations across multiple
      projects. Needs to organize by topic and find related papers by content.
      Often downloads the same paper twice under different filenames.
    goals:
      - Organize papers by research topic, not just filename
      - Find papers by content similarity, not exact title match
      - Track citations and references across papers
    pain_points:
      - Can't find papers read months ago
      - Duplicate papers downloaded under different names
      - No way to see which papers relate to each other
    technical_level: "HIGH"
    is_primary: false
```

## Notes

- Personas feed directly into `/generate-user-stories` — ensure IDs are correct (P-001, P-002, etc.)
- The `is_primary` flag helps prioritize downstream work
- Keep personas grounded in the actual project description — don't invent users that aren't implied
- Technical level affects how downstream agents write acceptance criteria