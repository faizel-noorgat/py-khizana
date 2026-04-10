# Dynamic Question Generation Guide

This guide explains when to derive MCQ options from PRD context versus using predefined options. Add this section to Phase files where questioning is a core activity.

---

## Question Derivation Strategy

Not all questions are the same. Use this classification to decide how to generate options:

### Category 1: Standard Domain (Keep Base Options)

**What it means:** Universal concerns that apply to most projects.

**Approach:** Use predefined base options, but add PRD-derived options if the project has specific needs.

**Examples:**
| Question Topic | Base Options |
|----------------|--------------|
| Performance | Instant, Fast, Moderate, Not a priority |
| Uptime | Critical, Significant, Minor, Flexible |
| Data sensitivity | PII, Financial, Health, Business, None |
| Deployment | Web, Mobile, Desktop, Embedded |

**When to add PRD-derived options:**
- PRD mentions specific compliance requirements (GDPR, HIPAA) → add compliance option
- PRD mentions specific performance targets → add as concrete option

---

### Category 2: Project-Specific (PRD-Derived Options)

**What it means:** Questions where options depend entirely on what the PRD contains.

**Approach:** Read PRD first, derive options from actual content. Never use placeholder text.

**Examples:**
| Question Topic | Source in PRD |
|----------------|---------------|
| Core entities | Features, User stories → extract nouns |
| Features to prioritize | features list, requirements.functional |
| User roles | users section, stakeholders |
| Integrations | dependencies, requirements |

**Process:**
1. Read `memory/product-context.yaml` → find relevant section
2. Extract specific items (entity names, feature names, role names)
3. Generate 2-4 options using actual names, not placeholders
4. Add "Other" escape hatch

**Example:**
```
WRONG: "What are your core entities: [Entity A], [Entity B]?"
RIGHT: "From the user stories, I see Book, Reading Progress, and List. Are these the core entities?"
```

---

### Category 3: Hybrid (Base + PRD-Derived)

**What it means:** Questions with some universal options plus project-specific additions.

**Approach:** Start with base options, add PRD-derived options for items the user mentioned.

**Examples:**
| Question Topic | Base Options | PRD-Derived Additions |
|----------------|--------------|----------------------|
| Integrations | Payments, Email, Auth, Storage | Specific APIs mentioned in PRD |
| Out of scope | Mobile app, Offline, Reporting | Features PRD deferred to future |
| Actors | End user, Admin, Manager | Specific roles from PRD |

**Example:**
```json
{
  "header": "Integrations",
  "options": [
    {"label": "Google Books API", "description": "Already mentioned in PRD for metadata"},
    {"label": "Payments", "description": "If you plan to monetize"},
    {"label": "Email", "description": "For notifications"},
    {"label": "None additional", "description": "The PRD covers integrations"},
    {"label": "Other", "description": "I have other integrations"}
  ]
}
```

---

### Category 4: Exploratory (Discovery-Based)

**What it means:** Questions where you're probing for unknowns.

**Approach:** Offer probe options that help surface issues, plus "Other" for unexpected answers.

**Examples:**
| Question Topic | Probe Options |
|----------------|---------------|
| Unproven integrations | Yes - new to us, No - all familiar, Not sure |
| Scale uncertainty | Very confident, Reasonable estimate, Uncertain |
| Risk areas | Complexity, Coupling, Unknowns, Constraints |

**Process:**
1. Offer structured probes based on common risk areas
2. Let "Other" capture project-specific risks
3. Follow up on "Other" responses with clarifying questions

---

## Quick Reference Table

When generating options, check the question type:

| If asking about... | Category | Approach |
|-------------------|----------|----------|
| Performance, uptime, security | Standard Domain | Use base options + PRD specifics |
| Core entities, features, user roles | Project-Specific | Derive entirely from PRD |
| Integrations, scope boundaries | Hybrid | Base options + PRD mentions |
| Risks, unknowns, gaps | Exploratory | Probe options + Other |

---

## Validation Checklist

Before calling AskUserQuestion:

- [ ] Did I read the PRD context first?
- [ ] Are options specific (actual names, not placeholders)?
- [ ] Is "Other" included?
- [ ] Is there a recommendation marked?
- [ ] Does the question reference PRD content (e.g., "From the user stories..."?

---

## Reference

For detailed templates by question type, invoke the `/dynamic-questioning` skill or read `.claude/skills/dynamic-questioning/references/option-templates.md`.