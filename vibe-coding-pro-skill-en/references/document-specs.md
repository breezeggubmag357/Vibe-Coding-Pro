# Document Format Specifications · 8 Deliverable Standards

> Distilled from real-world MediaForge project experience. All user-facing docs use unified HTML format (PRD/research/design/API/task list); SQL is standalone; MD is used for engineering docs.

## 1. research-report.html (Optional, recommended for B2C projects)

### Required Sections
1. Industry background and trends (with data charts)
2. Target user personas (prerequisites, capability requirements, external preparations)
3. Content generation types and AI application methods
4. Top player case studies (how they succeeded + how they monetize)
5. Competitor comparison table
6. Opportunities and recommendations

### Format Requirements
- Dark technical blueprint style (background `#0a0e0d` + neon green `#00ffa3` + neon blue `#00d4ff`)
- Data visualization (ECharts or CSS charts)
- Responsive layout
- Inlined CSS (single shareable file)

---

## 2. PRD.html (Required)

### Required 7 Sections
1. **Product Positioning & Core Value**: one-sentence positioning + value proposition canvas
2. **User Roles & Permission Matrix**: role table + permission matrix (role × feature)
3. **Feature List**: labeled by P0/P1/P2 priority, each feature with acceptance criteria
4. **Data Model & Entity Relationships**: ER diagram + entity field tables
5. **User Flow Diagrams**: including exception branches (Mermaid or flowchart)
6. **Non-functional Requirements**: performance / security / availability / compliance (quantified metrics)
7. **Explicit Non-Goals (Not-Do List)**: explicitly state features not being built

### Acceptance Criteria
- Each feature has Given/When/Then acceptance conditions
- Data model field types are explicit
- Permission matrix covers all roles

---

## 3. uiux-design-spec.html

### Required Sections
1. **Design System**: palette (primary/secondary/semantic) + typography (display + body + mono) + spacing system
2. **Component Library List**: declared component names + Props + use cases
3. **Layout Specs**: grid + breakpoints + container widths
4. **Interaction Specs**: animation duration + easing functions + state transitions
5. **Themes**: dark / light / brand theme switching

### Key Principles
- Declared component count MUST match actual implementation count (mismatch is a Major issue)
- Use CSS variables for colors to enable theme switching
- Don't use generic fonts (Inter/Roboto); pick distinctive ones

---

## 4. tech-architecture.html

### Required Sections
1. **Architecture Diagram**: system overview + service breakdown + data flow
2. **Tech Stack**: backend + frontend + middleware + deployment (with versions)
3. **Module Breakdown**: each service's responsibilities + port + dependencies
4. **Data Flow**: sequence diagrams for key business processes
5. **Third-party Services**: dependency list + alternatives
6. **Security Design**: auth chain + encryption system + rate-limiting strategy

---

## 5. swagger-api-doc.html

### Format Requirements
- OpenAPI 3.0 spec
- Grouped by service/domain (tag)
- Each endpoint: path + method + request body + response body + error codes + examples
- Error code convention: `{domain}{HTTP status}{number}` e.g. `USR404001`
- Interactive online (Swagger UI style)

### Backend Implementation Requirements
- One `OpenApiConfig.java` per microservice, unified config
- `@Tag` grouping, `@Operation` descriptions, `@Schema` field annotations

---

## 6. DDL.sql

### Required Content
1. **Table Creation**: `CREATE TABLE` (with field comments + table comments)
2. **Indexes**: primary key + unique indexes + business composite indexes
3. **Foreign Keys**: explicitly declared relationships (or maintained at app layer)
4. **Seed Data**: `INSERT INTO` for roles, permissions, dictionary tables
5. **Idempotency**: `CREATE TABLE IF NOT EXISTS` + `INSERT ... ON DUPLICATE KEY UPDATE`

### Naming Conventions
- Table: `t_` prefix + snake_case (e.g., `t_user`, `t_ai_task`)
- Field: snake_case (e.g., `created_at`, `user_id`)
- Index: `idx_{table_short}_{fields}` e.g. `idx_user_status`

### Required Fields (all business tables)
```sql
id BIGINT PRIMARY KEY,
created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
deleted_at DATETIME NULL  -- soft delete
```

### Flyway Migration Specs
- Filename: `V1.0.[n]__{description}.sql` (e.g., `V1.0.11__create_content_version_table.sql`)
- MUST be idempotent and re-entrant (`IF NOT EXISTS` / `ON DUPLICATE KEY`)
- MUST NOT conflict with earlier versions (e.g., V1.0.10 should not CREATE a table already created in V1.0.2 — use ALTER to add indexes)

---

## 7. review-and-kickoff.html

### Required Sections
1. **Product Review**: PRD review minutes + decisions
2. **UI Review**: design spec review + decisions
3. **Technical Review**: architecture review + decisions
4. **Kick-off**: team + milestones + Sprint plan

---

## 8. dev-task-breakdown.html

### Task Numbering System
`P[Stage]-[ModuleCode]-[Sequence]`
- P4-AIC-001 = P4 stage, AI Adapter module, task #1
- Module codes: USR (User) / CRT (Creation) / AIC (AI Adapter) / CNT (Content) / DST (Distribution) / ANL (Analytics) / MON (Monetization) / ADM (Admin)

### Required Fields
- Task number
- Task name
- Priority (P0/P1/P2)
- Assigned Sprint
- Acceptance criteria
- Dependencies

### Sprint Planning
- Each Sprint = 2 weeks
- Sprint 1.5 = fix sprint (unplanned Sprint, used to fix issues found during review)

---

## 9. DELIVERABLES.md (Required at Project Wrap-up)

### Format Spec
Categorize and list all deliverables:
1. Documentation (HTML/MD)
2. Backend (by service: Entity/DTO/VO/Mapper/Service/Controller/tests)
3. Frontend (by app: View/components/API Client/Store/routes)
4. Infrastructure (Docker/K8s/CI-CD/Flyway)
5. Tests (unit/integration/E2E/security review)
6. Scripts (deployment/verification/smoke)

Each item annotated with: file path + description.

---

## 10. security-checklist.md (Required at P6)

### Format Spec
Grouped by 6 dimensions (Auth & Authz / Data Security / API Security / Frontend Security / Encryption / Deployment Security), each item:
- [x] or [ ] checkbox
- Implementation location (file path + class/method)
- Sprint fix record (e.g., "Fixed in Sprint 1.5")

### Review Conclusion
- Passed items / total items
- Design-exempt items (e.g., CSRF not applicable due to Bearer Token)
- Recommended items to add before production launch

---

## Document Quality Baseline

- [ ] All HTML docs are single-file shareable (inlined CSS/JS)
- [ ] All SQL is idempotent and re-entrant
- [ ] All MD docs have a TOC
- [ ] Documentation matches code implementation (mismatch is a Major issue, must be fixed)
- [ ] Key decisions have an ADR (Architecture Decision Record)
