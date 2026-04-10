---
paths: "memory/system-patterns.yaml"
description: "Completion rules for Phase 02 HLD memory file"
priority: 50
enabled: true
---

# system-patterns.yaml Completion Rules

This file defines what "complete" means for each section of `memory/system-patterns.yaml`. Use these rules to validate content before marking Phase 02 sections as complete.

## File Purpose

Phase 02 (High-Level Specification) produces this file. It captures the technology-agnostic system architecture: actors, components, boundaries, quality attributes, data flows, failure modes, deployment topology, and architectural decisions.

---

## actors

### What goes here
All human users, system actors, and external services that interact with the system.

### Completion rules
- **Minimum**: At least one actor defined
- **Each actor requires**:
  - `id`: Format `ACT-XXX` (sequential, e.g., ACT-001, ACT-002)
  - `name`: Human-readable name (e.g., "End User", "Admin", "Scheduler")
  - `type`: One of `HUMAN` | `SYSTEM` | `EXTERNAL`
  - `primary_goal`: One sentence describing what this actor wants from the system
  - `key_interactions`: List of main interaction types (can be empty initially, filled as components are defined)

### Validation questions
- Can you name every person or thing that triggers activity in the system?
- Are there scheduled tasks or background workers that act without human input?
- Are there external systems that call into yours (webhooks, APIs)?

### Common gaps
- Missing background/system actors (scheduled jobs, workers)
- Missing external actors (partner APIs, third-party services)
- Vague goals (e.g., "uses the system" — needs specificity like "tracks their reading progress")

---

## architecture.components

### What goes here
Logical system components — the big pieces that make up the system.

### Completion rules
- **Minimum**: At least 2 components defined (even a simple system has a frontend + backend concept)
- **Each component requires**:
  - `id`: Format `CMP-XXX` (sequential)
  - `name`: Domain-language name (e.g., "User Interface", "Order Processor", not "React App")
  - `responsibility`: Single sentence — ONE thing it does
  - `actors`: Which actors interact with this component (reference ACT-XXX IDs)
  - `inputs`: At least one defined (source + description)
  - `outputs`: At least one defined (destination + description)
  - `dependencies`: List of other components or external systems it depends on

### Validation questions
- Does every component have at least one input AND one output?
- If a component has no consumer, does it belong?
- Can you explain what each component does in ONE sentence?
- Are dependencies bidirectional? (If A depends on B, does B know about A?)

### Common gaps
- Missing inputs/outputs — component exists but flows aren't traced
- Overly broad responsibility ("handles everything") — should be split
- Missing actor mapping — who uses this component?

---

## system_boundaries

### What goes here
Explicit scope definition: what's in the system and what's explicitly out.

### Completion rules
- `in_scope`: List of capabilities the system delivers
- `out_of_scope`: Each item has:
  - `capability`: What's excluded
  - `reason`: One of `FUTURE_PHASE` | `SEPARATE_SYSTEM` | `NOT_NEEDED`

### Validation questions
- Would a user mistakenly expect something that's listed as out-of-scope?
- Are out-of-scope items temporary (future phase) or permanent (separate system)?

### Common gaps
- Empty out-of-scope — implies everything is in, which is rarely true
- Vague reason codes — need to know whether to revisit later

---

## integration_points

### What goes here
External system connections — inbound (others call us) and outbound (we call them).

### Completion rules
- **outbound**: Systems we call
  - `id`: Format `INT-XXX`
  - `external_system`: Name of the external service
  - `purpose`: Why we connect to it
  - `direction`: `REQUEST_RESPONSE` | `FIRE_AND_FORGET` | `STREAM`
  - `pattern`: `SYNC` | `ASYNC`
  - `data_format`: `STRUCTURED` | `UNSTRUCTURED` | `BINARY`
- **inbound**: Systems that call us
  - `id`: Format `INT-XXX`
  - `consumer`: Who calls us
  - `purpose`: Why they call us
  - `direction`: `WEBHOOK` | `POLLING` | `SUBSCRIPTION`
  - `pattern`: `SYNC` | `ASYNC`
  - `data_format`: Same as outbound

### Validation questions
- Are all integrations from Phase 01 dependencies represented?
- Is the pattern (sync vs async) decided? If not, flag as provisional.

### Common gaps
- Missing inbound integrations (webhooks from payment providers, etc.)
- Undecided pattern — needs to be resolved before Phase 04

