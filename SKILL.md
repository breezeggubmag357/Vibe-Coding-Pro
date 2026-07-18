---
name: vibe-coding-pro
description: 基于 MediaForge 全栈项目实战沉淀的 Conscious Vibe Coding 研发流程指导。覆盖需求→设计→上下文→架构→开发→联调→安全→性能→部署→监控 10 阶段闭环，强化 Sprint 审验-修复循环、AI 代码不可信输入审查、跨服务一致性验证三大核心机制。当用户需要从零开始创建生产级项目、规划研发里程碑、系统性推进 vibecoding 开发时使用。核心原则：人主导架构与决策，AI 负责实现与验证，分阶段交付、每步可验收、每 Sprint 必审验。
---

# Vibe Coding Pro · 生产级研发流程

## 适用场景

- **生产级项目**：要部署到真实环境、服务真实用户的项目
- **全栈项目**：前后端分离 / 微服务 / 多端（Web + 管理端 + 移动端）
- **AI 协作开发**：用 AI（Claude/GPT/Cursor/Trae）作为主要实现者

不适用：一次性脚本、个人玩具项目、对安全无要求的 demo。

## 第一原则：Conscious Vibe Coding

**黄金法则：AI 生成的每一行代码都视为不可信输入，必须经过人工审查 + 自动化审验后才能进入主干。**

| 层次 | 理念 | 适用 | 风险 |
|------|------|------|------|
| Pure Vibe | 全盘接受 AI 输出 | 一次性原型 | 极高 |
| Casual Vibe | 轻度审查 | 个人 MVP | 中 |
| **Conscious Vibe** | **引导+验证+审验循环** | **生产级** | **低** |
| Hybrid | AI 脚手架 + 人工核心代码 | 企业级/安全敏感 | 极低 |

### 本 skill 强化的 3 大机制（区别于通用 vibe-coding-workflow）

1. **Sprint 审验-修复循环**：每个 Sprint 完成后必须审验，发现 Critical/Major 问题立即进入修复 Sprint，复验通过才能推进。详见 [references/sprint-review.md](references/sprint-review.md)。
2. **AI 代码不可信输入审查**：10 大常见坑 checklist（含 JWT 默认密钥、CORS `*`、SecurityConfig `permitAll()`、跨服务 MQ 消息体字段不一致等真实踩坑）。详见 [references/ai-code-review.md](references/ai-code-review.md)。
3. **跨服务一致性验证**：微服务架构必须验证 API 路径、状态枚举、错误码、DTO 字段、MQ 消息体跨服务一致。详见 [references/cross-service-check.md](references/cross-service-check.md)。

---

## 10 阶段研发流程总览

```
P0 需求 → P1 设计 → P2 上下文 → P3 架构 → P4 开发
→ P5 联调 → P6 安全 → P7 性能 → P8 部署 → P9 监控
```

每个阶段都有：**输入 → 执行步骤 → 输出产物（含格式规范）→ 验收标准 → 检查清单**。

文档格式规范详见 [references/document-specs.md](references/document-specs.md)。

---

## P0 · 需求分析

### 目标
将模糊想法转化为结构化 PRD，让 AI 能准确实现。

### 执行步骤
1. 问题定义：解决什么问题、目标用户、核心价值
2. 用户故事：每角色写「作为X，我需要Y，以便Z」
3. 验收标准：Given/When/Then 格式
4. 数据模型：核心实体 + 字段类型 + 关系
5. 非功能性需求：性能/安全/扩展性/合规
6. 显式非目标：明确"不做"防 scope creep

### 输出产物（HTML 格式）
- `调研报告.html`：行业调研 + 竞品分析（可选，to C 项目推荐）
- `PRD.html`：产品需求文档（必含 7 章节：定位/角色/功能清单/数据模型/用户流程/非功能/非目标）

### 验收标准
- [ ] 每个功能有验收标准
- [ ] 数据模型字段类型明确
- [ ] 权限矩阵覆盖所有角色
- [ ] 非目标已显式列出

---

## P1 · 技术设计

### 目标
写代码前确定技术栈、架构、目录结构、API 契约、数据库 Schema。

### 执行步骤
1. 技术栈选型（"用熟悉的工具做对的事"）
2. 架构设计（分层/微服务/单体）
3. 目录结构定义到二级目录
4. API 契约（RESTful，含错误码规范）
5. 数据库 Schema（表结构 + 索引策略 + 迁移方案）
6. 第三方服务依赖清单 + 替代方案

