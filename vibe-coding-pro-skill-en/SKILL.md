---
name: vibe-coding-pro
description: A Conscious Vibe Coding development workflow guide. Covers the complete 10-stage closed loop: Requirements → Design → Context → Architecture → Development → Integration → Security → Performance → Deployment → Monitoring. It reinforces three core mechanisms: Sprint review-fix loop, AI code untrusted-input review, and cross-service consistency verification. Use when users need to create production-grade projects from scratch, plan development milestones, or systematically advance vibecoding development. Core principle: humans lead architecture and decisions, AI handles implementation and verification; deliver in stages, every step verifiable, every Sprint must be reviewed.
metadata:
  author: Foppa
---

# Vibe Coding Pro · Production-Grade Development Workflow

## Applicable Scenarios

- **Production-grade projects**: Projects that will be deployed to real environments serving real users
- **Full-stack projects**: Frontend-backend separation / microservices / multi-client (Web + Admin + Mobile)
- **AI-assisted development**: Using AI (Claude/GPT/Cursor/Trae) as the primary implementer

Not applicable to: one-off scripts, personal toy projects, demos with no security requirements.

## First Principle: Conscious Vibe Coding

**Golden Rule: Every line of AI-generated code is treated as untrusted input — it must pass both human review and automated verification before entering the main branch.**

| Level | Philosophy | Suitable For | Risk |
|-------|-----------|--------------|------|
| Pure Vibe | Accept all AI output | One-off prototypes | Very High |
| Casual Vibe | Light review | Personal MVP | Medium |
| **Conscious Vibe** | **Guide + Verify + Review Loop** | **Production-grade** | **Low** |
| Hybrid | AI scaffolding + human core code | Enterprise / Security-sensitive | Very Low |

### 3 Core Mechanisms Reinforced by This Skill (vs. generic vibe-coding-workflow)

1. **Sprint Review-Fix Loop**: Every Sprint MUST be reviewed upon completion. When Critical/Major issues are found, immediately launch a fix Sprint; only proceed after re-verification passes. See [references/sprint-review.md](references/sprint-review.md).
2. **AI Code Untrusted-Input Review**: A 10-item common-pitfall checklist (including real-world traps like default JWT secrets, CORS `*`, SecurityConfig `permitAll()`, cross-service MQ message field mismatch, etc.). See [references/ai-code-review.md](references/ai-code-review.md).
3. **Cross-Service Consistency Verification**: Microservice architectures MUST verify that API paths, status enums, error codes, DTO fields, and MQ message bodies are consistent across services. See [references/cross-service-check.md](references/cross-service-check.md).

---

## 10-Stage Development Workflow Overview

```
P0 Requirements → P1 Design → P2 Context → P3 Architecture → P4 Development
→ P5 Integration → P6 Security → P7 Performance → P8 Deployment → P9 Monitoring
```

Each stage has: **Input → Execution Steps → Output Artifacts (with format specs) → Acceptance Criteria → Checklist**.

Document format specifications: see [references/document-specs.md](references/document-specs.md).

---

## P0 · Requirements Analysis

### Goal
Transform vague ideas into a structured PRD that AI can accurately implement.

### Execution Steps
1. Problem definition: What problem to solve, target users, core value
2. User stories: For each role, write "As X, I need Y, so that Z"
3. Acceptance criteria: Given/When/Then format
4. Data model: Core entities + field types + relationships
5. Non-functional requirements: Performance / security / scalability / compliance
6. Explicit non-goals: Clearly state what is "not done" to prevent scope creep

### Output Artifacts (HTML format)
- `research-report.html`: Industry research + competitive analysis (optional, recommended for B2C projects)
- `PRD.html`: Product Requirements Document (must include 7 sections: positioning/roles/feature list/data model/user flows/non-functional/non-goals)

### Acceptance Criteria
- [ ] Every feature has acceptance criteria
- [ ] Data model field types are explicit
- [ ] Permission matrix covers all roles
- [ ] Non-goals are explicitly listed

---

## P1 · Technical Design

### Goal
Before writing any code, determine the tech stack, architecture, directory structure, API contracts, and database schema.

### Execution Steps
1. Tech stack selection ("use familiar tools to do the right thing")
2. Architecture design (layered / microservices / monolith)
3. Directory structure defined down to the second level
4. API contracts (RESTful, including error code conventions)
5. Database schema (tables + indexes + migration strategy)
6. Third-party service dependency list + alternatives

