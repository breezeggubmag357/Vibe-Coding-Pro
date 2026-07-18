# AI 代码不可信输入审查清单 · 10 大常见坑

> 基于 MediaForge 项目实战沉淀。所有问题均为真实踩坑案例，AI 生成代码时高频出现。

## 核心理念

AI 生成的代码"看起来能跑"但可能含严重漏洞。审查时不能只看功能正确性，必须按此清单逐项排查。

---

## 坑 1：硬编码密钥（Critical）

### 现象
```java
// ❌ AI 生成的默认密钥
private static final String JWT_SECRET = "my-secret-key-12345678901234567890";
private static final String AES_KEY = "default-aes-key";
```

### 危害
- 密钥泄露到代码仓库
- 任何人可伪造 JWT Token
- AES 加密形同虚设

### 修复
```java
// ✅ 环境变量 + Fail-Fast 校验
@Value("${mediaforge.jwt.secret}")
private String jwtSecret;

@PostConstruct
public void validate() {
    if (jwtSecret == null || jwtSecret.length() < 32) {
        throw new IllegalStateException("JWT_SECRET 必须配置且 ≥ 32 字节");
    }
}
```

### 审查清单
- [ ] 所有 Secret/Key/API_Key 都从环境变量读取
- [ ] 有启动时校验（Fail-Fast）
- [ ] 默认值仅 dev 环境，标注"生产必须覆盖"

---

## 坑 2：CORS 配置 `*` + credentials（Critical）

### 现象
```java
// ❌ AI 常生成的不安全配置
config.setAllowedOriginPatterns("*");
config.setAllowCredentials(true);
```

### 危害
- 任何网站可携带用户 Cookie/Token 发起跨域请求
- CSRF 攻击风险

### 修复
```java
// ✅ 精确白名单
config.setAllowedOrigins(List.of(
    "http://localhost:5173",
    "https://mediaforge.com"
));
config.setAllowCredentials(true);
```

### 审查清单
- [ ] CORS 白名单是精确 origin 列表
- [ ] 禁止 `*` 与 `allowCredentials=true` 共存
- [ ] 生产环境通过环境变量配置

---

## 坑 3：SecurityConfig `permitAll()`（Critical）

### 现象
```java
// ❌ AI 常生成的"方便调试"配置
http.authorizeHttpRequests(auth -> auth.anyRequest().permitAll());
```

### 危害
- 所有接口免鉴权
- JWT 鉴权形同虚设

### 修复
```java
// ✅ 默认拒绝 + 显式放行
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/api/v1/auth/**", "/actuator/health").permitAll()
    .anyRequest().authenticated()
)
.addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);
```

### 审查清单
- [ ] 默认 `.authenticated()`（非 `permitAll()`）
- [ ] 仅显式放行公开接口（登录/健康检查/Swagger）
- [ ] JWT Filter 已集成到过滤链

---

## 坑 4：弱加密密钥长度（Critical）

### 现象
```java
// ❌ AES-256 需要 32 字节密钥，但 AI 给了 30 字节
private static final byte[] KEY = "dev-only-aes-key-30-bytes".getBytes();  // 30 字节
```

### 危害
- AES 加密启动失败或降级到弱算法
- 加密数据可被破解

### 修复
```java
// ✅ 严格校验 32 字节
@PostConstruct
public void validate() {
    byte[] keyBytes = Base64.getDecoder().decode(aesKey);
    if (keyBytes.length != 32) {
        throw new IllegalStateException("AES_KEY 必须是 base64 编码的 32 字节");
    }
}
```

### 审查清单
- [ ] AES-256 密钥严格 32 字节
- [ ] 启动时校验密钥长度
- [ ] IV 随机生成（非固定值）

---

## 坑 5：跨服务 MQ 消息体字段不一致（Critical）

### 现象
```java
// ❌ 生产者 service A
class WorkflowMessage { String inputParams; }

// 消费者 service B
class AiTaskMessage { String params; }  // 字段名不一致！
```

### 危害
- 消息反序列化失败
- 消费者取不到字段值（null）
- 静默失败，难排查

### 修复
```java
// ✅ 跨服务消息体字段完全对齐
// service A 和 service B 都用：
class AiTaskMessage {
    String params;            // 统一字段名
    String subscriptionTier;   // 补齐订阅等级
    Integer retryCount;        // 补齐重试次数
}
```

