# Sprint 审验-修复循环机制

> 本机制是 vibe-coding-pro 区别于通用流程的核心。基于 MediaForge 项目实战验证——Sprint 1.5 修复冲刺发现了 5 项 Critical 安全问题，避免了带病推进。

## 核心理念

**每个 Sprint 完成后必须审验，不可跳过。发现 Critical/Major 问题立即进入修复 Sprint，复验通过才能推进下一 Sprint。**

AI 生成代码的特点是"看起来能跑"但可能含严重漏洞。如果不审验就推进，问题会层层累积，到项目后期修复成本指数级上升。

---

## 审验流程

```
Sprint N 完成
  ↓
审验 4 维度
  ├─ 编译验证（mvn compile / pnpm build）
  ├─ 测试执行（mvn test / pnpm test）
  ├─ 安全清单复验（P6 清单）
  └─ 跨服务一致性验证（API/枚举/错误码/MQ消息体）
  ↓
发现问题？
  ├─ Critical → 立即启动 Sprint N.5 修复冲刺
  ├─ Major → 启动 Sprint N.5 或纳入下一 Sprint
  └─ Minor → 记入 backlog
  ↓
修复完成 → 复验 → 通过 → 推进 Sprint N+1
```

---

## 问题分级标准

### Critical（阻塞推进）
- 安全漏洞：硬编码密钥、CORS `*`、鉴权失效、SQL 注入
- 数据风险：数据丢失、脏数据、并发冲突
- 功能阻断：核心流程跑不通、关键 API 不可用
- 架构缺陷：跨服务字段不一致导致消息消费失败

### Major（影响质量但不阻塞）
- 缺测试覆盖（核心模块 < 60%）
- 文档与实现不符（声明 28 组件实际 1 个）
- 性能未达标（API P95 > 500ms）
- 配置 Bean 缺失（声明依赖但无 @Bean）
- 跨服务命名不一致（前端 `xiaohongshu` 后端 `xhs`）

### Minor（可记入 backlog）
- 代码风格小问题
- 注释缺失
- 日志级别不当
- 非 P0 功能的小 bug

---

## 审验 4 维度详解

### 1. 编译验证
```bash
# 后端
mvn compile -pl <service> -am
# 前端
pnpm --filter <pkg> build
```
- 所有模块 BUILD SUCCESS
- TypeScript 0 错误
- 检查废弃 API 使用（如 Spring Boot 3.x 的 `SecurityProperties.BASIC_SECURITY_ORDER` 已删除，应用 `BASIC_AUTH_ORDER`）

### 2. 测试执行
```bash
mvn test -pl <service>
pnpm --filter <pkg> test
```
- 测试通过率 100%
- 核心模块覆盖率 ≥ 60%
- 集成测试覆盖关键路径

### 3. 安全清单复验
对照 `scripts/security-checklist.md` 逐项检查：
- [ ] JWT Secret 环境变量（非硬编码）
- [ ] AES Key 环境变量 + 32 字节
- [ ] CORS 精确白名单（非 `*`）
- [ ] SecurityConfig `.authenticated()`（非 `permitAll()`）
- [ ] admin 密码环境变量
- [ ] bcrypt 加盐
- [ ] 敏感数据脱敏
- [ ] 限流配置
- [ ] 审计日志
- [ ] Fail-Fast 启动校验

### 4. 跨服务一致性验证
详见 [cross-service-check.md](cross-service-check.md)：
- API 路径前缀统一
- 状态枚举一致
- MQ 消息体字段一致
- 错误码规则一致
- 平台标识/订阅等级等枚举值跨前后端一致

---

## 修复 Sprint（Sprint N.5）

### 启动条件
- 审验发现 ≥ 1 项 Critical
- 或 ≥ 3 项 Major

### 执行方式
1. 列出所有问题（编号 + 分级 + 修复方案）
2. 并行修复（后端问题 + 前端问题 分派不同 subagent）
3. 修复后立即复验
4. 复验通过才能推进下一 Sprint

### 真实案例：MediaForge Sprint 1.5

Sprint 1 审验发现：
- **5 项 Critical**：JWT/AES 默认密钥、CORS `*`、admin 密码硬编码、SecurityConfig `permitAll()`
- **10 项 Major**：SELECT *、init.sql 缺角色、平台标识不一致、零测试、RabbitMQ/MinIO Config 缺失等

启动 Sprint 1.5 修复冲刺：
- 并行 2 个 subagent（后端修复 + 前端修复）
- 后端修改 14 文件（5 Critical + 4 Major）
- 前端修改 5 文件（平台标识对齐）
- 修复中又发现 1 项 Critical（AES 默认密钥仅 30 字节，需 32 字节）
- 复验通过后推进 Sprint 2

**价值**：避免了带着 5 项安全漏洞推进 6 个 Sprint，防止后期返工。

---

## 审验交付物

每个 Sprint 完成后必须输出：

### 1. 完成报告
```markdown
# Sprint N 完成报告

## 交付物审验
| 模块 | 文件数 | 测试用例 | 验证状态 |
|------|--------|---------|---------|
| xxx  | xx     | xx      | ✅ BUILD SUCCESS |

## 累计交付
| 维度 | 数量 |
|------|------|
| 新增 Java | xx |
| 新增前端 | xx |
| 新增测试 | xx |
```

### 2. 审验中发现并修复的问题
```markdown
## 审验中处理的问题
| # | 问题 | 处理 |
|---|------|------|
| 1 | xxx  | 修复说明 |
```

### 3. 跨服务一致性验证结果
```markdown
## 跨服务一致性验证
| 审验项 | 结果 |
|--------|------|
| API 路径 | ✅ |
| 状态枚举 | ✅ |
```

### 4. 下一 Sprint 任务明细
```markdown
## 下一步任务明细：Sprint N+1

### 任务分组
#### 第一组：xxx
| 编号 | 任务 | 优先级 |
|------|------|--------|
| P4-XXX-001 | xxx | P0 |

### 关键约束
1. xxx
2. xxx

### 验收标准
- [ ] xxx
```

---

## 反模式（禁止）

- ❌ 跳过审验直接推进
- ❌ 发现 Critical 但"先推进下个 Sprint 再修"
- ❌ 审验只看编译不跑测试
- ❌ 不验证跨服务一致性
- ❌ 修复后不复验就推进
- ❌ 审验报告只写"通过"不列具体问题
