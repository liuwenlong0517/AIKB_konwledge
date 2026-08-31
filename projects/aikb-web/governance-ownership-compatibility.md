---
id: aikb:projects:aikb-web:governance-ownership-compatibility
type: project-memory
status: verified
governance_version: 2
change_class: factual-update
authority: "控制仓提交 20c7b38 的共享核心、Web 后端/前端实现与回归测试；由 codex-root 独立复核"
preparer: luna-stage-f
reviewer: codex-root
reviewed_at: 2026-09-01
approval_status: not-required
tags: [aikb-web, webui, working-state, ownership, session, governance, rules, compatibility, read-only]
applicable_versions: "AIKB WebUI 控制仓 20c7b38；Working State Markdown schema v2 / SQLite derived index v3；知识治理 v2"
last_verified: 2026-09-01
review_when: "Working State 或 SQLite 索引 schema、owner/participant 归属、Web agent 筛选、SessionStart/foreign task 行为、规则预算/白名单、知识治理 v2 或 review summary 投影变化时"
supersedes: []
duplicate_check_statuses: [verified, candidate, deprecated]
evidence:
  - kind: commit
    ref: "20c7b38c2d87635c2b141079db1e352dfa7c6316"
    result: "提交本次跨会话归属、规则双阈值、知识治理 v2、Web 安全投影和回归测试修改。"
    date: 2026-09-01
  - kind: file
    ref: "system/tools/aikb-mcp/aikb/workstate.py"
    result: "WORK_METADATA_SCHEMA_VERSION 为 2、WORK_SCHEMA_VERSION 为 3；owner/author/participant 分离，legacy-unbound 不猜测 owner；Web agent 查询使用最新检查点作者字段。"
    date: 2026-09-01
  - kind: file
    ref: "system/tools/aikb-mcp/aikb/hooks.py"
    result: "SessionStart/Stop 只按 owner 或登记 participant 的 Agent+精确 session 过滤；foreign_active_work 只记录不注入、不阻断，缺失 session 时不按 Agent 单独接管。"
    date: 2026-09-01
  - kind: file
    ref: "system/tools/aikb-web/backend/aikb_web/core/rules.py"
    result: "规则注册表从共享核心读取 recommended_chars/max_chars；Web 读模型不暴露物理路径，只有 user 规则可写。"
    date: 2026-09-01
  - kind: file
    ref: "system/tools/aikb-mcp/aikb/indexer.py"
    result: "治理 v2 是显式 opt-in；无 governance_version 的 legacy 条目仍通过索引，审查摘要仅保留有限计数和字段投影。"
    date: 2026-09-01
  - kind: file
    ref: "system/tools/aikb-web/backend/aikb_web/api/v1/rules.py"
    result: "规则 API 只接收稳定 rule_id；预览/确认应用链路不接受路径，现有可写边界仍仅为 USER_RULES.md。"
    date: 2026-09-01
  - kind: file
    ref: "system/tools/aikb-web/frontend/src/pages/RulesPage.tsx"
    result: "页面以 recommended/max 双阈值展示校验预算，并要求完整 diff 与确认后才提交 USER_RULES.md 应用；AI_RULES/CONTRIBUTING 保持只读。"
    date: 2026-09-01
  - kind: test
    ref: "system/tools/aikb-mcp/tests/test_web_runtime_models.py; system/tools/aikb-mcp/tests/test_core.py; system/tools/aikb-mcp/tests/test_knowledge_governance.py"
    result: "Working State owner/author、legacy-unbound、foreign task 与 v2/legacy 知识索引兼容回归通过。"
    date: 2026-09-01
  - kind: command
    ref: "$env:PYTHONPATH='system/tools/aikb-mcp'; python -m unittest system/tools/aikb-mcp/tests/test_web_runtime_models.py system/tools/aikb-mcp/tests/test_core.py system/tools/aikb-mcp/tests/test_knowledge_governance.py"
    result: "通过，52 项测试全部通过（运行时 Web 投影、owner/author、Hook 归属和知识治理 v2/legacy 兼容）。"
    date: 2026-09-01
  - kind: command
    ref: "python -m unittest discover -s tests -v (cwd=system/tools/aikb-mcp)"
    result: "通过，MCP 共享核心 85 项通过、1 项按当前 Windows 权限条件跳过。"
    date: 2026-09-01
relations:
  - type: related_to
    target: aikb:projects:aikb-web:phase-2-runtime-audit
  - type: related_to
    target: aikb:projects:aikb-web:phase-4-rule-governance-preconditions
  - type: related_to
    target: aikb:projects:aikb-web:phase-4b-install-repair-preconditions
---

# AIKB WebUI 所有权与知识治理兼容契约

## 背景

本条目收口控制仓 `20c7b38` 中所有权与规则治理修复对 WebUI 的兼容约束。WebUI 的公共接口必须继续建立在 Markdown/JSONL 事实源和可重建 SQLite 派生层之上；其前置提交 `d9df240` 已包含维护事务、恢复门禁和 Windows 环境执行核心，但维护页面仍按只读 inspect/plan 暴露能力，不能把内部执行核心误报为全部安装修复或公开 apply 已完成。

## 兼容契约

### Working State schema 与所有权

