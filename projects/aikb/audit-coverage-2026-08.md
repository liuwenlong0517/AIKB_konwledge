---
id: aikb:projects:aikb:audit-coverage-2026-08
type: project-memory
status: verified
tags: [aikb, audit, governance, rule-closure, runtime, webui, maintenance]
applicable_versions: "AIKB 控制仓/知识仓；审计与验证于 2026-08-27 至 2026-09-01"
last_verified: 2026-08-27
review_when: "AIKB 规则、运行层、WebUI 治理或审计机制发生重大变化，需要复核本覆盖面是否仍准确时"
supersedes: []
relations: []
---

# AIKB 本体审计覆盖面与治理结果（2026-08 审计波次）

## 背景

2026-08-27 至 09-01 对 AIKB 本体（控制仓、知识仓、运行面）执行系统性审计，目标是判断规则与实现是否闭环、Agent 能否钻空子、运行层是否可维护。审计覆盖五个维度，所有发现都经负面用例、生产审计 JSONL 证据或回归测试实证，不是纯文本推理。本条目记录覆盖范围与封堵结果，供后续维护、再审计和回归检查复用，避免重复审计或遗漏。

## 问题

审计揭示的共性结构：AIKB 的约束力呈"两层分化"——格式、结构、边界、索引层有机器强制（schema、validate-structure、aikb validate），Agent 无法钻空子；但语义判断层（"常规/争议"、"验证依据"、"长期价值"、"实质修改"）缺乏客观判据，全凭 Agent 自律，存在绕过确认、伪造验证、自审自提交等逃逸通道。运行层另有维护缺口（runtime 临时数据无清理覆盖），配置面曾出现工作状态索引碰撞导致恢复功能静默失效。

## 解决方案（覆盖图与封堵）

### 维度 1：配置与运行面（hooks/MCP）

- 评估结论：hooks/MCP 配置合理、可正常运行（shell 语义、matcher、超时、MCP stdio 启动器均经官方文档与项目既有知识核对）。
- 发现的唯一功能性故障：重复 `work_id` 同时存在于 `active/` 与 `archive/`，索引重建以归档副本覆盖活动副本，导致 SessionStart 恢复与 Stop 阻断静默失效。
- 封堵：控制仓 `6162c40` —— `rebuild_index` 改为活动优先 + `INSERT OR IGNORE`；`checkpoint` 拒绝复用已归档的显式 `work_id`。
- 验证：`get_work_state` 恢复唯一任务、SessionStart 注入恢复胶囊、Stop 触发 `checkpoint_required` 与 `recursion_skipped`，均有生产审计事件佐证。

### 维度 2：控制层规则闭环

- G1 写入闭环缺 Git 提交步骤（高）→ 封堵 `a6d9fa2`（CONTRIBUTING §6 补第 6-7 步）。
- G2 缺失 front matter 的知识文件被静默跳过、校验假通过（高）→ 封堵 `a6d9fa2`（load_documents 报错）。
- G3 type↔目录一致性无机器校验（中）→ 封堵 `a6d9fa2`（TYPE_DIRECTORY_PREFIXES 映射）。
- G4 局部 README 登记不在校验内（中）→ 封堵 `a6d9fa2`（validate-structure 新增局部 README 链接检查）。
- G5 缺 pitfall/workflow 模板（中）→ 封堵 `a6d9fa2`（新增两模板）。
- G6 supersedes 悬空目标不校验（低）→ 封堵 `a6d9fa2`（格式 + 存在性双校验）。
- G7 规则预算余量极小（AI_RULES 余 16 字符）（低）→ 改善 `a6d9fa2`（余量 16→227）。
- U1 `get_work_state` 省略 project_path 时跨项目返回歧义（中）→ 封堵 `a6d9fa2`（工具描述显式标注）。
- U2 SessionStart 多候选不注入语义未强调（低）→ 封堵 `a6d9fa2`。
- U3 测试桩向真实审计写 invalid_project 噪声（低）→ 封堵 `a6d9fa2`（测试桩补 cwd payload），但暴露运行层 R1。

### 维度 3：知识写入流程（发现→查重→验证→归类→写入→索引→提交→回流）

