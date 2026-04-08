---
project_os_version: "1.0"
memory_bank: "memory/"
load_order:
  - memory/current-state.md
  - memory/project-brief.yaml
  - memory/product-context.yaml
  - memory/tech-context.yaml
  - memory/system-patterns.yaml
  - memory/progress.md
  - memory/NOTES_NEXT_SESSION.yaml
---

# Project OS

## Session Start

Read the memory bank files in the order defined above. Do not begin any work until all files have been read.

## Active Phase

After reading the memory bank, check `active_phase` in `progress.yaml` and load the corresponding phase file from `phases/`.

| active_phase | Load |
|---|---|
| 01-PRD_DISCOVERY | phases/01-prd-discovery.md |
| 02-HIGH_LEVEL_SPEC | phases/02-high-level-spec.md |
| 03-TECH_STACK_SELECTION | phases/03-tech-stack-selection.md |
| 04-BACKEND_LLD | phases/04-backend-lld.md |
| 05-UI_UX_LLD | phases/05-ui-ux-lld.md |
| 06-IMPLEMENTATION_PLANNING | phases/06-implementation-planning.md |
| 07-IMPLEMENTATION | phases/07-implementation.md |
| 08-VERIFICATION_UAT | phases/08-verification-uat.md |
| 09-RELEASE | phases/09-release.md |

The active phase file defines how to behave, what to ask, what to document, and what the exit criteria are for that phase.