---

## quality_attributes

### What goes here
Non-functional requirements that shape architecture (performance, security, etc.).

### Completion rules
- **Minimum**: At least one quality attribute defined (even a trivial project has expectations)
- **Each attribute requires**:
  - `id`: Format `QA-XXX`
  - `attribute`: One of `PERFORMANCE` | `AVAILABILITY` | `SECURITY` | `SCALABILITY` | `RELIABILITY` | `MAINTAINABILITY` | `OBSERVABILITY`
  - `requirement`: Specific, measurable (e.g., "< 200ms response", not "fast")
  - `priority`: `MUST_HAVE` | `SHOULD_HAVE` | `NICE_TO_HAVE`
  - `notes`: Context for why this matters

### Validation questions
- Can a tester write a pass/fail test for this requirement?
- If you replaced vague terms with numbers, would implementation change?

### Common gaps
- Vague requirements ("fast", "secure") — use anti-pattern checklist from Phase 01
- Missing security for systems handling PII/financial data
- Missing observability for systems that need debugging in production

---

## domain_model

### What goes here
Core data entities the system revolves around.

### Completion rules
- **Minimum**: At least one core entity defined
- **Each entity requires**:
  - `id`: Format `DM-XXX`
  - `entity`: Name of the thing (e.g., "User", "Order", "Document")
  - `description`: What it represents in domain terms
  - `owned_by`: Which component manages this entity
  - `relationships`: Links to other entities (type: `HAS_MANY` | `BELONGS_TO` | `HAS_ONE` | `MANY_TO_MANY`)

### Validation questions
- Are entities derived from Phase 01 features/requirements?
- Can you trace where each entity is created, stored, and consumed?

