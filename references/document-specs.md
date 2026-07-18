# 文档格式规范 · 8 类交付物标准

> 基于 MediaForge 项目实战沉淀。所有文档统一 HTML 格式（PRD/调研/设计/API/任务明细），SQL 独立文件，MD 用于工程文档。

## 1. 调研报告.html（可选，to C 项目推荐）

### 必含章节
1. 行业背景与趋势（含数据图表）
2. 目标用户画像（前置条件、能力要求、外在准备）
3. 内容生成类型与 AI 应用方式
4. 头部案例分析（如何做好 + 如何盈利）
5. 竞品对比表
6. 机会与建议

### 格式要求
- 深色技术蓝图风（背景 `#0a0e0d` + 霓虹绿 `#00ffa3` + 霓虹蓝 `#00d4ff`）
- 数据可视化（ECharts 或 CSS 图表）
- 响应式布局
- 内嵌 CSS（单文件可分享）

---

## 2. PRD.html（必产）

### 必含 7 章节
1. **产品定位与核心价值**：一句话定位 + 价值主张画布
2. **用户角色与权限矩阵**：角色表 + 权限矩阵（角色×功能）
3. **功能清单**：按 P0/P1/P2 优先级标注，每功能含验收标准
4. **数据模型与实体关系**：ER 图 + 实体字段表
5. **用户流程图**：含异常分支（用 Mermaid 或流程图）
6. **非功能性需求**：性能/安全/可用性/合规（量化指标）
7. **显式非目标（Not-Do List）**：明确不做的功能

### 验收标准
- 每功能有 Given/When/Then 验收条件
- 数据模型字段类型明确
- 权限矩阵覆盖所有角色

---

## 3. UIUX设计规范.html

### 必含章节
1. **设计系统**：色板（主色/辅色/语义色）+ 字体（display + body + mono）+ 间距系统
2. **组件库清单**：声明组件名 + Props + 使用场景
3. **布局规范**：栅格 + 断点 + 容器宽度
4. **交互规范**：动效时长 + 缓动函数 + 状态过渡
5. **主题**：深色/浅色/品牌主题切换

### 关键原则
- 组件清单声明数量必须与实际实现一致（不一致是 Major 问题）
- 颜色用 CSS 变量，便于主题切换
- 字体不用通用字体（Inter/Roboto），选有特色的

---

## 4. 技术架构文档.html

### 必含章节
1. **架构图**：系统全景图 + 服务划分 + 数据流
2. **技术栈**：后端 + 前端 + 中间件 + 部署（含版本号）
3. **模块划分**：每个服务的职责 + 端口 + 依赖
4. **数据流**：关键业务流程的时序图
5. **第三方服务**：依赖清单 + 替代方案
6. **安全设计**：鉴权链 + 加密体系 + 限流策略

---

## 5. Swagger-API文档.html

### 格式要求
- OpenAPI 3.0 规范
- 按服务/域名分组（tag）
- 每接口含：path + method + 请求体 + 响应体 + 错误码 + 示例
- 错误码规范：`{域名}{HTTP状态}{编号}` 如 `USR404001`
- 在线可交互（Swagger UI 风格）

### 后端实现要求
- 每个微服务 1 个 `OpenApiConfig.java`，统一配置
- `@Tag` 分组，`@Operation` 描述，`@Schema` 字段说明

---

## 6. DDL.sql

### 必含内容
1. **建表语句**：`CREATE TABLE`（含字段注释 + 表注释）
2. **索引**：主键 + 唯一索引 + 业务复合索引
3. **外键**：显式声明关系（或应用层维护）
4. **种子数据**：`INSERT INTO` 角色、权限、字典表
5. **幂等**：`CREATE TABLE IF NOT EXISTS` + `INSERT ... ON DUPLICATE KEY UPDATE`

### 命名规范
- 表名：`t_` 前缀 + 蛇形（如 `t_user`、`t_ai_task`）
- 字段：蛇形（如 `created_at`、`user_id`）
- 索引：`idx_{表简}_{字段}` 如 `idx_user_status`

### 必含字段（所有业务表）
```sql
id BIGINT PRIMARY KEY,
created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
deleted_at DATETIME NULL  -- 软删除
```

### Flyway 迁移规范
- 文件名：`V1.0.[n]__{描述}.sql`（如 `V1.0.11__create_content_version_table.sql`）
- 必须幂等可重入（`IF NOT EXISTS` / `ON DUPLICATE KEY`）
- 不可与早期版本冲突（如 V1.0.10 不能再 CREATE 已在 V1.0.2 创建的表，应用 ALTER 补索引）

---

## 7. 评审与Kick-off.html

### 必含章节
1. **产品评审**：PRD 评审记录 + 决议
2. **UI 评审**：设计规范评审 + 决议
3. **技术评审**：架构评审 + 决议
4. **Kick-off**：团队 + 里程碑 + Sprint 计划

---

## 8. 开发任务明细.html

### 任务编号体系
`P[阶段]-[模块代码]-[序号]`
- P4-AIC-001 = P4 阶段 AI Adapter 模块第 1 个任务
- 模块代码：USR（用户）/CRT（创作）/AIC（AI适配器）/CNT（内容）/DST（分发）/ANL（分析）/MON（变现）/ADM（管理）

### 必含字段
- 任务编号
- 任务名称
- 优先级（P0/P1/P2）
- 所属 Sprint
- 验收标准
- 依赖任务

### Sprint 规划
- 每 Sprint 2 周
- Sprint 1.5 = 修复冲刺（非计划 Sprint，用于修复审验发现的问题）

---

## 9. DELIVERABLES.md（项目收尾必产）

### 格式规范
分类列出全部交付物：
1. 文档类（HTML/MD）
2. 后端（按服务分：Entity/DTO/VO/Mapper/Service/Controller/测试）
3. 前端（按端分：View/组件/API Client/Store/路由）
4. 基础设施（Docker/K8s/CI-CD/Flyway）
5. 测试（单测/集成/E2E/安全复验）
6. 脚本（部署/验证/冒烟）

每项标注：文件路径 + 说明。

---

## 10. security-checklist.md（P6 阶段必产）

### 格式规范
按 6 维度分组（认证授权/数据安全/API安全/前端安全/加密/部署安全），每项：
- [x] 或 [ ] 复选框
- 实现位置（文件路径 + 类/方法）
- Sprint 修复记录（如"Sprint 1.5 修复"）

### 复验结论
- 通过项数 / 总项数
- 设计性豁免项（如 CSRF 因 Bearer Token 不适用）
- 生产上线前建议补充项

---

## 文档质量基线

- [ ] 所有 HTML 文档单文件可分享（内嵌 CSS/JS）
- [ ] 所有 SQL 幂等可重入
- [ ] 所有 MD 文档有目录（TOC）
- [ ] 文档与代码实现一致（不一致是 Major 问题，必须修复）
- [ ] 关键决策有 ADR（Architecture Decision Record）记录
