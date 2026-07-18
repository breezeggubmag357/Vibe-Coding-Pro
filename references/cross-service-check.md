# 跨服务一致性验证清单

> 基于 MediaForge 8 微服务架构实战沉淀。微服务最大隐患不是单服务 bug，而是跨服务"约定不一致"导致静默失败。

## 核心理念

微服务架构中，每个服务独立开发独立部署，但跨服务的"约定"必须一致：
- API 路径前缀
- 状态枚举值
- 错误码规则
- DTO/MQ 消息体字段
- 业务常量（平台标识、订阅等级、限流阈值）

不一致会导致：消息消费失败、前端传值后端不识别、错误码无法分类等静默问题。

---

## 验证 5 维度

### 1. API 路径前缀一致性

**规则**：所有后端 Controller 统一 `/api/v1/*` 前缀。

**验证方式**：
```bash
# 检查所有 Controller 的 @RequestMapping
grep -rn "@RequestMapping" server/*/src/main/java --include="*.java"
# 应全部是 /api/v1/xxx
```

**验证清单**：
- [ ] 所有 Controller 路径以 `/api/v1` 开头
- [ ] 前端 API Client 路径与后端完全匹配
- [ ] 无 `/api` 或 `/v1` 残缺前缀
- [ ] 公开接口（登录/健康检查）显式放行

### 2. 状态枚举跨服务一致

**规则**：同一业务状态码在所有服务中含义一致。

**典型枚举**：
- 内容状态：0草稿 / 1待审核 / 2已发布 / 3已下架 / 4审核中 / 5拒绝
- AI 任务状态：0排队 / 1处理中 / 2成功 / 3失败 / 4审核中 / 5超时
- 分发任务状态：0待发布 / 1发布中 / 2成功 / 3失败 / 4审核中 / 5已删除 / 6已撤回
- 商单状态：0洽谈 / 1已签约 / 2创作中 / 3已交付 / 4已回款 / 5已取消

**验证方式**：
- 后端：Service 层状态机 + 数据库字段注释
- 前端：`packages/shared/src/utils/constants.ts` 的 `XXX_STATUS_LABEL`
- 数据库：DDL 字段注释

**验证清单**：
- [ ] 同一状态码前后端含义一致
- [ ] 状态码在 DDL 字段注释中标注
- [ ] 状态机流转规则跨服务一致（如内容审核流程）

### 3. 错误码规则一致

**规则**：错误码格式统一为 `{域名}{HTTP状态}{编号}`。

| 域 | 前缀 | 示例 |
|----|------|------|
| 用户 | USR | USR404001 |
| 创作 | CRT | CRT400001 |
| AI | AI | AI500001 |
| 内容 | CNT | CNT403001 |
| 分发 | DST | DST409001 |
| 分析 | ANL | ANL500001 |
| 变现 | MON | MON400001 |
| 管理 | ADM | ADM403001 |

**验证清单**：
- [ ] 所有 ServiceException 用统一格式错误码
- [ ] 前端 ERROR_CODE_PREFIX 与后端一致
- [ ] 错误码有文档（Swagger 或错误码字典）

### 4. MQ 消息体字段跨服务一致

**规则**：消息生产者和消费者字段名/类型完全对齐。

**验证方式**：
- 对比生产者的 Message 类和消费者的 Message 类
- 字段名、类型、必填性必须一致
- 推荐共享 DTO 到 common 模块

**典型场景**：
- creation-service 发 `WorkflowMessage` → ai-adapter-service 消费为 `AiTaskMessage`
- 字段必须对齐：`params` / `subscriptionTier` / `retryCount` / `userId` / `taskId`

**验证清单**：
- [ ] 生产者/消费者 Message 类字段名一致
- [ ] 字段类型一致（如 `String` vs `Long` 不能混用）
- [ ] 必填字段一致
- [ ] 序列化方式一致（推荐 Jackson + 共享 DTO）

### 5. 业务常量跨前后端一致

**规则**：平台标识、订阅等级、限流阈值等常量前后端完全对齐。

**典型常量**：
- 平台标识：douyin / xhs / bilibili / wechat_video / wechat_mp / youtube / tiktok / weibo（8 个）
- 订阅等级：free / creator / team / enterprise
- AI 配额：free=100 / creator=1000 / team=5000 / enterprise=∞
- 限流 QPS：free=10 / creator=50 / team=200 / enterprise=1000
- 算法铁三角权重：分享3 + 收藏2 + 完播率2 + 互动率1

**验证方式**：
- 后端：`UserCacheKeys` / `IronScoreWeights` 等常量类
- 前端：`packages/shared/src/utils/constants.ts`
- 数据库：DDL GENERATED 列 + 字段注释

**验证清单**：
- [ ] 平台标识前后端数量一致（不能前端 9 后端 8）
- [ ] 平台命名规则一致（不能前端 `xiaohongshu` 后端 `xhs`）
- [ ] 订阅等级命名一致
- [ ] 限流阈值前后端一致
- [ ] 算法权重三处一致（后端常量 + DDL + 前端常量）

---

## 验证时机

### 每 Sprint 完成后必验
- 列出本 Sprint 涉及的跨服务交互
- 对照本清单逐项验证
- 发现不一致立即修复（视为 Critical）

### 真实案例：MediaForge Sprint 4 修复

Sprint 4 审验发现 3 处跨服务不一致：
1. 前端 `distribution.ts` 缺 `/api/v1` 前缀
2. 前端 `analytics.ts` 用 `/analytics/*` 但后端是 `/api/v1/metrics/events` 等
3. creation-service 的 `WorkflowMessage.inputParams` 与 ai-adapter-service 的 `AiTaskMessage.params` 字段名不一致 + 缺 `subscriptionTier`/`retryCount`

修复后避免了消息消费失败和前端请求 404。

---

## 验证自动化（推荐）

```bash
# 1. 检查 API 前缀一致性
grep -rn "@RequestMapping" server/*/src --include="*.java" | grep -v "/api/v1"

# 2. 检查前端 API 路径
grep -rn "url:" packages/api-client/src --include="*.ts" | grep -v "/api/v1"

# 3. 对比 MQ 消息体字段
diff <(grep "private" server/creation-service/.../WorkflowMessage.java) \
     <(grep "private" server/ai-adapter-service/.../AiTaskMessage.java)

# 4. 对比平台标识
grep -A20 "SUPPORTED_PLATFORMS" server/user-service/.../UserCacheKeys.java
grep -A20 "PLATFORMS" packages/shared/src/utils/constants.ts
```

---

## 反模式（禁止）

- ❌ 生产者改字段名不通知消费者
- ❌ 前端用 `xiaohongshu` 后端用 `xhs`
- ❌ 后端某服务用 `/api/v2` 其他用 `/api/v1`
- ❌ 错误码不按域名规则随意编
- ❌ 共享枚举散落各处（应放 common 或 shared）
