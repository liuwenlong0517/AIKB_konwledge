---
id: aikb:projects:aikb-web:phase-2-runtime-audit
type: project-memory
status: verified
governance_version: 2
change_class: factual-update
authority: "控制仓提交 d00425efc033950b76710272c4efb2030edfb5a0 的归档 Working State 读模型、Web API/UI 与定向测试；用户于 2026-09-02 审查通过"
preparer: codex
reviewer: user-liuwenlong
reviewed_at: 2026-09-02
approval_status: not-required
tags: [aikb-web, webui, fastapi, react, work-state, checkpoint, audit, read-only, windows, macos]
applicable_versions: AIKB WebUI 0.1.0 phase 2 baseline plus phase 5 runtime history enhancement at control commit d00425efc033950b76710272c4efb2030edfb5a0
last_verified: 2026-09-02
review_when: 修改活动或归档 Working State schema、生命周期状态、Web 安全投影、分页筛选、降级语义、监听地址或平台适配器时
supersedes: []
duplicate_check_statuses: [verified, candidate, deprecated]
evidence:
  - kind: commit
    ref: "d00425efc033950b76710272c4efb2030edfb5a0"
    result: "新增独立归档 Working State 读模型、Gateway/API、活动/历史页签和历史检查点深链，并保持只读与路径脱敏边界。"
    date: 2026-09-02
  - kind: test
    ref: "system/tools/aikb-web/backend/tests/test_runtime_history.py; system/tools/aikb-web/backend/tests/test_runtime_audit_api.py; system/tools/aikb-web/frontend/tests/RuntimePage.test.tsx"
    result: "主会话定向后端测试包含运行历史契约并通过；RuntimePage 5 项通过，生产构建和定向 ESLint 通过。"
    date: 2026-09-02
  - kind: command
    ref: "python -m unittest backend.tests.test_manuals_api backend.tests.test_runtime_history backend.tests.test_runtime_audit_api backend.tests.test_phase4b_maintenance_readonly (cwd=system/tools/aikb-web)"
    result: "共 24 项通过，1 项按环境条件跳过。"
    date: 2026-09-02
  - kind: user-confirmation
    ref: "当前任务用户消息：我已经审查通过，提交并更新知识"
    result: "用户确认本轮 WebUI 实现审查通过并授权提交与更新知识。"
    date: 2026-09-02
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

- 运行状态把 `planned`、`active`、`blocked` 活动任务与 `completed`、`abandoned`、`superseded` 历史归档建模为两个独立只读接口；归档列表同时校验终态和 `workspace/archive` 事实路径；
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

当前形成 Windows 本机完整只读运行观察面：知识读取、活动任务、历史归档、两类检查点、双仓安全摘要、脱敏审计和系统状态可用。仍不包含 Working State 写入、知识或规则任意修改、任意 Shell、多用户认证、局域网监听和原始诊断下载。

macOS 只保留公共 API、逻辑路径、平台能力与适配器目录位置；没有 macOS 设备实测，因此不得宣称 macOS 已实现或已兼容。后续设备就位后，应在不改变前端公共契约的前提下补充平台实现和真实进程回归。

## 所有权与治理兼容补记（2026-09-01）

Working State Markdown 使用 schema v2，SQLite 派生索引使用独立 schema v3；Markdown 仍是事实源，索引只在版本或指纹变化时重建。Web 运行时保留 `agent` 查询筛选为 latest checkpoint author，同时单独投影 `owner_agent`、`owner_session_id`、`ownership_mode` 和 `participants`，不能把最近作者当作 owner。旧任务无法确定 owner 时显示 `legacy-unbound`，不自动恢复或阻断。

SessionStart/Stop 只接受当前 Agent 与精确 `session_id` 或已登记 participant；`session_id` 在 1..160 个字符内原样保存，拒绝空白/控制字符，Web 的 `owner_session_id` 投影不得截断。旧 `agent+declared-session` 仅在显式升级、同 Agent、完整值经旧算法对应现有 owner 且无 participant 时迁移；旧算法存在碰撞可能，所以还必须有针对具体任务的用户授权。无效 Hook session 降级为无授权路径，不回显、不恢复、不阻断；foreign task 只产生有限提示/审计，不注入任务正文、不阻断当前会话。`session_id` 只是关联标签，不是密码学凭据。SessionStart 的 review summary 是有限只读投影；当前 WebUI 没有 candidate review 管理入口，Web 审计与正式知识读取仍分别遵循既有安全投影。详见[所有权与知识治理兼容契约](governance-ownership-compatibility.md)。

本次完整会话归属修复由控制仓 `90fb17610efc6fe42fb45829fa15d4eacc88030f` 提供；MCP 共享核心共运行 91 项并结果 OK（1 项跳过），Web 后端共运行 257 项并结果 OK（16 项跳过），前端 49 项测试及类型检查、lint、生产构建通过。该修复仅收紧 Working State 归属与 Web 只读投影边界，不新增 WebUI 写入、Shell 或任务管理能力。

## 阶段 5 历史活动补记（2026-09-02）

控制仓 `d00425efc033950b76710272c4efb2030edfb5a0` 增加 `web_archived_work_states`、归档详情和归档检查点读模型，后端使用独立 `/runtime/archived-working-states` 资源，前端以 `/runtime` 与 `/runtime/history` 路由区分活动和历史页签。两类 React Query 只启用当前路由对应请求，历史终态筛选不会误发给活动接口，刷新历史详情或检查点深链仍使用归档 API。

Owner 与 latest checkpoint author 继续分开投影；历史响应不返回归档物理路径或原始 Markdown。当前 Working State 事实源没有独立 `closed_at`，因此历史页把 `updated_at` 明确标作“关闭/最后活动”，不得伪造更精确的关闭时间。真实本机只读冒烟共识别 33 个归档任务，抽取前三项均为 `completed`，响应扫描未包含 `workspace` 物理路径。

## 关联信息

- WebUI 源码与阶段文档：`system/tools/aikb-web/`；
- 共享 Working State 与审计读模型：`system/tools/aikb-mcp/aikb/workstate.py`、`audit.py`；
- 完整验证入口：`system/tools/aikb-web/scripts/validate-aikb-web.ps1`；
- 操作命令：`system/COMMANDS.md`。
