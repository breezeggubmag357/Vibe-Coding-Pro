# Sprint Review-Fix Loop Mechanism

> This mechanism is the core differentiator of vibe-coding-pro from generic workflows. Validated by MediaForge project — the Sprint 1.5 fix sprint discovered 5 Critical security issues, preventing us from shipping with defects.

## Core Philosophy

**Every Sprint MUST be reviewed upon completion — non-negotiable. When Critical/Major issues are found, immediately launch a fix Sprint; only proceed to the next Sprint after re-verification passes.**

AI-generated code has a characteristic: it "looks runnable" but may contain severe vulnerabilities. If you proceed without review, issues accumulate layer by layer, and the cost of fixing them later in the project grows exponentially.

---

## Review Process

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

---

## Issue Severity Standards

### Critical (Blocks Progression)
- Security vulnerabilities: hardcoded secrets, CORS `*`, auth failure, SQL injection
- Data risks: data loss, dirty data, concurrency conflicts
- Functional blocking: core flows don't work, critical APIs unavailable
- Architecture defects: cross-service field mismatch causing message consumption failure

### Major (Affects Quality but Not Blocking)
- Missing test coverage (core modules < 60%)
- Documentation-implementation mismatch (declared 28 components, actual 1)
- Performance below target (API P95 > 500ms)
- Missing Config Beans (declared dependency but no @Bean)
- Cross-service naming inconsistency (frontend `xiaohongshu` vs backend `xhs`)

### Minor (Can Be Recorded in Backlog)
- Minor code style issues
- Missing comments
- Inappropriate log levels
- Small bugs in non-P0 features

---

## Review 4 Dimensions in Detail

### 1. Compilation Verification
```bash
# Backend
mvn compile -pl <service> -am
# Frontend
pnpm --filter <pkg> build
```
- All modules BUILD SUCCESS
- TypeScript 0 errors
- Check for deprecated API usage (e.g., Spring Boot 3.x removed `SecurityProperties.BASIC_SECURITY_ORDER` — should use `BASIC_AUTH_ORDER`)

### 2. Test Execution
```bash
mvn test -pl <service>
pnpm --filter <pkg> test
```
- Test pass rate 100%
- Core module coverage ≥ 60%
- Integration tests cover key paths

### 3. Security Checklist Re-verification
Check item-by-item against `scripts/security-checklist.md`:
- [ ] JWT Secret in env var (not hardcoded)
- [ ] AES Key in env var + 32 bytes
- [ ] CORS precise whitelist (not `*`)
- [ ] SecurityConfig `.authenticated()` (not `permitAll()`)
- [ ] admin password in env var
- [ ] bcrypt salting
- [ ] Sensitive data desensitization
- [ ] Rate limit configuration
- [ ] Audit logs
- [ ] Fail-Fast startup validation

### 4. Cross-Service Consistency Verification
See [cross-service-check.md](cross-service-check.md):
- API path prefix unified
- Status enums consistent
- MQ message body fields consistent
- Error code rules consistent
- Platform identifiers / subscription tiers and other enum values consistent across frontend/backend

---

## Fix Sprint (Sprint N.5)

### Launch Conditions
- Review finds ≥ 1 Critical
- OR ≥ 3 Major

### Execution Approach
1. List all issues (number + severity + fix plan)
2. Fix in parallel (backend issues + frontend issues dispatched to different subagents)
3. Immediately re-verify after fix
4. Only proceed to next Sprint after re-verification passes

### Real Case: MediaForge Sprint 1.5

Sprint 1 review found:
- **5 Critical**: JWT/AES default secrets, CORS `*`, admin password hardcoded, SecurityConfig `permitAll()`
- **10 Major**: SELECT *, init.sql missing role, platform identifier inconsistency, zero tests, RabbitMQ/MinIO Config missing, etc.

Launched Sprint 1.5 fix sprint:
- 2 subagents in parallel (backend fix + frontend fix)
- Backend modified 14 files (5 Critical + 4 Major)
- Frontend modified 5 files (platform identifier alignment)
- During fix, discovered 1 more Critical (AES default key only 30 bytes, needs 32)
- After re-verification passed, proceeded to Sprint 2

**Value**: Avoided carrying 5 security vulnerabilities through 6 Sprints, preventing late-stage rework.

---

## Review Deliverables

After every Sprint, output:

### 1. Completion Report
```markdown
# Sprint N Completion Report

## Deliverable Review
| Module | File Count | Test Cases | Verification Status |
|--------|-----------|-----------|---------------------|
| xxx    | xx        | xx        | ✅ BUILD SUCCESS |

## Cumulative Deliverables
| Dimension | Count |
|-----------|-------|
| New Java files | xx |
| New frontend files | xx |
| New tests | xx |
```

### 2. Issues Found and Fixed During Review
```markdown
## Issues Handled During Review
| # | Issue | Resolution |
|---|-------|------------|
| 1 | xxx   | fix description |
```

### 3. Cross-Service Consistency Verification Results
```markdown
## Cross-Service Consistency Verification
| Check Item | Result |
|------------|--------|
| API paths | ✅ |
| Status enums | ✅ |
```

### 4. Next Sprint Task Breakdown
```markdown
## Next Steps: Sprint N+1

### Task Groups
#### Group 1: xxx
| Number | Task | Priority |
|--------|------|----------|
| P4-XXX-001 | xxx | P0 |

### Key Constraints
1. xxx
2. xxx

### Acceptance Criteria
- [ ] xxx
```

---

## Anti-Patterns (Prohibited)

- ❌ Skipping review and proceeding directly
- ❌ Finding Critical but "proceed to next Sprint, fix later"
- ❌ Review only checks compilation, doesn't run tests
- ❌ Not verifying cross-service consistency
- ❌ Proceeding after fix without re-verification
- ❌ Review report only says "passed" without listing specific issues
