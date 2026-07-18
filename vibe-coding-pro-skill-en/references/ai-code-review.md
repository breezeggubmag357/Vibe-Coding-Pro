# AI Code Untrusted-Input Review Checklist · 10 Common Pitfalls

> Distilled from real-world MediaForge project experience. All issues are real-world trap cases that frequently appear in AI-generated code.

## Core Philosophy

AI-generated code "looks runnable" but may contain severe vulnerabilities. Review cannot focus only on functional correctness — you must check item-by-item against this list.

---

## Pitfall 1: Hardcoded Secrets (Critical)

### Symptom
```java
// ❌ AI-generated default secret
private static final String JWT_SECRET = "my-secret-key-12345678901234567890";
private static final String AES_KEY = "default-aes-key";
```

### Impact
- Secrets leaked into code repository
- Anyone can forge JWT tokens
- AES encryption is effectively useless

### Fix
```java
// ✅ Environment variable + Fail-Fast validation
@Value("${mediaforge.jwt.secret}")
private String jwtSecret;

@PostConstruct
public void validate() {
    if (jwtSecret == null || jwtSecret.length() < 32) {
        throw new IllegalStateException("JWT_SECRET must be configured and >= 32 bytes");
    }
}
```

### Review Checklist
- [ ] All Secret/Key/API_Key values read from environment variables
- [ ] Startup validation (Fail-Fast) exists
- [ ] Default values are dev-only, annotated "production must override"

---

## Pitfall 2: CORS `*` + credentials (Critical)

### Symptom
```java
// ❌ AI commonly generates insecure config
config.setAllowedOriginPatterns("*");
config.setAllowCredentials(true);
```

### Impact
- Any website can make cross-origin requests carrying user Cookie/Token
- CSRF attack risk

### Fix
```java
// ✅ Precise whitelist
config.setAllowedOrigins(List.of(
    "http://localhost:5173",
    "https://mediaforge.com"
));
config.setAllowCredentials(true);
```

### Review Checklist
- [ ] CORS whitelist is a precise origin list
- [ ] No `*` coexisting with `allowCredentials=true`
- [ ] Production configured via environment variables

---

## Pitfall 3: SecurityConfig `permitAll()` (Critical)

### Symptom
```java
// ❌ AI commonly generates "convenient for debugging" config
http.authorizeHttpRequests(auth -> auth.anyRequest().permitAll());
```

### Impact
- All endpoints bypass authentication
- JWT auth is effectively useless

### Fix
```java
// ✅ Default deny + explicit allow
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/api/v1/auth/**", "/actuator/health").permitAll()
    .anyRequest().authenticated()
)
.addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);
```

### Review Checklist
- [ ] Default `.authenticated()` (not `permitAll()`)
- [ ] Only explicitly allow public endpoints (login/health check/Swagger)
- [ ] JWT Filter integrated into filter chain

---

## Pitfall 4: Weak Encryption Key Length (Critical)

### Symptom
```java
// ❌ AES-256 requires 32-byte key, but AI gave 30 bytes
private static final byte[] KEY = "dev-only-aes-key-30-bytes".getBytes();  // 30 bytes
```

### Impact
- AES encryption fails on startup or downgrades to weak algorithm
- Encrypted data can be cracked

### Fix
```java
// ✅ Strict 32-byte validation
@PostConstruct
public void validate() {
    byte[] keyBytes = Base64.getDecoder().decode(aesKey);
    if (keyBytes.length != 32) {
        throw new IllegalStateException("AES_KEY must be base64-encoded 32 bytes");
    }
}
```

### Review Checklist
- [ ] AES-256 key is strictly 32 bytes
- [ ] Startup validates key length
- [ ] IV is randomly generated (not a fixed value)

---

## Pitfall 5: Cross-Service MQ Message Body Field Mismatch (Critical)

### Symptom
```java
// ❌ Producer service A
class WorkflowMessage { String inputParams; }

// Consumer service B
class AiTaskMessage { String params; }  // field name mismatch!
```

### Impact
- Message deserialization failure
- Consumer gets null for field values
- Silent failure, hard to debug

### Fix
```java
// ✅ Cross-service message body fields fully aligned
// Both service A and service B use:
class AiTaskMessage {
    String params;            // unified field name
    String subscriptionTier;   // add subscription tier
    Integer retryCount;        // add retry count
}
```

### Review Checklist
- [ ] Producer/consumer message body field names match
- [ ] Field types match
- [ ] Shared DTO placed in common module (recommended)

