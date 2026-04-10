# Option Templates by Question Type

This reference provides templates for generating MCQ options based on question type. Use these as starting points, then customize based on PRD context.

## Actor/Role Questions

**Read from:** `users` section, stakeholder info

### Base options (always include):
```json
{"label": "End user/customer", "description": "The primary person the product is built for"},
{"label": "Admin/operator", "description": "Internal team managing the system or users"},
{"label": "Other", "description": "I have other user types to add"}
```

### PRD-derived additions:
- If PRD mentions "vendors" → add `{"label": "Vendor/supplier", "description": "External partners who provide goods/services"}`
- If PRD mentions "reviewers" → add `{"label": "Reviewer/approver", "description": "People who approve or reject submissions"}`
- If PRD mentions "guests" → add `{"label": "Guest/anonymous", "description": "Unauthenticated visitors with limited access"}`

---

## Feature Validation Questions

**Read from:** `features` list, `requirements.functional`

### Structure:
```json
{"label": "[FTR-XXX name] (Recommended)", "description": "[Why essential based on PRD priority]"},
{"label": "[FTR-XXX name]", "description": "[Context from PRD]"},
{"label": "Defer to later phase", "description": "Not critical for first release"},
{"label": "Other", "description": "I want to adjust priorities"}
```

### Example for a book tracking app:
```json
{"label": "Book search (Recommended)", "description": "Core feature - find books by title/author"},
{"label": "Reading progress", "description": "Track page numbers and completion"},
{"label": "Bookmarks/notes", "description": "Save passages and annotations"},
{"label": "Other", "description": "I have other features to discuss"}
```

---

## Integration Questions

**Read from:** `dependencies`, `requirements`

### Base options:
```json
{"label": "Payments", "description": "Process transactions"},
{"label": "Email/SMS", "description": "Send notifications"},
{"label": "Authentication", "description": "Social login or SSO"},
{"label": "File storage", "description": "Store uploads and media"},
{"label": "None additional", "description": "Current list is complete"},
{"label": "Other", "description": "I have other integrations"}
```

### PRD-derived additions:
- If PRD mentions "Google Books API" → add as specific option
- If PRD mentions "Stripe" → add `{"label": "Payment processor (Stripe)", "description": "Already mentioned in PRD"}`
- If PRD mentions "AWS S3" → add `{"label": "Cloud storage", "description": "Store files and images"}`

---

## Data Entity Questions

**Read from:** `features`, `user-stories.yaml`

### Structure:
```json
{"label": "Yes, that's correct", "description": "Those are the main things the system tracks"},
{"label": "Close, but missing something", "description": "There's another key entity"},
{"label": "Something is wrong", "description": "One of those isn't central"},
{"label": "Other", "description": "Let me clarify the data model"}
```

### How to derive entities:
1. Look for nouns in feature descriptions
2. Check user stories for "As a [user], I want to [verb] [ENTITY]"
3. Identify what data persists between sessions

### Example entities from a library app:
- User → Profile, preferences
- Book → Title, author, ISBN
- Reading → Progress, bookmarks
- List → Collection, shelf

---

## Scope/Out-of-Scope Questions

**Read from:** `scope` section, feature priorities

### Base options:
```json
{"label": "Mobile app", "description": "Native iOS/Android"},
{"label": "Offline mode", "description": "Works without internet"},
{"label": "Advanced reporting", "description": "Detailed analytics"},
{"label": "Multi-language", "description": "UI translations"},
{"label": "Nothing to exclude", "description": "Everything in PRD is in scope"},
{"label": "Other", "description": "Something else should be out of scope"}
```

### PRD-derived additions:
- If PRD mentions "MVP" or "phase 1" → emphasize minimal scope
- If PRD mentions "future phases" → add those as out-of-scope options

---

## Quality Attribute Questions

**Read from:** `constraints`, `requirements.non_functional`

### Performance options:
```json
{"label": "Instant (< 1 second)", "description": "Every action feels immediate"},
{"label": "Fast (2-3 seconds)", "description": "Most actions quick, some slower"},
{"label": "Moderate", "description": "Delays acceptable for complex tasks"},
{"label": "Not a priority", "description": "Correctness > speed"},
{"label": "Other", "description": "I have specific requirements"}
```

### Data sensitivity options:
```json
{"label": "Personal data (PII)", "description": "Names, emails, addresses"},
{"label": "Financial data", "description": "Payments, transactions"},
{"label": "Health data", "description": "Medical information (HIPAA)"},
{"label": "Business confidential", "description": "Internal documents"},
{"label": "No sensitive data", "description": "Public or non-personal"},
{"label": "Other", "description": "I have other data types"}
```

### PRD-derived additions:
- If PRD mentions compliance (GDPR, HIPAA) → add specific compliance option
- If PRD mentions "audit trail" → add security/audit option

---

## Deployment Questions

**Read from:** `constraints`, `dependencies`

### Deployment shape:
```json
{"label": "Web browser only", "description": "URL access, no install"},
{"label": "Web + mobile app", "description": "Browser and native apps"},
{"label": "Desktop app", "description": "Download and install"},
{"label": "Embedded", "description": "Runs inside another platform"},
{"label": "Other", "description": "Different deployment model"}
```

### Infrastructure constraints:
```json
{"label": "Cloud-hosted only", "description": "No on-premise requirement"},
{"label": "Must run on-premise", "description": "Data can't leave servers"},
{"label": "Specific region", "description": "Data sovereignty requirements"},
{"label": "Offline capable", "description": "Works without internet"},
{"label": "No constraints", "description": "Free to choose"},
{"label": "Other", "description": "I have specific constraints"}
```

---

## Business Rules Questions

**Read from:** `features`, `user-stories.yaml`

### Structure:
```json
{"label": "[Rule category from PRD]", "description": "[What it governs]"},
{"label": "[Rule category from PRD]", "description": "[What it governs]"},
{"label": "No complex rules", "description": "Simple logic, no special handling"},
{"label": "Other", "description": "I have rules not listed"}
```

### Common rule categories:
| Category | Example Rules |
|----------|---------------|
| Pricing | Discounts, thresholds, tiers |
| Access | Permissions, role-based visibility |
| Workflow | Approval chains, state transitions |
| Validation | Input constraints, format rules |
| Automation | Triggers, scheduled actions |

### PRD-derived additions:
- If PRD mentions "free shipping over $100" → Pricing rule
- If PRD mentions "managers approve expenses" → Workflow rule
- If PRD mentions "password must be 8+ characters" → Validation rule

---

## Validation Checklist

After generating options, verify:

- [ ] Options reference actual PRD content (feature names, not "Feature 1")
- [ ] At least 2 concrete options provided
- [ ] "Other" option included
- [ ] Recommended option marked if applicable
- [ ] Descriptions explain what each option means in plain language
- [ ] Options are mutually exclusive (for single-select) or independent (for multi-select)