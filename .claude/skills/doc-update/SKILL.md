---
name: doc-update
description: >
  Incrementally update Memory Bank YAML files with findings from conversation.
  Spawns an isolated sub-agent to prevent context pollution. Use when user says
  "document this", "update the memory bank", "save findings", "write to
  project-brief", or when completing a phase activity. Also use before context
  compaction to checkpoint findings. Main context receives only brief summaries.
---

# Document Update

Update Memory Bank YAML files incrementally by spawning an isolated sub-agent.
This prevents context pollution from reading entire files and rules.

## Architecture

**Why spawn a sub-agent:**
- Memory Bank files are updated 10+ times per phase
- Each update would read the same 250+ line rules file
- Each update would read the same 100+ line YAML file
- This accumulates in main context, wasting tokens

**Solution:**
- Spawn a sub-agent with its own isolated context window
- Sub-agent reads rules + document once, applies edit, returns brief summary
- Main context never sees the full rules file or document content

## Invocation Pattern

When this skill is triggered, spawn a sub-agent using the Agent tool:

```
Agent(
  subagent_type: "general-purpose",
  description: "Update memory bank file",
  prompt: """
  You are a Document Update Agent. Update a Memory Bank YAML file.

  ## Target
  File: {file_path}
  Section: {section_name}
  Content to add: {content_summary}

  ## Token-Saving Workflow (REQUIRED)

  DO NOT read entire files. Follow this sequence:

  ### Step 1: Grep Headers
  Use Grep to find YAML headers and their line positions:
  - pattern: "^[a-zA-Z_]+:"
  - output_mode: "content"
  - path: {file_path}

  This returns a table like:
  | Line | Header |
  |------|--------|
  | 1    | file_id |
  | 22   | features |
  | 35   | requirements |

  ### Step 2: Calculate Line Range
  From the grep output, identify the target section's start line.
  The next header's line minus 1 is the section's end line.

  Example: If target is "features" at line 22, and next header is
  "requirements" at line 35, read lines 22-34 (offset 22, limit 13).

  ### Step 3: Read Only Needed Lines
  Use Read with offset/limit, NOT the entire file:
  - offset: {section_start_line}
  - limit: {section_end_line - section_start_line + 1}

  ### Step 4: Read Relevant Rules Section
  Rules files are in `.claude/rules/`. Match the section name.
  Use the same grep + selective read pattern on the rules file.

  Rules file mapping:
  - project-brief.yaml → .claude/rules/project-brief.md
  - product-context.yaml → .claude/rules/product-context.md

  ### Step 5: Apply Edit
  Use Edit tool to update the section. Preserve existing content,
  append new items, or replace placeholder content.

  ### Step 6: Return Summary
  Return ONLY a brief summary (max 50 words):
  - File updated
  - Section updated
  - Items added/replaced
  - No full document content

  ## Example Output
  "Updated memory/product-context.yaml → features section with 3 new items (FTR-002, FTR-003, FTR-004)"

  ## Error Handling
  If section doesn't exist: Add it after the last section.
  If old_string not unique: Include more context lines.
  If file doesn't exist: Use Write tool to create it.
  """
)
```

## Arguments Passed to Skill

The skill receives two arguments:
1. **file_path** - Memory Bank file (e.g., `memory/project-brief.yaml`)
2. **section_name** - Section to update (e.g., `Goals`, `Constraints`)

The content is extracted from conversation context - review what was discussed
with the user before the skill was invoked.

## Extracting Content

Before spawning the sub-agent:
1. Review the conversation context
2. Identify key findings for the target section
3. Structure appropriately:
   - Lists with IDs for Goals/Constraints/Features (G-001, C-001, FTR-001)
   - Prose for Overview
   - Two-part structure for Scope (in_scope, out_of_scope)
4. Pass as `{content_summary}` in the prompt

## Section Formats