---

## Pitfall 6: SELECT * Queries (Major)

### Symptom
```java
// ❌ AI commonly generates
@Select("SELECT * FROM t_role WHERE code = #{code}")
Role selectByCode(String code);
```

### Impact
- Queries unnecessary fields (large text, sensitive fields)
- Implicit impact when schema changes
- Index coverage fails

### Fix
```java
// ✅ Explicit fields
@Select("SELECT id, code, name, description, data_scope, sort, status "
      + "FROM t_role WHERE code = #{code} AND deleted_at IS NULL LIMIT 1")
Role selectByCode(String code);
```

### Review Checklist
- [ ] All SQL uses explicit fields (not `*`)
- [ ] Soft-delete field `deleted_at IS NULL` added
- [ ] LIMIT 1 for single-row queries

---

## Pitfall 7: Cross-frontend/backend Enum Inconsistency (Major)

### Symptom
```typescript
// ❌ Frontend has 9 platforms
export const PLATFORMS = { douyin, xiaohongshu, bilibili, ... }

// Backend has 8 platforms
public static final String[] SUPPORTED_PLATFORMS = {"douyin", "xhs", ...};  // xiaohongshu → xhs
```

### Impact
- Frontend sends `xiaohongshu` but backend doesn't recognize it
- Data persistence fails or silently drops

### Fix
- Unify identifiers (e.g., `xhs`)
- Share enum constants in `packages/shared`
- Platform count consistent across frontend/backend

### Review Checklist
- [ ] Platform/status/tier and other enum values consistent across frontend/backend
- [ ] Enum count matches
- [ ] Naming rules match

---

## Pitfall 8: Declared Dependency Without Config Bean (Major)

### Symptom
```java
// ❌ pom.xml declares RabbitMQ dependency, but no @Bean config
@Autowired RabbitTemplate rabbitTemplate;  // startup failure
```

### Fix
```java
// ✅ Create RabbitMqConfig
@Configuration
public class RabbitMqConfig {
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory factory) {
        RabbitTemplate template = new RabbitTemplate(factory);
        template.setMessageConverter(jacksonConverter());
        return template;
    }
}
```

### Review Checklist
- [ ] All declared middleware dependencies have Config Beans
- [ ] ConnectionFactory/Template configured
- [ ] Serialization unified (Jackson)

---

## Pitfall 9: Calling Remote APIs Inside Transactions (Major)

### Symptom
```java
// ❌ Transaction method calls AI API
@Transactional
public void generate(AiTask task) {
    saveTask(task);
    aiClient.call(task);  // remote call, long transaction!
    updateResult(task);
}
```

### Impact
- Long transaction holds DB connection
- Remote timeout causes transaction rollback
- Severe performance degradation

### Fix
```java
// ✅ Split transaction boundary + async
public void generate(AiTask task) {
    saveTask(task);  // short transaction
    rabbitTemplate.convertAndSend("ai.task.queue", task);  // async dispatch
}
// Consumer handles remote call separately
```

### Review Checklist
- [ ] Transaction methods don't call remote APIs
- [ ] Long-running operations use MQ async
- [ ] Transaction scope minimized

---

## Pitfall 10: N+1 Queries (Major)

### Symptom
```java
// ❌ Query inside loop
List<User> users = userMapper.selectAll();
for (User u : users) {
    u.setRoles(roleMapper.selectByUserId(u.getId()));  // N queries
}
```

### Fix
```java
// ✅ Batch query
List<Long> userIds = users.stream().map(User::getId).toList();
Map<Long, List<Role>> roleMap = roleMapper.selectByUserIds(userIds)
    .stream().collect(groupingBy(UserRole::getUserId));
users.forEach(u -> u.setRoles(roleMap.get(u.getId())));
```

### Review Checklist
- [ ] No DB calls inside loops
- [ ] List queries use IN batch
- [ ] Use JOIN or batch queries instead of loops

---

## Review Prompt Template

```
Please review the following code for security and quality risks, focusing on:
1. Whether secrets are hardcoded
2. Whether CORS is a precise whitelist
3. Whether auth is complete (permitAll misuse)
4. Whether encryption key length is compliant
5. Whether cross-service fields/enums are consistent
6. Whether SQL uses SELECT *
7. Whether enum values match across frontend/backend
8. Whether dependencies have Config Beans
9. Whether transactions call remote APIs
10. Whether there are N+1 queries

[paste code]
```
