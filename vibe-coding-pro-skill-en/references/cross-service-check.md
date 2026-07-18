# Cross-Service Consistency Verification Checklist

> Distilled from real-world MediaForge 8-microservice architecture. The biggest hidden risk in microservices is not single-service bugs, but cross-service "contract inconsistencies" that cause silent failures.

## Core Philosophy

In microservice architectures, each service is developed and deployed independently, but cross-service "contracts" must be consistent:
- API path prefix
- Status enum values
- Error code rules
- DTO/MQ message body fields
- Business constants (platform identifiers, subscription tiers, rate-limit thresholds)

Inconsistency causes: message consumption failure, frontend sends values backend doesn't recognize, error codes can't be categorized, and other silent issues.

---

## 5 Verification Dimensions

### 1. API Path Prefix Consistency

**Rule**: All backend Controllers use unified `/api/v1/*` prefix.

**Verification**:
```bash
# Check all Controller @RequestMapping
grep -rn "@RequestMapping" server/*/src/main/java --include="*.java"
# Should all be /api/v1/xxx
```

**Checklist**:
- [ ] All Controller paths start with `/api/v1`
- [ ] Frontend API Client paths exactly match backend
- [ ] No `/api` or `/v1` truncated prefixes
- [ ] Public endpoints (login/health check) explicitly allowed

### 2. Status Enum Cross-Service Consistency

**Rule**: The same business status code has the same meaning in all services.

**Typical Enums**:
- Content status: 0 Draft / 1 Pending Review / 2 Published / 3 Archived / 4 Reviewing / 5 Rejected
- AI task status: 0 Queued / 1 Processing / 2 Success / 3 Failed / 4 Reviewing / 5 Timeout
- Distribution task status: 0 Pending / 1 Publishing / 2 Success / 3 Failed / 4 Reviewing / 5 Deleted / 6 Withdrawn
- Brand order status: 0 Negotiating / 1 Signed / 2 In Creation / 3 Delivered / 4 Paid / 5 Cancelled

**Verification**:
- Backend: Service layer state machine + DB field comments
- Frontend: `packages/shared/src/utils/constants.ts` `XXX_STATUS_LABEL`
- Database: DDL field comments

**Checklist**:
- [ ] Same status code has same meaning across frontend/backend
- [ ] Status codes annotated in DDL field comments
- [ ] State machine transition rules consistent across services (e.g., content review flow)

### 3. Error Code Rule Consistency

**Rule**: Error code format unified as `{domain}{HTTP status}{number}`.

| Domain | Prefix | Example |
|--------|--------|---------|
| User | USR | USR404001 |
| Creation | CRT | CRT400001 |
| AI | AI | AI500001 |
| Content | CNT | CNT403001 |
| Distribution | DST | DST409001 |
| Analytics | ANL | ANL500001 |
| Monetization | MON | MON400001 |
| Admin | ADM | ADM403001 |

**Checklist**:
- [ ] All ServiceException uses unified error code format
- [ ] Frontend ERROR_CODE_PREFIX matches backend
- [ ] Error codes have documentation (Swagger or error code dictionary)

### 4. MQ Message Body Field Cross-Service Consistency

**Rule**: Producer and consumer field names/types fully aligned.

**Verification**:
- Compare producer Message class with consumer Message class
- Field name, type, required-ness must match
- Recommended: share DTO in common module

**Typical Scenario**:
- creation-service sends `WorkflowMessage` → ai-adapter-service consumes as `AiTaskMessage`
- Fields must align: `params` / `subscriptionTier` / `retryCount` / `userId` / `taskId`

**Checklist**:
- [ ] Producer/consumer Message class field names match
- [ ] Field types match (e.g., no mixing `String` vs `Long`)
- [ ] Required fields match
- [ ] Serialization format consistent (recommended Jackson + shared DTO)

### 5. Business Constants Cross-Frontend/Backend Consistency

**Rule**: Platform identifiers, subscription tiers, rate-limit thresholds and other constants fully aligned across frontend/backend.

**Typical Constants**:
- Platform identifiers: douyin / xhs / bilibili / wechat_video / wechat_mp / youtube / tiktok / weibo (8 total)
- Subscription tiers: free / creator / team / enterprise
- AI quota: free=100 / creator=1000 / team=5000 / enterprise=∞
- Rate-limit QPS: free=10 / creator=50 / team=200 / enterprise=1000
- Algorithm iron-triangle weights: share=3 + favorite=2 + completion_rate=2 + interaction_rate=1

**Verification**:
- Backend: `UserCacheKeys` / `IronScoreWeights` and other constant classes
- Frontend: `packages/shared/src/utils/constants.ts`
- Database: DDL GENERATED columns + field comments

**Checklist**:
- [ ] Platform identifier count matches across frontend/backend (no 9 vs 8)
- [ ] Platform naming rules match (no frontend `xiaohongshu` vs backend `xhs`)
- [ ] Subscription tier naming matches
- [ ] Rate-limit thresholds match across frontend/backend
- [ ] Algorithm weights match in 3 places (backend constant + DDL + frontend constant)

---

## Verification Timing

### Must Verify After Every Sprint
- List cross-service interactions in this Sprint
- Verify item-by-item against this checklist
- If inconsistency found, fix immediately (treat as Critical)

### Real Case: MediaForge Sprint 4 Fix

Sprint 4 review found 3 cross-service inconsistencies:
1. Frontend `distribution.ts` missing `/api/v1` prefix
2. Frontend `analytics.ts` used `/analytics/*` but backend was `/api/v1/metrics/events` etc.
3. creation-service `WorkflowMessage.inputParams` vs ai-adapter-service `AiTaskMessage.params` field name mismatch + missing `subscriptionTier`/`retryCount`

After fix, avoided message consumption failure and frontend 404 errors.

---

## Verification Automation (Recommended)

```bash
# 1. Check API prefix consistency
grep -rn "@RequestMapping" server/*/src --include="*.java" | grep -v "/api/v1"

# 2. Check frontend API paths
grep -rn "url:" packages/api-client/src --include="*.ts" | grep -v "/api/v1"

# 3. Compare MQ message body fields
diff <(grep "private" server/creation-service/.../WorkflowMessage.java) \
     <(grep "private" server/ai-adapter-service/.../AiTaskMessage.java)

# 4. Compare platform identifiers
grep -A20 "SUPPORTED_PLATFORMS" server/user-service/.../UserCacheKeys.java
grep -A20 "PLATFORMS" packages/shared/src/utils/constants.ts
```

---

## Anti-Patterns (Prohibited)

- ❌ Producer changes field name without notifying consumer
- ❌ Frontend uses `xiaohongshu` while backend uses `xhs`
- ❌ One backend service uses `/api/v2` while others use `/api/v1`
- ❌ Error codes assigned without following domain rules
- ❌ Shared enums scattered across places (should be in common or shared)
