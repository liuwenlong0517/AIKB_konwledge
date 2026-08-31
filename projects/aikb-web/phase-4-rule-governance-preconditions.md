---
id: aikb:projects:aikb-web:phase-4-rule-governance-preconditions
type: project-memory
status: verified
tags: [aikb-web, webui, phase-4, rule-governance, atomic-write, rollback, audit, security, windows]
applicable_versions: AIKB WebUI phase 4A first release acceptance completed on Windows
last_verified: 2026-08-30
review_when: 阶段4A开始任务分发、可写规则白名单变化、规则验证或原子替换方案变化、审计schema变化、安装修复动作准备准入时
supersedes: []
relations:
  - type: implements
    target: aikb:projects:aikb-web:management-webui-implementation-plan
  - type: depends_on
    target: aikb:projects:aikb-web:phase-3-controlled-actions-implementation
---

# AIKB WebUI 阶段 4A 规则治理前置基线

## 结论

阶段 3 的静态动作、任务、SSE、Windows 执行器和审计链路已经稳定，可以进入阶段 4 的前置开发阶段，但不能直接开放高风险写入。阶段 4 拆为 4A 规则治理和 4B 安装修复；第一小版本只建立规则审阅、候选校验、完整差异预览和 `USER_RULES.md` 的受控修改闭环。

本基线是任务分发与验收契约。2026-08-30 已完成波次 0～3，包括隔离临时 Git 仓的 Windows 故障注入、两个独立进程认领竞争、崩溃恢复、UTF-8/CRLF 边界、真实浏览器成功应用流程，以及经用户授权的真实 AIKB checkout `USER_RULES.md` 等价往返。真实往返的任务、变更事务与审计关联成功，恢复后文件哈希回到原始值且双仓 clean。应用只接收服务端变更 ID 与进程内令牌，不产生 Git commit/push；其他规则、知识、安装和修复仍不可写。阶段 4A 第一小版本已满足发布门槛，但该结论不能用于扩大可写规则范围；阶段 4B 已于 2026-08-31 另建独立前置基线，仍未开始实现。完整接口、状态机、失败矩阵和哈希证据位于控制仓 `system/tools/aikb-web/docs/phase-4-preconditions.md`。

## 首批规则边界

- `ENTRY_RULES.md`、`system/rules/AI_RULES.md`、`system/rules/CONTRIBUTING.md`：可审阅，不可修改；
- `system/rules/USER_RULES.md`：唯一可修改目标；
- 浏览器只提交稳定规则 ID，不能提交物理路径、脚本、工作目录或环境；
- 正式知识、`INDEX.md`、Agent 根指令、适配器和 Git 操作继续排除；
- macOS 仍只预留公共接口，不实现或声称支持。

首批采取全控制仓干净门槛。预览和应用时只要存在未提交、暂存、未合并、rebase 或 revision 变化就拒绝，避免规则写入与开发中的变更混合。该限制可以在后续有充分并发证据时重新评审，不能静默放宽。

## 预览与确认

规则 API 使用稳定 ID 提供目录、详情、预览和应用。预览必须返回完整统一 diff、候选校验结果、变更 ID、摘要和五分钟单次确认令牌；diff 超过预算时拒绝应用，不能截断后继续确认。

令牌绑定规则 ID、变更 ID、风险、控制仓 revision、原/新内容哈希、diff 哈希、校验器版本和预览摘要。应用前重新核对全部绑定；任何变化稳定返回冲突，不自动合并、重放或覆盖。

## 事务与恢复

候选和短期备份进入 `workspace/runtime/web/rule-changes/`，不进入 Git、任务 JSONL 或审计。事务记录只保存逻辑 ID、哈希、状态、时间和任务关联。应用使用目标同目录临时文件与 `os.replace`；写入前通过候选覆盖模式执行完整结构校验，写入后再次复核，失败时原子恢复。

