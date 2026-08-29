---
id: aikb:projects:aikb-web:phase-1-read-only-mvp
type: project-memory
status: verified
tags: [aikb-web, webui, fastapi, react, sqlite, fts, markdown, read-only, windows]
applicable_versions: AIKB WebUI 0.1.0 phase 1
last_verified: 2026-08-29
review_when: 修改知识 API、共享 KnowledgeService、verified 可见性边界、Markdown 渲染、监听地址、前端构建或进入检查点与审计阶段时
supersedes: []
relations:
  - type: implements
    target: aikb:projects:aikb-web:management-webui-implementation-plan
---

# AIKB WebUI 第一阶段只读知识 MVP 实现基线

## 背景

AIKB 管理 WebUI 第一阶段需要在 Windows 本机形成可独立使用的知识读取闭环，同时为后续检查点、审计、受控动作和 macOS 适配保留稳定边界。第一阶段不修改正式知识、不执行 Shell，也不实现 macOS 功能。

## 问题

现有共享核心只提供面向 MCP 上下文预算的搜索和回读能力，缺少 Web 总览、目录和标签查询。MCP 回读还会压缩 Markdown 连续空白并限制在 12000 字符，不能直接用于完整文档页面。Web 前后端如果自行扫描 Markdown 或查询 SQLite，会产生第二套事实逻辑；如果直接暴露物理路径、候选知识或任意状态参数，又会破坏只读和知识治理边界。

## 解决方案

Web 专用前端、后端、配置、脚本、测试和文档统一位于控制仓 `system/tools/aikb-web/`。生产模式由 FastAPI 同时提供 `/api/v1` 和 Vite 构建产物，服务固定绑定 `127.0.0.1`。

共享 `KnowledgeService` 增加：

- `overview()`：返回 verified 总数、类型、标签、逻辑目录树、最近条目和索引状态；
- `list_documents()`：按逻辑路径、类型和标签分页列出 verified 文档；
- `list_tags()`：统计 verified 标签；
- `read_document()`：保留 Markdown 换行和块结构，以 500000 字符作为本地 Web 响应安全上限；原 MCP `read()` 继续压缩正文并保持 12000 字符预算。

FastAPI 提供健康、平台能力、系统摘要、知识总览、目录、标签、搜索和文档接口。成功响应统一使用 `{data, meta}`，错误统一使用 `{error, meta}`。文档只接受 schema 兼容的 `aikb:` 稳定 ID 或 `content/...` 逻辑路径；绝对路径、盘符、空字节和目录穿越被拒绝。搜索、目录、标签和正文固定过滤 `verified`，前端不能指定其他状态。

React 前端提供总览、知识库、搜索、阅读和系统状态五个路由。Markdown 使用 `react-markdown`、GFM 和 `rehype-sanitize`，不启用原始 HTML。检查点、控制中心、规则管理和审计日志在实现前不显示空入口。

平台目录 `backend/aikb_web/platform/` 定义公共状态契约。Windows 声明第一阶段只读能力可用；macOS 只保留目录和扩展说明，能力接口明确返回未支持。

## 验证

2026-08-29 完成以下验证：

- 共享 Python 核心 30 项 `unittest` 全部通过；
- Web 后端 7 项契约和安全测试全部通过；
- 前端 TypeScript 类型检查和 ESLint 通过；
- Vitest 2 个测试文件、6 项测试全部通过；
- Vite 生产构建通过；
- 前端生产依赖使用官方 npm registry 审计，结果为 0 个已知漏洞；
- AIKB 结构校验通过：132 个 Markdown 文件、45 个知识内容文件、2 个 Agent 适配器；
- 真实知识仓总览、目录、搜索、计划文档读取、系统信息和平台能力接口均返回 200，响应未包含控制仓绝对路径；
- Windows 实际启动脚本在 `127.0.0.1:8765` 启动成功，搜索 `WebUI` 命中当前计划，深层页面直接访问返回 200；
- 浏览器检查总览、搜索、目录、阅读和系统状态页面，控制台无错误；计划文档保留 1 个一级标题、16 个二级标题和完整正文结构。

前端构建仍提示主 JavaScript chunk 超过 500 kB，gzip 后约 333 kB；这是本地第一阶段性能优化项，不影响当前功能和构建成功。

## 适用范围

适用于 AIKB WebUI 0.1.0 第一阶段 Windows 本地只读知识能力。当前没有实现检查点、Working State、审计查询、规则修改、Shell 动作、多用户认证、局域网访问或 macOS 运行能力。

自动索引重建只修改 `workspace/db/` 派生数据库，不修改 Markdown/Git 事实源。前端 `node_modules/`、`dist/` 和缓存属于可重建内容，结构校验必须忽略这些第三方或生成目录，但仍完整检查受版本控制的 WebUI Markdown 文档。

## 关联信息

- WebUI 源码：`system/tools/aikb-web/`；
- 共享知识服务：`system/tools/aikb-mcp/aikb/knowledge.py`；
- Windows 构建、启动和验证：`system/tools/aikb-web/scripts/`；
- 完整命令说明：`system/COMMANDS.md`；
- 第一阶段 API、安全、页面和平台扩展说明：`system/tools/aikb-web/docs/`。