### 输出产物（HTML 格式）
- `UIUX设计规范.html`：设计系统（色板/字体/间距/组件库 + 主题）
- `技术架构文档.html`：架构图 + 技术栈 + 模块划分 + 数据流
- `Swagger-API文档.html`：OpenAPI 3.0 全量接口规范
- `DDL.sql`：建表脚本（含索引、外键、种子数据）

### 验收标准
- [ ] 目录结构定义到二级目录
- [ ] API 契约含错误码规范
- [ ] 数据库索引策略已设计
- [ ] 第三方服务有替代方案

---

## P2 · 上下文工程

### 目标
为 AI 创建持久化项目上下文。**优质上下文 + 普通 Prompt 远胜优质 Prompt + 贫乏上下文。**

### 上下文 5 支柱（每个 < 1000 字）

| 支柱 | 文件 | 内容 |
|------|------|------|
| 项目结构 | `.trae/rules/project.md` | 目录布局、命名规范、架构模式 |
| 代码风格 | `.trae/rules/code-style.md` | 命名规则 + 标准模板（Controller/Service/Mapper/组件） |
| 领域知识 | `.trae/rules/domain.md` | 术语表、业务规则、状态机、实体关系 |
| 代码示例 | `.trae/rules/examples.md` | 现有代码片段，让 AI 模式匹配 |
| 约束清单 | `.trae/rules/constraints.md` | 安全红线、性能基线、禁止事项 |

### 关键原则
- 规则文件 **< 1000 字/个**（超长会稀释 AI 注意力）
- 按需加载：基础规则常驻，场景规则按模块加载
- 规则不是越多越好（3 条精简规则 > 1.2 万字规范文档）

### 验收标准
- [ ] 每个规则文件 < 1000 字
- [ ] 已提交版本控制
- [ ] 命名规范有示例
- [ ] 安全约束已写入

---

## P3 · 基础架构搭建

### 目标
搭建项目骨架，让后续模块开发有地基可依。

### 执行步骤
1. 初始化 Monorepo 或多项目结构
2. 后端骨架：框架初始化 + DB/Redis/MQ 连接 + 日志 + 健康检查
3. 前端骨架：路由 + 状态管理 + API 封装 + UI 框架
4. 公共能力：统一响应、异常过滤器、认证守卫、验证管道
5. 种子数据：角色、权限、字典表、测试账号
6. 开发环境：Docker Compose 一键启动

### 输出产物
- Monorepo 目录结构（server/ + 前端 + packages/ + docker/ + database/）
- 公共模块 common/（响应/异常/JWT/加密/CORS/限流/审计/缓存/工具）
- docker-compose.yml（DB + Redis + MQ + 对象存储 + 服务注册）
- .env.example（所有环境变量模板，含 dev 占位值 + 生产强制覆盖注释）
- Flyway 迁移脚本 V1.0.0~V1.0.x（按版本号顺序，幂等可重入）

### 关键设计：Fail-Fast 启动校验
- JWT Secret 启动时校验 ≥ 32 字节，缺失则抛异常阻止启动
- AES Key 校验 base64 + 解码后 32 字节
- 数据库连接失败快速失败，不无限重试

### 验收标准
- [ ] `docker compose up` 能启动所有中间件
- [ ] 健康检查端点返回 200
- [ ] 种子数据可加载
- [ ] Swagger 文档可访问
- [ ] .env.example 含所有必需变量

---

## P4 · 核心模块开发

### 目标
按优先级实现核心功能模块，每个模块独立可验证。

### 任务编号体系
`P[阶段]-[模块代码]-[序号]`，如 `P4-AIC-001` = P4 阶段 AI Adapter 模块第 1 个任务。

### 单模块开发 SOP
```
1. 定义数据实体（Entity）
2. 编写 DTO（创建/更新/查询）
3. 实现 Service 层（业务逻辑）
4. 实现 Controller 层（路由 + 参数校验）
5. 编写单元测试（核心路径覆盖，目标 ≥ 60%）
6. 集成测试（API 级别验证）
7. 人工 Code Review（安全/性能/规范）
```

### Prompt 最佳实践：小步快跑
```
❌「构建完整的用户管理系统」
✅「创建登录表单，含邮箱+密码字段、表单校验、错误状态展示」
```

### 输出产物
- 后端：Entity + DTO + VO + Mapper + Service(Impl) + Controller + application.yml + 测试
- 前端：View + 组件 + API Client + Store + 路由 + 测试
- DDL：新表 Flyway 迁移脚本（V1.0.[n]__描述.sql，幂等）

### 验收标准
- [ ] 每个 API 有 DTO 校验（JSR-303 / class-validator）
- [ ] 敏感操作有权限守卫
- [ ] 列表接口支持分页
- [ ] 数据库查询无 N+1
- [ ] 异常有明确错误码
- [ ] 核心模块单测覆盖率 ≥ 60%