服务启动时恢复非终态事务：当前文件仍是候选哈希时回滚；若第三方已再次修改，则不得覆盖，标记为需要人工恢复。任务事件只显示校验、应用、复核和回滚等语义步骤，不保存正文、完整 diff、路径或底层异常。

## 审计与失败关闭

阶段 4A 需要兼容 v1-v3 的审计 v4，增加变更 ID、固定资源类型/ID、前后哈希和回滚状态。审计不保存正文、diff、备份、物理路径或 Git status 原文。

规则写入不能沿用普通观察操作的审计 fail-open：应用前无法写入开始事实时拒绝修改；终态审计暂时失败时事务保持非终态并阻止新的规则写入，直到补写或进入人工恢复状态。

## 开发波次

1. 契约与共享验证：静态规则注册表、候选覆盖验证、事务 schema、`source_write` 风险和审计 v4；
2. 只读审阅与预览：规则 API、三栏页面、完整 diff、草稿与确认令牌；
3. 原子应用与恢复：内部 `rule.user.update` 动作、任务关联、锁、替换、复核、回滚和启动恢复；
4. Windows 与浏览器终验：临时仓成功/失败注入、并发冲突、崩溃恢复、UTF-8、深层路由和真实页面流程。

每个波次完成自身契约测试后才能进入下一波。阶段 4A 全部门槛满足前，不扩大可写规则；4B 安装修复必须沿用独立前置契约逐波实现。

## 安装修复阻塞原因

当前 `setup-aikb.ps1` 会组合设置环境、运行测试、安装根指令和 adapters、重建索引并执行 doctor；现有 installer 还会修改用户级 Agent 文件。它们不是单一可预览副作用，不能直接注册为 Web 动作。

4B 已拆分每个 Agent 的固定目标、受管区块、旧值备份、部分失败回滚、权限/占用冲突和真实 handler 验收，并形成独立前置基线。在后续实现与终验完成前，WebUI 仍不提供安装、卸载或修复。

## 验证依据

- 真实 checkout 往返基线为控制仓 `main@39d794c`、知识仓 `main@55165c9` 且双仓 clean；恢复后 `USER_RULES.md` 与保留备份 SHA256 均为 `19c36e42eb891173c08d29a568d1a4a74241ef199f1d10fd33b3f941973847da`；
- 最终正式门禁通过共享核心 59 项（1 项因当前 Windows 符号链接权限跳过）、Web 后端 116 项（13 项显式验收开关用例跳过，相关 Windows 隔离场景已单独运行通过）、前端 42 项，以及 typecheck、lint、生产构建和 190 个 Markdown/50 个知识内容文件的结构校验；
- 阶段 3 终验覆盖 MCP 49 项、后端及真实 Windows 55 项、前端 33 项和浏览器任务中心；
- 2026-08-30 复核现有规则预算/职责校验、静态动作注册、同源保护、审计 v3、原子写入工具和安装脚本真实副作用；
- 写入前分别检索 `status=verified` 与 `status=candidate`；正式区只有总计划和阶段 3 后续边界涉及规则治理，候选区无同主题条目。

## 适用范围

适用于 Windows 本地单用户 AIKB WebUI 阶段 4A 的设计、任务分发与验收。本文确认的发布边界仅为当前 `USER_RULES.md` 受控修改闭环，不代表 `AI_RULES.md`、入口规则、正式知识、安装修复或 macOS 已开放。

## 治理兼容补记（2026-09-01）

规则注册表继续以共享核心的 `recommended_chars`/`max_chars` 双阈值为准：前者超出时警告，后者超出时拒绝。Web 仍只有 `USER_RULES.md` 进入预览、完整 diff 和确认应用闭环，`AI_RULES.md`、`CONTRIBUTING.md` 与入口规则只读。知识治理 v2 为 opt-in，legacy 条目仍可索引；SessionStart 的 review summary 只返回有限只读摘要，当前 WebUI 没有 candidate review 管理入口。Working State 的 owner/author、exact session 与 foreign task 边界见[所有权与知识治理兼容契约](governance-ownership-compatibility.md)。