### Output Artifacts (HTML format)
- `uiux-design-spec.html`: Design system (palette / typography / spacing / component library + themes)
- `tech-architecture.html`: Architecture diagram + tech stack + module breakdown + data flow
- `swagger-api-doc.html`: OpenAPI 3.0 full API specification
- `DDL.sql`: Table creation scripts (with indexes, foreign keys, seed data)

### Acceptance Criteria
- [ ] Directory structure defined to second level
- [ ] API contracts include error code conventions
- [ ] Database index strategy designed
- [ ] Third-party services have alternatives

---

## P2 · Context Engineering

### Goal
Create a persistent project context for AI. **High-quality context + a mediocre prompt vastly outperforms a great prompt + poor context.**

### 5 Pillars of Context (each < 1000 words)

| Pillar | File | Content |
|--------|------|---------|
| Project Structure | `.trae/rules/project.md` | Directory layout, naming conventions, architecture patterns |
| Code Style | `.trae/rules/code-style.md` | Naming rules + standard templates (Controller/Service/Mapper/components) |
| Domain Knowledge | `.trae/rules/domain.md` | Glossary, business rules, state machines, entity relationships |
| Code Examples | `.trae/rules/examples.md` | Existing code snippets for AI pattern matching |
| Constraints | `.trae/rules/constraints.md` | Security red lines, performance baselines, prohibited actions |

### Key Principles
- Rule files **< 1000 words each** (longer dilutes AI attention)
- Load on demand: base rules always loaded, scenario-specific rules loaded per module
- More rules ≠ better (3 concise rules > a 12,000-word spec document)

### Acceptance Criteria
- [ ] Each rule file < 1000 words
- [ ] Committed to version control
- [ ] Naming conventions have examples
- [ ] Security constraints are written down

---

## P3 · Infrastructure Setup

### Goal
Build the project skeleton so subsequent module development has a foundation.

### Execution Steps
1. Initialize Monorepo or multi-project structure
2. Backend skeleton: framework init + DB/Redis/MQ connections + logging + health checks
3. Frontend skeleton: routing + state management + API wrapper + UI framework
4. Common capabilities: unified response, exception filter, auth guard, validation pipeline
5. Seed data: roles, permissions, dictionary tables, test accounts
6. Development environment: Docker Compose one-click startup

### Output Artifacts
- Monorepo directory structure (server/ + frontend + packages/ + docker/ + database/)
- Common module common/ (response/exception/JWT/crypto/CORS/rate-limit/audit/cache/utils)
- docker-compose.yml (DB + Redis + MQ + object storage + service registry)
- .env.example (all env var templates, with dev placeholders + "production must override" comments)
- Flyway migration scripts V1.0.0~V1.0.x (ordered by version, idempotent and re-entrant)

### Key Design: Fail-Fast Startup Validation
- JWT Secret validated on startup ≥ 32 bytes; throw exception to block startup if missing
- AES Key validated as base64 + 32 bytes after decoding
- Database connection failure fails fast, no infinite retries

### Acceptance Criteria
- [ ] `docker compose up` can start all middleware
- [ ] Health check endpoint returns 200
- [ ] Seed data can be loaded
- [ ] Swagger docs are accessible
- [ ] .env.example includes all required variables

---

## P4 · Core Module Development

### Goal
Implement core functional modules by priority, each independently verifiable.

### Task Numbering System
`P[Stage]-[ModuleCode]-[Sequence]`, e.g., `P4-AIC-001` = P4 stage, AI Adapter module, task #1.

### Single Module Development SOP
```
1. Define data entity (Entity)
2. Write DTOs (create/update/query)
3. Implement Service layer (business logic)
4. Implement Controller layer (routing + parameter validation)
5. Write unit tests (core path coverage, target ≥ 60%)
6. Integration tests (API-level verification)
7. Manual Code Review (security/performance/conventions)
```

### Prompt Best Practice: Small Steps, Fast Iteration
```
❌ "Build a complete user management system"
✅ "Create a login form with email + password fields, form validation, and error state display"
```

### Output Artifacts
- Backend: Entity + DTO + VO + Mapper + Service(Impl) + Controller + application.yml + tests
- Frontend: View + components + API Client + Store + routes + tests
- DDL: New table Flyway migration scripts (V1.0.[n]__description.sql, idempotent)

### Acceptance Criteria
- [ ] Every API has DTO validation (JSR-303 / class-validator)
- [ ] Sensitive operations have permission guards
- [ ] List endpoints support pagination
- [ ] No N+1 queries
- [ ] Exceptions have explicit error codes
- [ ] Core module unit test coverage ≥ 60%

---

