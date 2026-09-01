# AIKB Web 项目知识

本目录记录 AIKB 管理 WebUI（`aikb-web`）的长期架构决策、分阶段实现基线和能力边界。阶段 1、2 与阶段 3 首批受控动作已在 Windows 本机实现并验证；阶段 4A 第一小版本发布门槛已满足；阶段 4B 已完成静态目标、安全预览、补偿事务、恢复门禁、Windows 用户环境与 Codex/Claude Code 安装修复、公开 apply、任务投影及三个真实用户目标的独立往返验收。阶段 5 首批实际使用优化已增加总览固定手册、可读配置漂移语义和历史 Working State 观察；数据维护仍是独立后续批次。macOS 仍只保留扩展位置，不代表已经支持。

## 项目索引

- [AIKB 管理 WebUI 分阶段建设计划](management-webui-implementation-plan.md)：统一前后端目录、页面布局、技术框架、受控动作模型、Windows 首发范围、阶段 5 反馈迭代和 macOS 扩展预留方案。
- [第一阶段只读知识 MVP 实现基线](phase-1-read-only-mvp.md)：记录共享知识查询、FastAPI/React 实现、安全边界、Windows 实际启动和完整验证结果。
- [第二阶段运行状态与审计实现基线](phase-2-runtime-audit.md)：记录活动与历史 Working State、检查点、双仓安全摘要、脱敏审计、只读 API/UI 和 Windows 验证结果。
- [第三阶段受控动作前置基线](phase-3-controlled-actions-preconditions.md)：冻结动作准入、任务 JSONL、SSE、Windows Job Object、同源确认和审计关联。
- [第三阶段受控动作与任务中心实现基线](phase-3-controlled-actions-implementation.md)：记录三项只读动作、任务中心、安全执行链路、审计 v3 和 Windows 真实验收。
- [第四阶段规则治理前置基线](phase-4-rule-governance-preconditions.md)：冻结首批规则白名单、差异确认、全仓冲突、原子替换、失败恢复、审计 v4 和开发波次。
- [第四阶段安装与修复前置基线](phase-4b-install-repair-preconditions.md)：冻结用户环境、Codex、Claude Code 三个静态目标，以及可读漂移语义、安全预览、跨文件补偿事务、崩溃恢复和 Windows 真实验收门槛。
- [所有权与知识治理兼容契约](governance-ownership-compatibility.md)：记录 Working State v2/v3、owner/author 分离、SessionStart 边界、规则治理 v2 和 Web 只读投影的兼容检查点。
- [共享检索服务多词查询必然返回空](search-multi-term-query-empty-results.md)：记录 `KnowledgeService.search()` 不做查询分词、含空格多词查询返回空的缺陷事实、根因与复现方式，影响 Web 搜索与 MCP `search_knowledge`。