- Working State Markdown 的治理元数据 schema 是 v2；SQLite 是派生索引，索引 schema 独立为 v3。Markdown 仍是事实源，schema 或指纹不匹配时只重建 SQLite，不能反向把索引当事实源。
- `owner_agent`/`owner_session_id` 表示任务所有者，`author_agent`/`author_session_id`/`author_role` 表示最近检查点作者；兼容字段 `agent`、`session_id`、`role` 在 Web 运行时投影中仍表示 latest author，不能据此推断 owner。
- `ownership_mode` 使用 `session-bound`、`shared`、`handed-off` 或 `legacy-unbound`；`participants` 必须登记精确 Agent/会话对。缺失或无法可靠解析 owner 的旧任务归一为 `legacy-unbound`，在显式 claim/handoff 前不自动恢复或阻断。
- Web 的 `agent` 查询筛选仍针对 latest author（索引兼容字段 `agent`），不是 owner 筛选；所有权字段以单独投影提供，前端不得把两者合并展示为“当前所有者”。

### Hook 生命周期与信任边界

- SessionStart/Stop 的任务候选必须通过当前 Agent 与精确 `session_id`，或已登记 participant；session 缺失、不同或无法匹配时不得退化为 Agent-only 接管。
- 其他会话的活动任务属于 `foreign_active_work`：Hook 只写有限审计结果和提示，不注入任务正文、不阻断当前会话，也不自动认领或关闭。
- Hook 观测到的 session ID 是防止误归属的关联标签，不是密码学凭据；同一 Windows 用户下可直接修改工作区或伪造调用的进程不在此边界内。需要强身份时必须另行使用 OS ACL、可信 broker 或进程隔离。

### Web 规则与知识治理

- 规则预算必须同时保留 `recommended_chars`（超出时警告）与 `max_chars`（超出时拒绝）两个阈值；Web 不能把推荐值当硬上限，也不能把最大值降级为提示。
- Web 规则注册表继续只允许 `USER_RULES.md` 进入预览/确认应用闭环；`ENTRY_RULES.md`、`AI_RULES.md`、`CONTRIBUTING.md` 只能只读审阅。浏览器只提交稳定规则 ID，不提交路径、正文或脚本。
- 知识治理 v2 是 opt-in：带 `governance_version: 2` 的条目使用结构化 change class、authority、approval 和 evidence 门禁；没有治理版本的 legacy 条目仍可 validate/rebuild 并被 Web 索引读取，不得伪装成已完成 v2 审查。
- SessionStart 的 review summary 是有限只读投影，只返回固定计数、状态和安全摘要；Web 审计与正式知识读取也继续遵循各自的有限安全投影，均不得返回聊天、隐藏推理、原始 payload、物理路径或密钥。当前 WebUI 没有 candidate review 管理入口，不得把 SessionStart 摘要描述成 Web 审查 API。

## 后续 Web 兼容检查点

每次变更 Working State、规则治理或 Web 读模型时，至少复核以下门槛：

1. Markdown v2 与 SQLite v3 的版本/指纹重建逻辑仍分离，legacy 工作项仍标记 `legacy-unbound`；
2. owner 与 latest author 在 API、筛选、页面文案和恢复逻辑中保持不同语义，`participants` 只接受登记关系；
3. SessionStart/Stop 对 exact session、foreign task 和缺失 session 的行为仍是不注入、不阻断、不 Agent-only 接管；
4. 规则推荐/最大预算、USER_RULES 唯一可写边界、完整 diff/确认和其他规则只读约束有正向与负向测试；
5. v2 正式知识的结构化证据与审批门禁不阻断 legacy 索引，review summary 仍是有限只读投影；
6. 运行时测试、知识治理测试、Web API/前端测试和结构校验均基于当前 checkout 重跑，不沿用旧提交或旧 Working State 的结论。

## 验证

上述事实由当前共享核心、Web 规则读模型、Hook、索引器和对应回归测试交叉核对；本条目不宣称 Codex/Claude Code 安装修复、全部维护目标或公开 apply API 已完成。正式知识查重覆盖 `verified`、`candidate`、`deprecated`：项目现有 `AIKB Web` 7 条 verified、`Working State` 3 条 verified、`规则治理` 4 条 verified，未发现 candidate/deprecated 重复项。

## 适用范围

适用于 AIKB Windows 本地 WebUI 当前及后续维护：运行状态/检查点投影、SessionStart 兼容、规则审阅与受控应用、知识索引及审查摘要。它不扩大 WebUI 写入权限，不改变 Markdown/JSONL 事实源边界，也不表示 macOS、Codex/Claude Code 安装修复或多用户认证已实现。

## 关联信息

- Working State 与 Web 运行时：`system/tools/aikb-mcp/aikb/workstate.py`、`system/tools/aikb-mcp/aikb/hooks.py`；
- 规则注册与治理：`system/tools/aikb-mcp/aikb/rules.py`、`system/tools/aikb-mcp/aikb/indexer.py`、`system/tools/aikb-web/backend/aikb_web/core/rules.py`；
- Web 阶段基线：[第二阶段运行状态与审计实现基线](phase-2-runtime-audit.md)、[第四阶段规则治理前置基线](phase-4-rule-governance-preconditions.md)、[第四阶段安装与修复前置基线](phase-4b-install-repair-preconditions.md)。
