# Skill: Stack Research

## Invocation
/stack-research [context-file]

## Description
Research technology stack options using Context7 (for libraries/frameworks) and WebSearch (for hosting/pricing). Returns structured findings with citations for use in tech stack selection.

## When to Use
- Phase 03 (Tech Stack Selection) Activity 1
- When needing up-to-date technology research before proposing stack options
- Any phase where current technology landscape needs investigation

## Prerequisites
- `memory/system-patterns.yaml` exists with architecture context
- Phase 02 complete (or equivalent architectural context available)

## Input
- **context-file**: Path to system patterns file (default: `memory/system-patterns.yaml`)
- Research targets extracted from context-file:
  - Deployment topology
  - Quality attributes (MUST_HAVE items)
  - Integration points
  - Data storage requirements

## Parameters

| Parameter | Required | Default | Example |
|-----------|----------|---------|---------|
| context-file | No | `memory/system-patterns.yaml` | `memory/system-patterns.yaml` |

## Process

### 1. Load Context

Read the context file to extract research parameters:

```
Read(file_path: "[context-file]")
```

Extract and note:
- `deployment_topology` — What type of frontend/backend deployment?
- `quality_attributes` — Which are marked MUST_HAVE?
- `integration_points` — External services to integrate?
- `data_storage` — Storage types and sensitivity levels?
- `constraints` — Budget, hosting preference, team expertise?

### 2. Research Frontend Options

**Use Context7** for frontend framework documentation:

```
mcp__context7__resolve-library-id(
  libraryName: "React",  // or "Vue", "Svelte", "Angular"
  query: "frontend framework for {deployment_topology} with REST API"
)

mcp__context7__query-docs(
  libraryId: "/vercel/next.js",  // resolved from above
  query: "How to build a web application that connects to a REST API backend, authentication, data fetching, performance best practices"
)
```

**Compare multiple options** by querying each framework:
- Next.js/React
- Vue/Nuxt
- Svelte/SvelteKit

### 3. Research Backend Options

**Use Context7** for backend framework documentation:

```
mcp__context7__resolve-library-id(
  libraryName: "Django",
  query: "Django REST framework with background workers, multi-tenant SaaS"
)

mcp__context7__query-docs(
  libraryId: "/websites/djangoproject_en_6_0",
  query: "Celery background workers, task queue, REST API, multi-tenant patterns, authentication with django-allauth"
)
```

**Query alternatives:**
- Django + DRF
- FastAPI
- Express/NestJS (if Node ecosystem preferred)

### 4. Research Database Options

**Use Context7** for database patterns:

```
mcp__context7__resolve-library-id(
  libraryName: "PostgreSQL",
  query: "PostgreSQL for multi-tenant SaaS application"
)

mcp__context7__query-docs(
  libraryId: "/websites/postgresql_18",
  query: "Multi-tenant patterns, row-level security, schema-per-tenant vs shared schema, performance tuning"
)
```

**Also research:**
- Redis (for Celery broker and caching)
- Managed database options

### 5. Research Hosting Platforms

**Use WebSearch** for hosting (pricing and comparisons not in docs):

```
WebSearch(query: "best hosting platforms for Django Celery 2026 pricing comparison")
WebSearch(query: "Railway vs Heroku vs Render for Python SaaS 2026")
WebSearch(query: "managed PostgreSQL Redis hosting pricing 2026")
```

Context7 is NOT suitable for hosting research because:
- Pricing changes frequently
- Platform comparisons are not in official docs
- Free tier/credit offers are marketing, not documentation

### 6. Quality Check

Verify research completeness:

| Check | Requirement |
|-------|-------------|
| Frontend | 2-3 options with docs from Context7 |
| Backend | 2-3 options with docs from Context7 |
| Database | Options with multi-tenant patterns from Context7 |
| Hosting | 2-3 options with pricing from WebSearch |
| Sources | All findings have citations (Context7 URLs or WebSearch sources) |

### 7. Synthesize Findings

Combine all results into a structured summary:

