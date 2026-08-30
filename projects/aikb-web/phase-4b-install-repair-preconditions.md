---
id: aikb:projects:aikb-web:phase-4b-install-repair-preconditions
type: project-memory
status: verified
tags: [aikb-web, webui, phase-4b, installation, repair, transaction, rollback, agent-adapter, windows, macos]
applicable_versions: AIKB WebUI phase 4B precondition baseline 2026-08-31; implementation not started
last_verified: 2026-08-31
review_when: 阶段4B开始任务分发、Agent用户配置格式变化、安装器目标或受管标记变化、环境变量策略变化、卸载准备准入、macOS设备就位时
supersedes: []
relations:
  - type: implements
    target: aikb:projects:aikb-web:management-webui-implementation-plan
  - type: depends_on
    target: aikb:projects:aikb-web:phase-4-rule-governance-preconditions
---

# AIKB WebUI 阶段 4B 安装与修复前置基线

## 结论

阶段 4A 已验证单文件规则治理事务，但阶段 4B 涉及用户环境和多个 Agent 配置文件，不能直接把现有组合安装脚本注册成 Web 动作。2026-08-31 已完成现有 `setup-aikb.ps1`、环境设置、根指令、MCP/hooks 安装、卸载和诊断链路的副作用审计，并冻结阶段 4B 的首版目标、安全边界、补偿事务、Windows 验收门槛和六个开发波次。

本基线只表示已经具备任务分发条件，安装与修复实现尚未开始。完整接口、状态机、失败矩阵和发布门槛位于控制仓 `system/tools/aikb-web/docs/phase-4b-install-repair-preconditions.md`。

## 现有链路为什么不能直接复用

- 一键配置会组合环境写入、测试、根指令、MCP/hooks、索引和诊断，无法形成一次可审阅的精确副作用；
- 多个用户文件按顺序写入，后续失败不会自动恢复先前目标；
- `.aikb-backup` 是首次历史快照，不等于本次事务前备份；
- 两个用户环境变量没有成组写入和失败恢复；
- `doctor.ps1` 会产生审计 probe，不是纯读取预览；
- CLI 允许传入配置路径，Web 不能接受任意路径；
- 现有卸载不覆盖根指令，也没有跨文件事务。

因此 4B 只复用受管标记、解析和生成语义，新增专用 Web 规划器、平台适配器和维护事务执行器，不通过 Shell 包装现有组合脚本。

## 首版固定目标

首版只允许逐个处理三个服务端静态目标：

1. `environment`：把当前 Web 服务已经验证的控制仓和知识仓写入当前 Windows 用户的两个固定 AIKB 环境变量；
2. `agent.codex`：安装或修复 Codex 根指令、AIKB MCP 受管区块和 AIKB hooks；
3. `agent.claude-code`：安装或修复 Claude Code 根指令、带 `AIKB_MANAGED=1` 的 MCP 对象和 AIKB hooks。

浏览器只提交目标 ID、基线指纹和单次确认令牌，不提交路径、环境值、配置正文、命令或脚本。页面不提供“一键全部修复”，也不自动关闭或重启 Agent。

卸载、删除备份、任意路径、索引重建、Git/知识/规则修改、完整 `doctor.ps1` 和 macOS 实现均不进入首版。

## 安全预览

预览必须零副作用，不创建事务、备份、任务、审计 probe 或子进程。用户配置可能含有密钥和私有插件信息，因此页面只显示 AIKB 自有受管片段的结构化差异和逻辑叶子，不返回整份 TOML、JSON、Markdown、绝对路径或非受管内容。

目标状态固定为 `ready`、`missing`、`drifted`、`conflict`、`invalid`、`unsupported` 和 `restart_required`。非受管同名对象、损坏配置、重解析点或配置根越界必须拒绝自动修复。

## 补偿事务与恢复

Windows 不能把多个配置文件和注册表环境值放进一次文件系统原子提交，因此 4B 使用持久化事务日志和相反顺序补偿回滚：

- 首次写入前备份所有目标的原始字节、存在状态、哈希和环境变量的缺失/空值/具体值语义；
- 每完成一个叶子立即刷新事务进度；文件采用同目录临时文件原子替换；
- 两个环境变量作为一个逻辑组写入和回读；
- 任一步或目标专属 probe 失败时恢复全部本次变化；
- 崩溃恢复只覆盖仍等于本事务期望哈希的目标，发现第三方修改立即转 `recovery_required`；
- 备份、正文、环境值和绝对路径永不进入 API、任务或审计；
- 首版不自动删除事务备份，人工恢复事务始终阻止新的维护写入。

维护事务与规则事务共享高风险确认和安全审计原则，但使用独立事实源、全局维护锁和 `user_config_write` 风险类型。

## Windows 与 macOS 边界

Windows 平台适配器负责固定用户配置根、重解析点、环境存储、原子替换、文件占用、广播和真实 handler probe。Codex 与 Claude Code 均要在生成后的真实配置上验证 MCP initialize/tools/list、生命周期事件、中文 UTF-8 和 Stop fail-open。

macOS 当前只保留 `MaintenancePlatformAdapter` 扩展接口并返回 `supported=false`。实体设备就位后再处理配置位置、符号链接、权限、大小写敏感路径、环境传播和真实 Agent 回归；通过前不能声明兼容。

## 开发波次

1. 静态目标、平台 SPI、安全模型、预览/事务 schema 和审计字段；
2. 零副作用 inspect/plan、维护 API、页面状态和受管片段差异；
3. 多目标事务、备份、全局锁、补偿回滚和启动恢复；
4. Windows 用户环境目标及两个变量的成组恢复；
5. Codex 与 Claude Code 安装修复，可按 Agent 并行，但共享事务核心；
6. Windows 多进程、真实浏览器、经逐目标授权的真实用户配置往返和发布终审。

每波完成自身负向测试和主会话安全审查后才能进入下一波。真实用户配置终验必须提前列出准确目标、外部备份、短暂影响和恢复方法，并取得用户对当次写入的明确授权。

## 验证依据

- 前置审计基线：控制仓 `main@1e47fda`、知识仓 `main@139e2df`，双仓 clean；
- 已核对环境设置、根指令、MCP/hooks 安装、卸载、共享配置模块和诊断 probe 的当前实现；
- Codex 与 Claude Code 清单当前都只声明 Windows，且共享 MCP 与四个生命周期能力；
- 正式写入前分别检索 `status=verified` 与 `status=candidate`，未发现独立的阶段 4B 安装修复条目；现有阶段 4A 和总计划应通过关系连接而非重复新增；
- 设计继承阶段 4A 已验证的同源保护、单次令牌、安全任务投影、审计 fail-closed 和第三方修改不覆盖原则。

## 适用范围

适用于 Windows 本地单用户 AIKB WebUI 阶段 4B 的任务分发和后续验收。本文不代表安装、修复、卸载、远程管理或 macOS 已经实现，也不授权在真实用户配置上立即执行写入。
