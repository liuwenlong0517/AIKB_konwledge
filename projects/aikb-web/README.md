# AIKB Web 项目知识

本目录记录 AIKB 管理 WebUI（`aikb-web`）的长期架构决策、分阶段实现基线和能力边界。阶段 1、2 与阶段 3 首批受控动作已在 Windows 本机实现并验证；阶段 4A 第一小版本发布门槛已满足；阶段 4B 安装与修复已完成独立前置契约和六波次拆分，但尚未开始实现；macOS 仍只保留扩展位置，不代表已经支持。

## 项目索引

- [AIKB 管理 WebUI 分阶段建设计划](management-webui-implementation-plan.md)：统一前后端目录、页面布局、技术框架、受控动作模型、Windows 首发范围和 macOS 扩展预留方案。
- [第一阶段只读知识 MVP 实现基线](phase-1-read-only-mvp.md)：记录共享知识查询、FastAPI/React 实现、安全边界、Windows 实际启动和完整验证结果。
- [第二阶段运行状态与审计实现基线](phase-2-runtime-audit.md)：记录活动 Working State、检查点、双仓安全摘要、脱敏审计、只读 API/UI 和 Windows 实际验收结果。
- [第三阶段受控动作前置基线](phase-3-controlled-actions-preconditions.md)：冻结动作准入、任务 JSONL、SSE、Windows Job Object、同源确认和审计关联。
- [第三阶段受控动作与任务中心实现基线](phase-3-controlled-actions-implementation.md)：记录三项只读动作、任务中心、安全执行链路、审计 v3 和 Windows 真实验收。
- [第四阶段规则治理前置基线](phase-4-rule-governance-preconditions.md)：冻结首批规则白名单、差异确认、全仓冲突、原子替换、失败恢复、审计 v4 和开发波次。
- [第四阶段安装与修复前置基线](phase-4b-install-repair-preconditions.md)：冻结用户环境、Codex、Claude Code 三个静态目标，以及安全预览、跨文件补偿事务、崩溃恢复和 Windows 真实验收门槛。