```yaml
research_summary:
  date: "[ISO 8601 timestamp]"
  context_file: "[context-file]"

  frontend:
    options:
      - name: ""
        context7_library_id: ""
        description: ""
        pros: []
        cons: []
        best_for: ""
        sources:
          - title: ""
            url: ""
    recommendation: ""
    rationale: ""

  backend:
    options:
      - name: ""
        context7_library_id: ""
        description: ""
        pros: []
        cons: []
        best_for: ""
        celery_support: ""
        multi_tenant_support: ""
        sources: []
    recommendation: ""
    rationale: ""

  database:
    options:
      - name: ""
        context7_library_id: ""
        storage_type: ""
        multi_tenant_pattern: ""
        pros: []
        cons: []
        sources: []
    recommendation: ""
    rationale: ""

  hosting:
    options:
      - name: ""
        type: ""  # PaaS, container, serverless
        mvp_cost: ""
        celery_redis_support: ""
        database_support: ""
        pros: []
        cons: []
        sources: []
    recommendation: ""
    rationale: ""

  all_sources:
    - title: ""
      url: ""
      type: "context7|websearch"
      layer: ""
```

### 8. Persist Results

Write to `memory/tech-context.yaml`:

```
Write(file_path: "memory/tech-context.yaml", content: research_summary)
```

## Research Queries Reference

### Frontend Queries

| Framework | Library ID | Key Query |
|-----------|------------|-----------|
| Next.js | `/vercel/next.js` | Authentication, data fetching, REST API integration |
| React | `/facebook/react` | Component patterns, state management |
| Vue | `/vuejs/vue` | Vue 3 composition API, state management |
| Nuxt | `/nuxt/nuxt` | Full-stack Vue with API routes |
| SvelteKit | `/sveltejs/kit` | Svelte full-stack framework |

### Backend Queries

| Framework | Library ID | Key Query |
|-----------|------------|-----------|
| Django | `/websites/djangoproject_en_6_0` | Celery, REST API, multi-tenant, authentication |
| FastAPI | `/fastapi/fastapi` | Background tasks, async, authentication |
| Express | `/expressjs/express` | REST API, middleware, error handling |
| NestJS | `/nestjs/nest` | Enterprise patterns, microservices |

### Database Queries

| Database | Library ID | Key Query |
|----------|------------|-----------|
| PostgreSQL | `/websites/postgresql_18` | Multi-tenant, RLS, performance |
| Redis | `/redis/redis` | Caching, pub/sub, Celery broker |
| MongoDB | `/mongodb/docs` | Document patterns, multi-tenant |

## Example Usage

### Phase 03 Invocation

```
[Phase 03 Activity 1]
↓
Skill(skill: "stack-research", args: "memory/system-patterns.yaml")
↓
[Context7 queries for frontend/backend/database]
[WebSearch for hosting/pricing]
↓
[Synthesize and return research_summary]
```

### Direct User Invocation

User types:
```
/stack-research memory/system-patterns.yaml
```

Agent executes skill, runs Context7 and WebSearch queries, returns research summary.

## Output

- `memory/tech-context.yaml` with:
  - Key findings per layer
  - 2-3 options per layer with pros/cons
  - Source citations (Context7 URLs and WebSearch sources)
  - Recommendations with rationale

## Success Criteria

- Context7 queries completed for frontend, backend, database
- WebSearch completed for hosting with 2025-2026 sources
- All findings have source citations
- Research summary persisted to memory/tech-context.yaml
- No placeholder values in output

## Error Handling

| Error | Resolution |
|-------|------------|
| Context7 library not found | Try alternative library name or version |
| Context7 query returns no results | Refine query with more specific terms |
| WebSearch returns stale results | Add year to query (e.g., "2026") |
| Context file not found | Ask user for correct path |
| Missing quality attribute | Note in research summary, proceed |

## Why This Approach

### Context7 for Libraries/Frameworks

- **Authoritative**: Official documentation, not random blogs
- **Current**: Version-specific docs (Django 6.0, Next.js 16, PostgreSQL 18)
- **Structured**: Code examples, API references
- **Citable**: Direct URLs to documentation

### WebSearch for Hosting

- **Pricing**: Changes frequently, not in docs
- **Comparisons**: Platform vs platform analysis
- **Free tiers**: Marketing offers, not technical docs
- **Real experiences**: User reviews, case studies

### Main Context Execution

- **Tool access**: Context7 and WebSearch both work in main context
- **No permission issues**: Sub-agents may not have tool access
- **Simpler**: One execution context, clear error handling

## Notes

- Runs entirely in main context (no sub-agent spawning)
- Context7 for technology documentation
- WebSearch for hosting/pricing/comparisons
- Results persist in memory/tech-context.yaml
- Can be re-run if Phase 04 surfaces new constraints
- Takes 2-5 minutes for complete research across all layers