## P5 · Frontend-Backend Integration

### Goal
Frontend correctly consumes backend APIs; data flow is complete.

### Execution Steps
1. API alignment: Frontend API Client paths exactly match backend routes (unified `/api/v1/*` prefix)
2. Type sync: Frontend TS types match backend DTOs
3. Error handling: 401/403/500 frontend behavior
4. Loading states: prevent duplicate submissions
5. End-to-end verification: walk through core user flows

### Cross-Service Consistency Must-Check (see [references/cross-service-check.md](references/cross-service-check.md))
- API path prefix unified `/api/v1`
- Status enums consistent across services (e.g., content status codes 0/1/2/3/4/5)
- MQ message body fields consistent between producers/consumers
- Error code naming consistent (`{domain}{HTTP}{number}` e.g. USR404001)

### Acceptance Criteria
- [ ] Core flows work end-to-end
- [ ] No Console errors
- [ ] No 4xx/5xx network requests (except business exceptions)
- [ ] Forms have duplicate-submission prevention
- [ ] Empty/error states have UI feedback

---

## P6 · Security Hardening

### Goal
Systematic security review. **Security is the most easily overlooked aspect of Vibe Coding — AI code "looks runnable" but may contain vulnerabilities.**

### Security Check 6 Dimensions (see [references/ai-code-review.md](references/ai-code-review.md))
1. Auth & Authorization: JWT Secret in env vars, token expiration, RBAC, login brute-force protection
2. Data Security: bcrypt, desensitization, parameterized SQL, input validation, file upload limits
3. API Security: rate limiting, CORS whitelist (no `*` + credentials), audit logs
4. Frontend Security: token storage, XSS, CSRF
5. Encryption: AES-256-CBC, payment callback signature verification, HTTPS
6. Deployment Security: DB port not exposed, Secrets in env vars, minimized images, non-root runtime

### Fail-Fast Design (Mandatory)
- JWT/AES/DB passwords/API Keys validated on startup; throw exception to block startup if missing or weak
- Default values are dev-only; must use `${ENV_VAR:dev-default}` placeholder, production must override

### Output Artifacts
- `scripts/security-checklist.md`: Item-by-item verification checklist (each item annotated with implementation location + Sprint fix record)

### Acceptance Criteria
- [ ] Security checklist fully passes
- [ ] No hardcoded secrets
- [ ] Sensitive data is desensitized
- [ ] APIs have access control
- [ ] Fail-Fast mechanism is in effect

---

## P7 · Performance Optimization

### Backend Optimization Checklist
| Item | Method | Priority |
|------|--------|----------|
| Redis Cache | TTL tiers (base data 1h / profiles 5min / details 1min / dashboards 10s) | P0 |
| Database Indexes | Composite indexes on common query conditions | P0 |
| Query Optimization | Eliminate N+1, use JOIN or batch queries | P1 |
| Pagination Control | Enforce pagination, pageSize cap 100 | P1 |
| Response Compression | gzip/brotli | P2 |
| Connection Pool | HikariCP tuning | P2 |

### Frontend Optimization Checklist
| Item | Method | Priority |
|------|--------|----------|
| Code Splitting | Route lazy loading + CSS code splitting | P0 |
| Build Optimization | terser (drop_console + drop_debugger) | P0 |
| Dependency Chunking | Vite manualChunks to split large deps | P1 |
| Image Lazy Loading | `loading="lazy"` | P2 |
| Virtual Lists | Virtual scrolling for large lists | P2 |

### Acceptance Criteria
- [ ] API P95 < 200ms (excluding AI calls)
- [ ] First page load < 3s
- [ ] Lighthouse > 80
- [ ] No memory leaks

---

## P8 · Deployment

### Output Artifacts
- One multi-stage Dockerfile per service (build stage + runtime stage, alpine-based)
- docker-compose.yml (full service orchestration: middleware + backend + frontend)
- Nginx config (SPA routing fallback + API proxy + gzip + static cache)
- K8s manifests (Deployment + Service + Ingress + ConfigMap + Secret)
- CI/CD pipeline (.gitlab-ci.yml: lint→test→build→scan→image→push→deploy→verify, 8 stages)
- Full Flyway migration verification (sequential execution, no conflicts)

### Acceptance Criteria
- [ ] All services have Dockerfiles
- [ ] `docker compose up` starts everything
- [ ] .env.example includes all required variables
- [ ] DB port not exposed in production
- [ ] Logs have persistent Volumes
- [ ] Health check configured

---

## P9 · Monitoring & Alerting

