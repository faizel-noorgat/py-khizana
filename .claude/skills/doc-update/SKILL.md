---
name: doc-update
description: >
  Incrementally update Memory Bank files with findings from conversation. Handles
  three scenarios: create new file, add new section, or append to existing section.
  Use when user says "document this", "update the memory bank", "save findings",
  "write to project-brief", or when completing a phase activity that produces
  discoverable content. Also use when context compaction is imminent and findings
  need checkpointing.
---

# Document Update

Update Memory Bank files incrementally. This skill prevents context loss during
long discovery sessions by checkpointing findings as they emerge.

## When to Use

- **End of phase activity** - After completing any discovery/planning conversation
- **Before context compaction** - When findings need preservation
- **User request** - When user says "document this" or "save to memory bank"
- **Agent workflow** - Phase agents call this to checkpoint incremental progress

## Process

### 1. Identify Target

Parse the request to determine:
- **file_path** - Which Memory Bank file (e.g., `memory/project-brief.md`)
- **section_name** - Which section to update (e.g., `Goals`, `Constraints`)
- **content** - Extract from conversation context (not passed as argument)

### 2. Detect Scenario

Read the target file and determine which scenario applies:

| Condition | Scenario | Tool |
|-----------|----------|------|
| File doesn't exist | Create | `Write` |
| File exists, section missing | Add | `Edit` |
| File exists, section present | Append/Replace | `Edit` |

**Why this matters:** Each scenario requires different tool usage. Create builds
structure from scratch. Add extends existing structure. Append preserves existing
content while adding new findings.

### 3. Extract Content

Content comes from conversation context - review what was discussed:

- Identify the key findings for this topic
- Structure appropriately: lists for Goals/Constraints, prose for Overview
- Use IDs where expected (G-001, C-001, etc.)

### 4. Apply Tool

#### Scenario 1: File Doesn't Exist

Use `Write` to create:

```
Write(
  file_path: "memory/project-brief.md",
  content: """# Project Brief

## Overview
[Content from conversation]

## Goals
- id: "G-001"
  description: "[First goal from conversation]"

---
*Last updated: [date]*
"""
)
```

#### Scenario 2: File Exists, Section Missing

Use `Edit` to add section after the last existing section:

```
Edit(
  file_path: "memory/project-brief.md",
  old_string: "---\n*Last updated: ...*",
  new_string: """## Constraints
- id: "C-001"
  description: "[New constraint from conversation]"

---
*Last updated: [new date]*
"""
)
```

#### Scenario 3: Section Exists

**If placeholder content:** Replace entirely
**If real content:** Append new items while preserving existing

```
Edit(
  file_path: "memory/project-brief.md",
  old_string: """## Goals
- id: "G-001"
  description: "Existing goal"""",
  new_string: """## Goals
- id: "G-001"
  description: "Existing goal"
- id: "G-002"
  description: "[New goal from conversation]""""
)
```

### 5. Confirm

Report what was updated:
- File path
- Section name
- Brief summary of content added

## Section Formats

Common Memory Bank sections follow predictable formats. Match the expected structure:

| Section | Format |
|---------|--------|
| Overview | 1-2 sentence prose description |
| Goals | `- id: "G-XXX" description: "..."` |
| Constraints | `- id: "C-XXX" description: "..."` |
| Success Criteria | `- id: "S-XXX" description: "..."` |
| Scope | Two lists: In-scope, Out-of-scope |
| Target Users | Bullet list with persona details |
| Features | P0/P1/P2 priority tiers |
| Risks | Table: Risk, Likelihood, Impact, Mitigation |

## Error Handling

| Error | Resolution |
|-------|------------|
| `old_string` not unique | Include more context lines for unique match |
| Section spans many lines | Match the section header and first item only |
| File permission denied | Report error, ask user for intervention |
| Content exceeds reasonable length | Summarize, note "see conversation for full details" |

## Example

**User says:** "We discussed the core goals - save those to project-brief"

**Agent actions:**
1. Parse: file_path = `memory/project-brief.md`, section = `Goals`
2. Read file: exists with Overview section, Goals section missing
3. Extract content: 3 goals from conversation with IDs G-001, G-002, G-003
4. Edit: Add Goals section after Overview
5. Confirm: "Updated project-brief.md → Goals with 3 items"

## Why This Matters

Memory Bank files persist across sessions. Incremental updates mean:
- Findings are captured before context compaction erases them
- Each phase builds on documented discoveries
- Handoffs to future sessions have complete context
- No need to re-derive what was already discovered