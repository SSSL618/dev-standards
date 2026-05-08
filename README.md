# dev-standards

跨项目通用开发文档、架构规范和迁移指南。

## 文档列表

### 架构
- [Web App 默认部署架构基线](architecture/web-app-architecture-baseline.md) — 默认架构、固定原则、偏离条件

### 部署
- [Vercel + Cloudflare + Firebase 部署手册](deployment/vercel-cloudflare-firebase-runbook.md) — 完整部署流程 + 踩坑记录
- [Web App 部署验收 SOP](deployment/deployment-acceptance-sop.md) — 4 级验收 + 排障分层 + 回滚规则

### Review 标准
- [Review Standards Index](review/README.md) — GitHub 项目提交、提测、验收、增量复核的统一入口
- [GitHub 项目验收与交付 Review 标准](review/acceptance-standard.md) — 结论口径、问题分级、硬门槛、证据要求
- [GitHub 项目提交规范](review/submission-standard.md) — branch、commit、PR、验证证据、closing rule

### 迁移指南
- [Vertex AI 迁移指南](migrations/vertex-ai-migration.md) — 从 AI Studio 迁移到 Vertex Express Mode

### Prompt 工程
- [AI Prompt 设计规范](prompts/prompt-design.md) — 分层上下文、对话窗口优化

---

> 开新项目时让 AI Agent 先读对应文档，避免重复踩坑。