### Three-Layer Monitoring
1. **Logging Layer**: Structured JSON + daily rotation + 30-day retention + separate error log + TraceId propagation
2. **Health Check Layer**: `/actuator/health` with DB/Redis/MQ/object storage probes
3. **Alerting Layer**: API P95>500ms / error rate>5% / service unavailable / memory>80%

### Output Artifacts
- HealthIndicator (one each for DB/Redis/MQ/MinIO)
- Prometheus metrics exposure + business metrics (AI/distribution/content/payment)
- Grafana dashboards (overview + business)
- Alert rules (Prometheus Alertmanager)

### Acceptance Criteria
- [ ] Logs rotate daily with retention period
- [ ] Request logs include latency
- [ ] Health check endpoint is usable
- [ ] Error logs in a separate file
- [ ] Logs are structured JSON

---

## Sprint Review-Fix Loop (Core Mechanism)

Every Sprint MUST be reviewed upon completion — non-negotiable. See [references/sprint-review.md](references/sprint-review.md).

### Review Process
```
Sprint N complete
  ↓
Review 4 dimensions
  ├─ Compilation verification (mvn compile / pnpm build)
  ├─ Test execution (mvn test / pnpm test)
  ├─ Security checklist re-verification (P6 list)
  └─ Cross-service consistency verification (API/enums/error codes/MQ message body)
  ↓
Issues found?
  ├─ Critical → immediately launch Sprint N.5 fix sprint
  ├─ Major → launch Sprint N.5 or fold into next Sprint
  └─ Minor → record in backlog
  ↓
Fix complete → re-verify → pass → proceed to Sprint N+1
```

### Issue Severity Levels
- **Critical**: Blocks progression (security vulnerabilities, data loss risk, auth failure)
- **Major**: Affects quality but not blocking (missing tests, doc mismatch, performance below target)
- **Minor**: Can be recorded in backlog for later

### Review Deliverables
After every Sprint, output:
1. Completion report (modules / file count / test cases / verification status)
2. List of issues found and fixed during review
3. Cross-service consistency verification results
4. Next Sprint task breakdown

---

## Milestone Planning Template

| Milestone | Stage | Core Deliverable | Acceptance |
|-----------|-------|------------------|------------|
| M0 | P0-P1 | Research + PRD + Design Docs | Doc review |
| M1 | P2-P3 | Context + Project Skeleton | docker-compose up works |
| M2 | P4 | Core module CRUD | APIs callable |
| M3 | P4-P5 | Integration + Feature completion | End-to-end works |
| M4 | P6-P7 | Security + Performance | Checklist passes |
| M5 | P8-P9 | Deployment + Monitoring | Production accessible |

---

## General Pitfall Guide (Distilled from Real Projects)

1. **Don't let AI generate large modules at once**: small steps, verify each time before continuing
2. **More context isn't better**: rule files < 1000 words, otherwise AI attention diffuses
3. **Long sessions lose context**: write key agreements into `.trae/rules/`, don't rely on chat memory
4. **AI code "looks right but has hidden traps"**: common pitfalls in [references/ai-code-review.md](references/ai-code-review.md)
5. **Prefer free/open-source third-party services**: OSS→MinIO, SMS→SMTP, Maps→OSM
6. **Port conflicts**: check before startup, Docker Compose services must not conflict
7. **Database migration irreversible ops caution**: use migration scripts in prod, disable `synchronize: true`
8. **Environment variable management**: sensitive info only in `.env` (gitignored), `.env.example` as template
9. **Preview fallback**: configure Mock login fallback in frontend so UI can still be previewed when backend is unreachable
10. **Structured deliverables list**: at project wrap-up, output `DELIVERABLES.md` categorizing all deliverables

---

## How to Use

1. **New project**: start from P0, advance stage by stage, confirm with user after each stage
2. **Existing project**: diagnose current stage, continue from there
3. **Troubleshooting**: jump to the relevant stage's checklist
4. **Milestone planning**: use the milestone template, tailor to project scale

### Interaction Principles
- Report after each stage, wait for user confirmation before continuing
- When tech selection diverges, provide comparison tables for user decision
- Proactively flag risks, don't execute blindly
- Use tables and checklists instead of long paragraphs
- Every Sprint must be reviewed; output next Sprint task breakdown

### Difference from vibe-coding-workflow
- vibe-coding-workflow: generic workflow guidance, good for quickly understanding the methodology
- **vibe-coding-pro (this skill)**: reinforces Sprint review loop, AI code review, cross-service consistency, document specs, and Fail-Fast design — for production-grade projects