### Common gaps
- Entities not mapped to components (who owns this data?)
- Missing relationships (entities exist but don't connect)

---

## data_flows

### What goes here
Step-by-step trace of how data moves for key use cases.

### Completion rules
- **Minimum**: At least one primary use case traced end-to-end
- **Each flow requires**:
  - `id`: Format `DF-XXX`
  - `name`: Use case name (e.g., "User places an order")
  - `steps`: Sequential list with actor, action, what, to, and optional emits

### Validation questions
- Can someone who doesn't know the system follow this flow?
- Does every step connect to the next (no gaps)?
- Are events/emissions captured where they happen?

### Common gaps
- Missing intermediate steps (actor → storage with no processing)
- Missing error paths (what happens if step 2 fails?)

---

## data_storage

### What goes here
Summary of what data is stored and its characteristics.

### Completion rules
- **Each storage item requires**:
  - `id`: Format `DS-XXX`
  - `data`: What's stored (e.g., "User profiles", "Session tokens")
  - `storage_type`: `PERSISTENT` | `EPHEMERAL` | `APPEND_ONLY` | `CACHE`
  - `stateful`: `true` | `false`
  - `sensitivity`: `PII` | `SENSITIVE` | `INTERNAL` | `PUBLIC`

### Validation questions
- Is sensitivity consistent with quality_attributes security requirements?
- Are ephemeral items (sessions, caches) distinguished from persistent?

### Common gaps
- Missing sensitivity classification for PII/financial data
- Event logs not classified as append-only

---

## failure_modes

### What goes here
What happens when things go wrong — failure scenarios and expected behavior.

### Completion rules
- **Minimum**: At least one critical failure scenario defined
- **Each failure requires**:
  - `id`: Format `FM-XXX`
  - `scenario`: What breaks (e.g., "External API down", "Storage unavailable")
  - `affected_components`: Which components are impacted
  - `expected_behavior`: `Graceful degrade` | `Queue and retry` | `Fail with error` | `Read-only mode`
  - `severity`: `LOW` | `MEDIUM` | `HIGH` | `CRITICAL`

### Validation questions
- For each integration point, what happens if it goes down?
- What happens if storage fails?
- Is there a critical path that has no failure handling?

### Common gaps
- Missing integration failure scenarios (payments, email, etc.)
- Missing data loss tolerance (how much can we lose?)

---

## deployment_topology

### What goes here
Logical deployment units — how the system is structured for running.

### Completion rules
- `units`: Each deployment unit:
  - `id`: Format `DU-XXX`
  - `name`: Logical name (e.g., "Frontend", "Backend", "Worker")
  - `components`: Which components run in this unit
  - `runtime_context`: Where it runs (e.g., "User's browser", "Cloud server")
- `environment_constraints`: List of hard constraints (e.g., "Must run on-premise", "Specific region required")

### Validation questions
- Does deployment shape match user access pattern (web, mobile, etc.)?
- Are environment constraints from Phase 01 represented?

### Common gaps
- Missing constraints when user mentioned them in Discovery
- Components not mapped to deployment units

---

## architectural_decisions

### What goes here
Key architecture decisions with rationale — ADR format.

### Completion rules
- **Minimum**: At least one decision recorded (the main structural choice)
- **Each decision requires**:
  - `id`: Format `ADR-XXX`
  - `title`: What decision (e.g., "Single system vs separate services")
  - `status`: `ACCEPTED` | `PROVISIONAL` | `SUPERSEDED`
  - `superseded_by`: Reference to new ADR if superseded (blank if accepted)
  - `context`: What problem prompted this decision
  - `decision`: What we chose
  - `rationale`: Why this over alternatives
  - `alternatives_considered`: At least one alternative with rejection reason
  - `consequences`: positive, negative, risks lists

### Validation questions
- Is the main structural decision recorded (monolith vs distributed, etc.)?
- Are provisional decisions flagged for revisit?

### Common gaps
- Missing alternatives — should show we considered other options
- Missing consequences — what does this choice make easier/harder?

---

## architectural_risks

### What goes here
Risks that could threaten delivery or quality, with mitigation strategies.

### Completion rules
- **Each risk requires**:
  - `id`: Format `AR-XXX`
  - `description`: What could go wrong
  - `likelihood`: `LOW` | `MEDIUM` | `HIGH`
  - `impact`: `LOW` | `MEDIUM` | `HIGH`
  - `mitigation`: How we address it (can be "Investigate in Phase 03")

### Validation questions
- Are unproven integrations flagged as risks?
- Is scale uncertainty captured?
- Are there complexity risks (too many components, tight coupling)?

### Common gaps
- Risks without mitigation — even "investigate later" is valid
- Missing scale/load risks for systems expecting growth

---

## business_rules

### What goes here
Logic that governs feature behavior — where rules live and how complex they are.

### Completion rules
- **Minimum**: Capture rules for each feature that implies logic
- **Each rule requires**:
  - `id`: Format `BR-XXX`
  - `name`: Rule name (e.g., "Free shipping threshold", "Order auto-cancel")
  - `feature`: Reference to feature (FTR-XXX from product-context.yaml)
  - `description`: What the rule does in plain language
  - `enforcing_component`: Which component enforces this (CMP-XXX)
  - `complexity`: `SIMPLE` | `CONFIGURABLE` | `EXTERNAL_API` | `DERIVED`
  - `inputs`: What data the rule needs
  - `outputs`: What the rule produces (decision, value, action)
  - `status`: `DEFINED` | `NEEDS_DETAIL` | `FUTURE_PHASE`

### Validation questions
- Are business rules derived from user stories and features?
- Is complexity level appropriate? (SIMPLE = hardcoded, CONFIGURABLE = admin can change, EXTERNAL_API = third party decides)
- Is the enforcing component identified?

### Common gaps
- Rules not mapped to components — who enforces this?
- Missing complexity classification — affects Phase 04 implementation
- Rules from user stories not captured

---

## Notes Parking Lot

During Phase 02 sections, if the user provides information that doesn't fit the current section's structure, it can temporarily go in a `notes` field. **Before gate approval**, the agent must process `notes` and move each item to its correct section.

### Mapping table
| Information type | Destination section |
|------------------|---------------------|
| User role or actor | `actors` |
| External system or service | `integration_points.outbound` or `inbound` |
| Performance expectation | `quality_attributes` (attribute: PERFORMANCE) |
| Security concern | `quality_attributes` (attribute: SECURITY) |
| Data entity | `domain_model` |
| Data flow description | `data_flows` |
| Failure scenario | `failure_modes` |
| Deployment constraint | `deployment_topology.environment_constraints` |
| Architectural preference | `architectural_decisions` |
| Something that might go wrong | `architectural_risks` |
| Business logic / rule | `business_rules` |
| Out of scope capability | `system_boundaries.out_of_scope` |

### Gate check
Before Phase 02 Review can proceed, `notes` must be empty. Any remaining items indicate incomplete processing.