- 查重默认漏 candidate/Inbox 条目（`search_knowledge` 默认 `verified`）（中）→ 封堵 `6debfdf`（CONTRIBUTING §6.1 强制至少 verified+candidate 各查一次）。
- 新分类 type 选择未显式说明（低）→ 封堵 `6debfdf`（CONTRIBUTING §3 禁止引入新 type）。
- Inbox 晋升 / `review_when` 复核无自动提醒（低）→ 封堵 `6debfdf`（新增 `review_knowledge` 工具、`aikb review` CLI、SessionStart 审查提醒）。
- 写入→检索回流：验证成立，服务按内容指纹自动重建索引，无需手工 rebuild。

### 维度 4：运行层（audit / 工作状态 / 清理 / runtime）

- R1 `clear-workspace.ps1` 从不清理 `runtime/`（遗留测试沙箱）（中）→ 封堵 `c2ffaf7`（新增 `-RuntimeRetentionDays` 默认 30 天、`Preserved` 字段保护 `audit.lock`、新增 `validate-clear-workspace.ps1` 回归）。
- R2 `audit/` 根游离报告（低）→ 封堵（移入 `reports/`）。
- R3 审计查询全量读取（扩展性）、R4 检查点原子写无 fsync、R5 Stop hook 将未跟踪文件计入 dirty、R6 测试桩仍写真实审计 —— 明确接受为取舍项。

### 维度 5：规则约束力（AI_RULES / CONTRIBUTING 语义层）

- L1 "常规 vs 争议"确认门虚设（可绕过用户确认）（高）、L2 验证真实性无法机器校验（可伪造"已验证"）（高）、L3 优先级条款可豁免安全规则（高）、L4 触发词主观（中高）、L5 Inbox 判定权在 Agent（中）、L7 "必要时查 deprecated"可选项（低）→ 封堵 `20c7b38`（加固跨会话归属与知识治理：schema/模板强化、indexer 校验增强、归属绑定）。
- L6 自审即提交、无独立审查门（中）→ 封堵 WebUI 规则治理系列 `8759ebb`/`8cfd9c7`/`8cc65b8`（rule review / preview / controlled application 工作流）。
- M1-M4 观察项：部分缓解（SessionStart 已补晋升提醒），清理节奏仍靠触发。

### 新增治理面（20c7b38 落地）

- 跨会话归属加固：会话绑定（agent+exact-session-required）、session_id 原样传递、不自动恢复其他会话的活动任务。
- 知识治理：knowledge-entry schema 扩展、模板强化、indexer 校验增强、workstate 治理（owner/author 分离）。

## 验证

- 每项发现对应封堵提交，均经回归验证：Python 单元测试（27 项基线）、`validate-structure.ps1`、`validate-adapters.ps1`、`validate-clear-workspace.ps1`、MCP JSON-RPC 实测、临时目录负面用例。
- 生产证据：`workspace/audit/events/*.jsonl` 中可观测到 `resume_context_injected`、`checkpoint_required`、`recursion_skipped` 等真实触发。
- 双环境验证：真实 Windows Terminal 与重定向（Git Bash/CI）各跑一遍适配器测试，暴露 `[Console]::OutputEncoding` 退化为 gb2312 的编码边界问题。

## 适用范围

- 本条目是 AIKB 项目本体的治理记录，覆盖 2026-08-27 至 09-01 的审计波次及其封堵结果。
- 适用于后续维护、再审计、回归检查的起点参照；不替代控制仓规则正文，具体规则以 `system/rules/` 当前版本为准。
- 条目中的提交号来自控制仓 Git 历史，作为定位锚点；提交后的规则或代码变化以当前工作区为准。
- 已知边界：审计未覆盖 WebUI 业务功能细节（见 `projects/aikb-web/` 各阶段基线）；运行时行为依赖 Windows 环境。

## 关联信息

- 控制仓：`system/rules/CONTRIBUTING.md`、`system/rules/AI_RULES.md`、`system/tests/validate-structure.ps1`、`system/tools/clear-workspace.ps1`、`system/tools/aikb-mcp/aikb/{workstate,audit,indexer,hooks}.py`。
- 知识仓：`projects/aikb-web/governance-ownership-compatibility.md`（Working State v2/v3 与归属治理兼容契约）、`projects/aikb-web/phase-4-rule-governance-preconditions.md`。
- 相关经验：`experience/pitfalls/windows-agent-hook-shell-encoding-boundaries.md`（hook Shell 与 UTF-8 边界）。