---

## P5 · 前后端联调

### 目标
前端正确消费后端 API，数据流转完整。

### 执行步骤
1. API 对齐：前端 API Client 路径与后端路由完全匹配（统一 `/api/v1/*` 前缀）
2. 类型同步：前端 TS 类型与后端 DTO 一致
3. 错误处理：401/403/500 等场景的前端表现
4. Loading 状态：防重复提交
5. 端到端验证：走通核心用户流程

### 跨服务一致性必查（详见 [references/cross-service-check.md](references/cross-service-check.md)）
- API 路径前缀统一 `/api/v1`
- 状态枚举跨服务一致（如内容状态码 0/1/2/3/4/5）
- MQ 消息体字段跨生产者/消费者一致
- 错误码命名规则一致（{域名}{HTTP}{编号} 如 USR404001）

### 验收标准
- [ ] 核心流程端到端走通
- [ ] Console 无报错
- [ ] 网络请求无 4xx/5xx（业务异常除外）
- [ ] 表单提交有防重复
- [ ] 空状态/错误状态有 UI 反馈

---

## P6 · 安全加固

### 目标
系统性安全审查。**安全是 Vibe Coding 最易被忽略的环节——AI 代码"看起来能跑"但可能含漏洞。**

### 安全检查 6 维度（详见 [references/ai-code-review.md](references/ai-code-review.md)）
1. 认证与授权：JWT Secret 环境变量、Token 过期、RBAC、登录防爆破
2. 数据安全：bcrypt、脱敏、参数化 SQL、输入校验、文件上传限制
3. API 安全：限流、CORS 白名单（禁止 `*`+credentials）、审计日志
4. 前端安全：Token 存储、XSS、CSRF
5. 加密：AES-256-CBC、支付回调验签、HTTPS
6. 部署安全：DB 端口不暴露、Secret 环境变量、镜像最小化、非 root 运行

### Fail-Fast 设计（强制）
- JWT/AES/DB 密码/API Key 启动时校验，缺失或弱密钥直接抛异常阻止启动
- 默认值仅 dev 环境，必须 `${ENV_VAR:dev-default}` 占位，生产强制覆盖

### 输出产物
- `scripts/security-checklist.md`：逐项复验清单（每项标注实现位置 + Sprint 修复记录）

### 验收标准
- [ ] 安全清单全部通过
- [ ] 无硬编码密钥
- [ ] 敏感数据脱敏
- [ ] 接口有权限控制
- [ ] Fail-Fast 机制生效

---

## P7 · 性能优化

### 后端优化清单
| 项 | 方法 | 优先级 |
|----|------|--------|
| Redis 缓存 | TTL 分级（基础数据 1h / 画像 5min / 详情 1min / 看板 10s） | P0 |
| 数据库索引 | 常用查询条件加复合索引 | P0 |
| 查询优化 | 消除 N+1，用 JOIN 或批量查询 | P1 |
| 分页控制 | 强制分页，pageSize 上限 100 | P1 |
| 响应压缩 | gzip/brotli | P2 |
| 连接池 | HikariCP 调优 | P2 |

### 前端优化清单
| 项 | 方法 | 优先级 |
|----|------|--------|
| 代码分割 | 路由懒加载 + CSS code splitting | P0 |
| 构建优化 | terser（drop_console + drop_debugger） | P0 |
| 依赖分包 | Vite manualChunks 拆分大依赖 | P1 |
| 图片懒加载 | `loading="lazy"` | P2 |
| 虚拟列表 | 大列表虚拟滚动 | P2 |

### 验收标准
- [ ] API P95 < 200ms（不含 AI 调用）
- [ ] 首页加载 < 3s
- [ ] Lighthouse > 80
- [ ] 无内存泄漏

---

## P8 · 部署上线

### 输出产物
- 每个服务 1 个多阶段 Dockerfile（构建阶段 + 运行阶段，基于 alpine 镜像）
- docker-compose.yml（全服务编排：中间件 + 后端 + 前端）
- Nginx 配置（SPA 路由回退 + API 代理 + gzip + 静态缓存）
- K8s 清单（Deployment + Service + Ingress + ConfigMap + Secret）
- CI/CD 流水线（.gitlab-ci.yml：lint→test→build→scan→image→push→deploy→verify 8 阶段）
- Flyway 迁移全量验证（顺序执行无冲突）

### 验收标准
- [ ] 所有服务有 Dockerfile
- [ ] `docker compose up` 一键启动
- [ ] .env.example 含所有必需变量
- [ ] 生产不暴露 DB 端口
- [ ] 日志有持久化 Volume
- [ ] 健康检查配置