### 审查清单
- [ ] 生产者/消费者消息体字段名一致
- [ ] 字段类型一致
- [ ] 共享 DTO 放 common 模块（推荐）

---

## 坑 6：SELECT * 查询（Major）

### 现象
```java
// ❌ AI 常生成
@Select("SELECT * FROM t_role WHERE code = #{code}")
Role selectByCode(String code);
```

### 危害
- 查询 unnecessary 字段（如大文本、敏感字段）
- 表结构变更时隐式影响
- 索引覆盖失效

### 修复
```java
// ✅ 明确字段
@Select("SELECT id, code, name, description, data_scope, sort, status "
      + "FROM t_role WHERE code = #{code} AND deleted_at IS NULL LIMIT 1")
Role selectByCode(String code);
```

### 审查清单
- [ ] 所有 SQL 明确字段（非 `*`）
- [ ] 软删除字段 `deleted_at IS NULL` 已加
- [ ] LIMIT 1 用于单条查询

---

## 坑 7：跨前后端枚举不一致（Major）

### 现象
```typescript
// ❌ 前端 9 个平台
export const PLATFORMS = { douyin, xiaohongshu, bilibili, ... }

// 后端 8 个平台
public static final String[] SUPPORTED_PLATFORMS = {"douyin", "xhs", ...};  // xiaohongshu → xhs
```

### 危害
- 前端传 `xiaohongshu` 后端不识别
- 数据入库失败或静默丢弃

### 修复
- 统一标识符（如 `xhs`）
- 共享枚举常量到 `packages/shared`
- 平台数量前后端一致

### 审查清单
- [ ] 平台/状态/等级等枚举值前后端一致
- [ ] 枚举数量一致
- [ ] 命名规则一致

---

## 坑 8：声明依赖但无 Config Bean（Major）

### 现象
```java
// ❌ pom.xml 声明了 RabbitMQ 依赖，但无 @Bean 配置
@Autowired RabbitTemplate rabbitTemplate;  // 启动失败
```

### 修复
```java
// ✅ 新建 RabbitMqConfig
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

### 审查清单
- [ ] 所有声明的中间件依赖都有 Config Bean
- [ ] ConnectionFactory/Template 已配置
- [ ] 序列化方式统一（Jackson）

---

## 坑 9：事务中调用远程接口（Major）

### 现象
```java
// ❌ 事务方法中调用 AI API
@Transactional
public void generate(AiTask task) {
    saveTask(task);
    aiClient.call(task);  // 远程调用，长事务！
    updateResult(task);
}
```

### 危害
- 长事务占用数据库连接
- 远程超时导致事务回滚
- 性能急剧下降

### 修复
```java
// ✅ 拆分事务边界 + 异步
public void generate(AiTask task) {
    saveTask(task);  // 短事务
    rabbitTemplate.convertAndSend("ai.task.queue", task);  // 异步投递
}
// 消费者单独处理远程调用
```

### 审查清单
- [ ] 事务方法不调用远程接口
- [ ] 长耗时操作用 MQ 异步
- [ ] 事务范围最小化

---

## 坑 10：N+1 查询（Major）

### 现象
```java
// ❌ 循环中查询
List<User> users = userMapper.selectAll();
for (User u : users) {
    u.setRoles(roleMapper.selectByUserId(u.getId()));  // N 次查询
}
```

### 修复
```java
// ✅ 批量查询
List<Long> userIds = users.stream().map(User::getId).toList();
Map<Long, List<Role>> roleMap = roleMapper.selectByUserIds(userIds)
    .stream().collect(groupingBy(UserRole::getUserId));
users.forEach(u -> u.setRoles(roleMap.get(u.getId())));
```

### 审查清单
- [ ] 循环中无数据库调用
- [ ] 列表查询用 IN 批量
- [ ] 用 JOIN 或批量查询替代循环

---

## 审查 Prompt 模板

```
请审查以下代码的安全与质量风险，重点检查：
1. 是否硬编码密钥
2. CORS 配置是否精确白名单
3. 鉴权是否完整（permitAll 误用）
4. 加密密钥长度是否合规
5. 跨服务字段/枚举是否一致
6. SQL 是否 SELECT *
7. 枚举值前后端是否一致
8. 依赖是否有 Config Bean
9. 事务中是否调用远程接口
10. 是否有 N+1 查询

[粘贴代码]
```