| Section | Document | Format |
|---------|----------|--------|
| raw_vision | project-brief.yaml | Prose (verbatim from user) |
| overview | project-brief.yaml | 1-2 sentence prose |
| current_landscape | project-brief.yaml | Prose describing what exists today |
| goals | project-brief.yaml | List with IDs: G-001, G-002... |
| in_scope | project-brief.yaml | List under scope: key |
| out_of_scope | project-brief.yaml | List under scope: key |
| success_criteria | project-brief.yaml | List with IDs: SC-001... |
| constraints | project-brief.yaml | List with IDs: C-001... |
| dependencies | project-brief.yaml | List with IDs: DEP-001... |
| stakeholders | product-context.yaml | List with roles and goals |
| target_users | product-context.yaml | List with persona details |
| user_needs | product-context.yaml | List with UN-XXX IDs |
| features | product-context.yaml | List with FTR-XXX IDs |
| features_discovered | product-context.yaml | Archive of discovery session with discovery_status |
| functional | product-context.yaml | List with FR-XXX IDs under requirements: key |
| non_functional | product-context.yaml | List with NFR-XXX IDs under requirements: key |
| risks | product-context.yaml | List with R-XXX IDs |
| assumptions | product-context.yaml | List with A-XXX IDs |
| open_questions | product-context.yaml | List with OQ-XXX IDs |
| glossary | product-context.yaml | Key-value pairs |
| user_personas | product-context.yaml | Structured persona objects |
| user_stories | product-context.yaml | List with US-XXX IDs |

## Nested Field Handling

Some sections are nested under parent keys in YAML:

| Section | Parent Key | Document |
|---------|------------|----------|
| in_scope | scope: | project-brief.yaml |
| out_of_scope | scope: | project-brief.yaml |
| functional | requirements: | product-context.yaml |
| non_functional | requirements: | product-context.yaml |

**Workflow for nested fields:**

1. **Grep for parent key first** — Use pattern `^scope:` or `^requirements:` to find the parent section start line
2. **Read full parent section** — From parent line to next top-level key (determined by next grep result)
3. **Edit within nested content** — The old_string should include enough context to uniquely identify the nested key (e.g., include `scope:` and `in_scope:` when editing in_scope)

**Example for updating in_scope:**
```
old_string: "scope:\n  in_scope:\n    - \"\""
new_string: "scope:\n  in_scope:\n    - \"Personal book library tracking\"\n    - \"Reading history and progress\""
```

**Rules file handling:**

Rules files are markdown (.md), not YAML. Use a different grep pattern:
- For top-level sections: `^## [a-zA-Z_]+` (matches `## scope`, `## goals`)
- For nested subsections: `^### [a-zA-Z_]+` (matches `### in_scope`, `### non_functional`)

When updating a nested field like `in_scope`, grep the rules file for `### in_scope` to find the subsection directly.

## Rules File Mapping

| Document | Rules File | Sections |
|----------|------------|----------|
| project-brief.yaml | .claude/rules/project-brief.md | raw_vision, overview, current_landscape, goals, in_scope, out_of_scope, success_criteria, constraints, dependencies |
| product-context.yaml | .claude/rules/product-context.md | stakeholders, target_users, user_needs, features, features_discovered, functional, non_functional, risks, assumptions, open_questions, glossary, user_personas, user_stories |

## Example

**Phase agent completes a discovery conversation:**

```
Skill(skill: "doc-update", args: "memory/project-brief.yaml raw_vision")
```

**Skill spawns sub-agent:**
- Grep finds headers in project-brief.yaml
- Calculates line range for raw_vision (e.g., lines 6-15)
- Reads lines 6-15 only (not entire 50-line file)
- Reads rules section for raw_vision only (not entire 246-line rules file)
- Applies edit
- Returns: "Updated memory/project-brief.yaml → raw_vision with user's project description"

**Main context receives:**
Only the brief summary. Rules file and document content stay in sub-agent's isolated context.

## Token Savings

| Scenario | Old Approach | New Approach | Savings |
|----------|--------------|--------------|---------|
| 10 updates per phase | 10 × (250 rules + 100 doc) = 3500 lines | 10 × (20 section + 50 rules section) = 700 lines | ~80% |
| Single update | Full rules file in main context | Rules in sub-agent only | 100% of rules tokens |

## Why This Matters

Memory Bank files persist across sessions. This architecture:
- Captures findings without polluting main context
- Enables 10+ updates per phase without token exhaustion
- Keeps main conversation focused on user dialogue
- Sub-agents handle document I/O in isolation