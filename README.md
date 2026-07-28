# Vibe Coding Pro · 生产级研发流程 Skill

> **Conscious Vibe Coding** 研发流程指导。
> 覆盖 **需求 → 设计 → 上下文 → 架构 → 开发 → 联调 → 安全 → 性能 → 部署 → 监控** 10 阶段闭环。

## 核心理念

**黄金法则：AI 生成的每一行代码都视为不可信输入，必须经过人工审查 + 自动化审验后才能进入主干。**

| 层次 | 理念 | 适用场景 | 风险 |
|------|------|----------|------|
| Pure Vibe | 全盘接受 AI 输出 | 一次性原型 | 极高 |
| Casual Vibe | 轻度审查 | 个人 MVP | 中 |
| **Conscious Vibe** | **引导 + 验证 + 审验循环** | **生产级** | **低** |
| Hybrid | AI 脚手架 + 人工核心代码 | 企业级 / 安全敏感 | 极低 |

## 区别于通用 vibe-coding-workflow 的 3 大机制

1. **Sprint 审验-修复循环**：每个 Sprint 完成后必须审验，发现 Critical/Major 问题立即进入修复 Sprint（Sprint N.5），复验通过才能推进。
2. **AI 代码不可信输入审查**：10 大常见坑 checklist，覆盖 JWT 默认密钥、CORS `*`、`permitAll()`、跨服务 MQ 消息体字段不一致等真实踩坑。
3. **跨服务一致性验证**：微服务架构下强制验证 API 路径、状态枚举、错误码、DTO 字段、MQ 消息体跨服务一致。

## 10 阶段研发流程

```
P0 需求 → P1 设计 → P2 上下文 → P3 架构 → P4 开发
→ P5 联调 → P6 安全 → P7 性能 → P8 部署 → P9 监控
```

每个阶段都有：**输入 → 执行步骤 → 输出产物（含格式规范）→ 验收标准 → 检查清单**。

| 阶段 | 核心交付 |
|------|----------|
| P0 需求 | `调研报告.html` + `PRD.html` |
| P1 设计 | `UIUX设计规范.html` + `技术架构文档.html` + `Swagger-API文档.html` + `DDL.sql` |
| P2 上下文 | `.trae/rules/` 5 支柱（项目/风格/领域/示例/约束） |
| P3 架构 | Monorepo 骨架 + 公共模块 + docker-compose + Flyway |
| P4 开发 | Entity/DTO/Service/Controller + 测试（覆盖率 ≥ 60%） |
| P5 联调 | API 对齐 + 类型同步 + 端到端验证 |
| P6 安全 | 6 维度安全清单 + Fail-Fast 启动校验 |
| P7 性能 | 后端缓存/索引/查询 + 前端分包/懒加载 |
| P8 部署 | Dockerfile + K8s + CI/CD 8 阶段流水线 |
| P9 监控 | 结构化日志 + 健康检查 + Prometheus + Grafana + 告警 |

## 目录结构

```
vibe-coding-pro-skill/
├── SKILL.md                       # Skill 主入口（中文，10 阶段流程总览）
├── references/                    # 中文详细参考文档
│   ├── ai-code-review.md          # AI 代码不可信输入审查清单
│   ├── cross-service-check.md     # 跨服务一致性验证
│   ├── document-specs.md          # 8 类交付物格式规范
│   └── sprint-review.md           # Sprint 审验-修复循环
├── vibe-coding-pro-skill-en/      # 🇬🇧 English version
│   ├── SKILL.md                   # Skill entry (English)
│   └── references/                # English reference docs
│       ├── ai-code-review.md
│       ├── cross-service-check.md
│       ├── document-specs.md
│       └── sprint-review.md
├── LICENSE
└── README.md
```

## 语言版本

| 语言 | 路径 | 说明 |
|------|------|------|
| 中文（默认） | `SKILL.md` + `references/` | 原版，基于中文项目实战沉淀 |
| English | `vibe-coding-pro-skill-en/` | 英文翻译版，内容与中文版一致 |

## 使用方式

### 适用场景
- 生产级项目（部署到真实环境、服务真实用户）
- 全栈项目（前后端分离 / 微服务 / 多端）
- AI 协作开发（Claude / GPT / Cursor / Trae 等）

### 不适用
一次性脚本、个人玩具项目、对安全无要求的 demo。

### 接入方式

1. **新项目**：从 P0 开始，逐阶段推进，每阶段完成向用户确认。
2. **已有项目**：诊断当前阶段，从该阶段继续。
3. **问题排查**：跳转到对应阶段的检查清单诊断。
4. **里程碑规划**：参考里程碑模板，按项目规模裁剪。

### 交互原则
- 每阶段完成后汇报，等待用户确认再继续
- 技术选型分歧时提供对比表让用户决策
- 发现风险主动提示，不盲目执行
- 用表格和清单代替长段落
- 每个 Sprint 完成必须审验，输出下一 Sprint 任务明细

## 里程碑规划模板

| 里程碑 | 阶段 | 核心交付 | 验收 |
|--------|------|----------|------|
| M0 | P0-P1 | 调研 + PRD + 设计文档 | 文档评审 |
| M1 | P2-P3 | 上下文 + 项目骨架 | `docker compose up` 可用 |
| M2 | P4 | 核心模块 CRUD | API 可调通 |
| M3 | P4-P5 | 联调 + 功能完善 | 端到端走通 |
| M4 | P6-P7 | 安全 + 性能 | 清单全过 |
| M5 | P8-P9 | 部署 + 监控 | 生产可访问 |

## 核心避坑指南

1. 不要让 AI 一次性生成大模块，分小步每次验证
2. 上下文规则文件 < 1000 字，否则 AI 注意力分散
3. 长会话会丢失上下文，关键约定写入 `.trae/rules/`
4. AI 代码"看起来对但实际有坑"，详见 `references/ai-code-review.md`
5. 第三方服务优先免费/开源（OSS→MinIO、短信→SMTP、地图→OSM）
6. 数据库迁移禁用 `synchronize: true`，用 Flyway migration 脚本
7. 敏感信息只放 `.env`（gitignore），`.env.example` 作模板
8. Fail-Fast：JWT/AES/DB 密码启动时校验，弱密钥直接抛异常阻止启动

## License

[MIT](./LICENSE) © 2026 breezeggubmag357
