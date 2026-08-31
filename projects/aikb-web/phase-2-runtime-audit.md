---
id: aikb:projects:aikb-web:phase-2-runtime-audit
type: project-memory
status: verified
tags: [aikb-web, webui, fastapi, react, work-state, checkpoint, audit, read-only, windows, macos]
applicable_versions: AIKB WebUI 0.1.0 phase 2
last_verified: 2026-08-29
review_when: 修改 Working State 或审计 schema、Web 安全投影、分页筛选、降级语义、监听地址、平台适配器，或开始阶段 3 受控动作开发时
supersedes: []
relations:
  - type: implements
    target: aikb:projects:aikb-web:management-webui-implementation-plan
  - type: depends_on
    target: aikb:projects:aikb-web:phase-1-read-only-mvp
---

# AIKB WebUI 第二阶段运行状态与审计实现基线

## 背景

第一阶段只读知识 MVP 已形成知识浏览闭环。第二阶段需要把活动 Working State、检查点、双仓摘要和 JSONL 审计接入同一管理终端，同时继续保持 Markdown/JSONL 事实源、派生索引、双 Git 根和只读安全边界。

## 解决方案

共享核心在 `system/tools/aikb-mcp/aikb/` 增加 Web 专用安全读模型。Web 后端只调用共享模型，不自行解析 Working State Markdown 或审计 JSONL：

- 运行状态只查询 `planned`、`active`、`blocked` 活动任务，不开放归档任务搜索；
- 检查点只返回白名单章节，标量和单个列表项最多 4000 字符，列表最多 50 项；
- 仓库只返回逻辑角色、分支、短 revision、可用性和脏状态，不返回物理路径、远端或完整 Git 输出；
- 审计列表、汇总和详情复用 JSONL 事实源，支持时间、Agent、来源、状态、操作、会话标签和项目逻辑 ID 筛选；
- 审计默认按最新活动倒序，同一时间使用调用/事件 ID 稳定排序；缺失 session 保持 `null`，不合成事实；
- Windows 与 Unix 绝对路径、密钥、原始 action/result、payload、traceback 和诊断附件均不进入公共响应，`content/...` 逻辑路径和 URL 保持可读。

FastAPI 新增 `/api/v1/runtime` 与 `/api/v1/audit` 只读路由，成功和错误继续使用统一包络。索引重建通过 `meta.degraded` 与 `index_rebuilt` 表达，索引不可用返回 503；未知 API 返回 JSON 404，写方法返回结构化 405。审计页码上限为 10000，运行状态和检查点页大小上限为 50，审计页大小上限为 100。

React 新增“运行状态”和“审计日志”导航及列表、筛选、分页、详情和深层路由。检查点列表可浏览后续页；审计汇总与列表使用相同筛选条件，并合并两个请求的降级告警。侧栏保持在视口内，页面主体独立滚动；整个阶段没有写入、执行、Git 操作或原始诊断下载入口。

## 验证

2026-08-29 在 Windows 本机完成：

- 共享核心 45 项 `unittest` 全部通过；
- Web 后端 13 项契约、安全与降级测试全部通过；
- 前端 TypeScript、ESLint、17 项 Vitest 和 Vite 生产构建全部通过；
- AIKB 结构校验通过；
- 实际启动脚本固定绑定 `127.0.0.1:8765`；活动任务、检查点、审计列表、审计详情和系统信息均读取真实本机数据；
- 运行状态、检查点和审计详情深层路由直接刷新返回 200，浏览器控制台无错误；
- 真实审计筛选中，`failed + session-start` 的列表总数与汇总计数一致；
- 运行状态、审计和系统响应扫描未命中 Windows/Unix 物理路径、原始 payload、traceback 或诊断路径字段；
- POST 审计路由返回结构化 405，未知 API 返回 JSON 404，审计 `page=10001` 被参数边界拒绝。

Vite 生产包仍有约 1.1 MB 的主 JavaScript chunk 警告，gzip 后约 346 kB。它是后续代码分割和首屏性能优化项，不影响本阶段功能、安全或构建通过。

## 适用范围

当前形成 Windows 本机第一版完整只读管理终端：知识读取、活动任务、检查点、双仓安全摘要、脱敏审计和系统状态可用。仍不包含归档任务搜索、知识或规则修改、任意 Shell、受控动作执行、多用户认证、局域网监听和原始诊断下载。

macOS 只保留公共 API、逻辑路径、平台能力与适配器目录位置；没有 macOS 设备实测，因此不得宣称 macOS 已实现或已兼容。后续设备就位后，应在不改变前端公共契约的前提下补充平台实现和真实进程回归。

## 所有权与治理兼容补记（2026-09-01）

Working State Markdown 使用 schema v2，SQLite 派生索引使用独立 schema v3；Markdown 仍是事实源，索引只在版本或指纹变化时重建。Web 运行时保留 `agent` 查询筛选为 latest checkpoint author，同时单独投影 `owner_agent`、`owner_session_id`、`ownership_mode` 和 `participants`，不能把最近作者当作 owner。旧任务无法确定 owner 时显示 `legacy-unbound`，不自动恢复或阻断。

SessionStart/Stop 只接受当前 Agent 与精确 `session_id` 或已登记 participant；foreign task 只产生有限提示/审计，不注入任务正文、不阻断当前会话。`session_id` 只是关联标签，不是密码学凭据。SessionStart 的 review summary 是有限只读投影；当前 WebUI 没有 candidate review 管理入口，Web 审计与正式知识读取仍分别遵循既有安全投影。详见[所有权与知识治理兼容契约](governance-ownership-compatibility.md)。

## 关联信息

- WebUI 源码与阶段文档：`system/tools/aikb-web/`；
- 共享 Working State 与审计读模型：`system/tools/aikb-mcp/aikb/workstate.py`、`audit.py`；
- 完整验证入口：`system/tools/aikb-web/scripts/validate-aikb-web.ps1`；
- 操作命令：`system/COMMANDS.md`。