---

## P9 · 监控告警

### 三层监控
1. **日志层**：结构化 JSON + 按日轮转 + 30 天保留 + 独立 error 日志 + TraceId 贯穿
2. **健康检查层**：`/actuator/health` 含 DB/Redis/MQ/对象存储探针
3. **告警层**：API P95>500ms / 错误率>5% / 服务不可用 / 内存>80%

### 输出产物
- HealthIndicator（DB/Redis/MQ/MinIO 各 1 个）
- Prometheus 指标暴露 + 业务指标（AI/分发/内容/支付）
- Grafana 仪表盘（overview + business）
- 告警规则（Prometheus Alertmanager）

### 验收标准
- [ ] 日志按日轮转，有保留期
- [ ] 请求日志含耗时
- [ ] 健康检查端点可用
- [ ] 错误日志独立文件
- [ ] 日志结构化 JSON

---

## Sprint 审验-修复循环（核心机制）

每个 Sprint 完成后**必须审验**，不可跳过。详见 [references/sprint-review.md](references/sprint-review.md)。

### 审验流程
```
Sprint N 完成
  ↓
审验（编译/测试/安全清单/跨服务一致性）
  ↓
发现 Critical/Major 问题？
  ├─ 是 → 启动 Sprint N.5 修复冲刺 → 复验 → 通过 → 推进 Sprint N+1
  └─ 否 → 推进 Sprint N+1
```

### 问题分级
- **Critical**：阻塞推进（如安全漏洞、数据丢失风险、鉴权失效）
- **Major**：影响质量但不阻塞（如缺测试、文档不符、性能未达标）
- **Minor**：可记入 backlog 后续处理

### 审验交付物
每个 Sprint 完成后输出：
1. 完成报告（模块/文件数/测试用例/验证状态）
2. 审验中发现并修复的问题清单
3. 跨服务一致性验证结果
4. 下一 Sprint 任务明细

---

## 里程碑规划模板

| 里程碑 | 阶段 | 核心交付 | 验收 |
|--------|------|----------|------|
| M0 | P0-P1 | 调研 + PRD + 设计文档 | 文档评审 |
| M1 | P2-P3 | 上下文 + 项目骨架 | docker-compose up 可用 |
| M2 | P4 | 核心模块 CRUD | API 可调通 |
| M3 | P4-P5 | 联调 + 功能完善 | 端到端走通 |
| M4 | P6-P7 | 安全 + 性能 | 清单全过 |
| M5 | P8-P9 | 部署 + 监控 | 生产可访问 |

---

## 通用避坑指南（基于真实项目提炼）

1. **不要让 AI 一次性生成大模块**：分小步，每次验证后再继续
2. **上下文不是越多越好**：规则文件 < 1000 字，否则 AI 注意力分散
3. **长会话会丢失上下文**：关键约定写入 `.trae/rules/`，不依赖对话记忆
4. **AI 代码"看起来对但实际有坑"**：常见陷阱见 [references/ai-code-review.md](references/ai-code-review.md)
5. **第三方服务优先免费/开源**：OSS→MinIO、短信→SMTP、地图→OSM
6. **端口冲突**：启动前检查，Docker Compose 服务间不冲突
7. **数据库迁移不可逆操作要谨慎**：生产用 migration 脚本，禁 `synchronize: true`
8. **环境变量管理**：敏感信息只放 `.env`（gitignore），`.env.example` 作模板
9. **预览兜底**：前端配 Mock 登录兜底，后端不可达时仍可预览 UI
10. **结构化交付物清单**：项目收尾输出 `DELIVERABLES.md`，分类列全部交付物

---

## 使用方式

1. **新项目**：从 P0 开始，逐阶段推进，每阶段完成向用户确认
2. **已有项目**：诊断当前阶段，从该阶段继续
3. **问题排查**：跳转到对应阶段的检查清单诊断
4. **里程碑规划**：参考里程碑模板，按项目规模裁剪

### 交互原则
- 每阶段完成后汇报，等待用户确认再继续
- 技术选型分歧时提供对比表让用户决策
- 发现风险主动提示，不盲目执行
- 用表格和清单代替长段落
- 每个 Sprint 完成必须审验，输出下一 Sprint 任务明细

### 与 vibe-coding-workflow 的区别
- vibe-coding-workflow：通用流程指导，适合快速了解方法论
- **vibe-coding-pro（本 skill）**：基于真实生产项目沉淀，强化 Sprint 审验循环、AI 代码审查、跨服务一致性、文档规范、Fail-Fast 设计，适合生产级项目
