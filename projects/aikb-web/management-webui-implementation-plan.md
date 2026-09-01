---
id: aikb:projects:aikb-web:management-webui-implementation-plan
type: project-memory
status: verified
tags: [aikb-web, webui, fastapi, react, knowledge-management, control-plane, audit, windows, macos]
applicable_versions: AIKB WebUI Windows first release complete through phase 4B; phase 5 is feedback-driven Windows continuous optimization
last_verified: 2026-09-01
review_when: WebUI 开始阶段开发、控制动作边界变化、AIKB 事实源或目录边界变化、macOS 设备就位并准备实现时
supersedes: []
relations:
  - type: related_to
    target: aikb:projects:aikb-web:phase-1-read-only-mvp
  - type: related_to
    target: aikb:projects:aikb-web:phase-4b-install-repair-preconditions
---

# AIKB 管理 WebUI 分阶段建设计划

## 背景

AIKB 需要一套本地管理 WebUI，第一阶段用于查看已经落地的知识内容，最终扩展为知识库管理终端。终端需要统一承载知识读取、检查点审阅、受控运维、规则治理和日志审计，但不能破坏 AIKB 现有的事实源、Git 边界和安全约束。

项目初期以 Windows 为正式开发和验证平台。macOS 设备尚未就位，因此现阶段不开发 macOS 实现，也不声明已经兼容 macOS；公共接口、目录和平台能力模型必须预留扩展位置，使后续能够增加 macOS 适配而不重做前端、API、任务和审计体系。

本计划是阶段开发的规划基线。阶段 0 至阶段 3 首批只读动作、阶段 4A 规则治理和阶段 4B Windows 安装修复均已完成；阶段 4B 已通过用户环境、Codex、Claude Code 三个真实目标的独立往返验收。下一阶段不预设大规模功能扩张，改为根据 Windows 版本实际使用反馈持续优化；macOS 在实体设备就位后再进入独立实现阶段。具体边界见关联条目。

## 建设目标

AIKB WebUI 最终定位为 AIKB 的本地管理终端，分为四个领域。

### 知识库读取

- 知识总览大屏；
- 分类、目录和标签浏览；
- 全文搜索和筛选；
- Markdown 阅读和文档内导航；
- 关联知识跳转；
- 来源路径、Git 版本和更新时间展示；
- 不提供正式知识的在线修改或删除能力。

### 运行状态管理

- Working State 总览；
- 检查点列表和详情；
- 工作目标、当前进度、下一步和阻塞项展示；
- 控制仓和知识仓状态展示；
- 运行异常和能力缺失提示。

### 控制层管理

- 触发预定义的校验、诊断、安装和审计动作；
- 查看任务执行进度、标准输出、错误输出和结构化结果；
- 支持超时、取消、并发限制和风险确认；
- 不提供任意 Shell 命令、脚本路径、工作目录或环境变量输入能力。

### 规则和审计治理

- 规则文档浏览、审阅和后续的受控修改；
- 修改前后差异预览和保存前校验；
- 审计事件检索、操作执行历史和失败原因追踪；
- 保持 JSONL 审计事实源与可重建页面视图的边界。

## 总体架构

```text
浏览器
  │
  │ HTTP / SSE
  ▼
aikb-web 前端
  │
  ▼
aikb-web FastAPI 后端
  ├─ 知识查询服务
  ├─ 检查点服务
  ├─ 规则治理服务
  ├─ 审计查询服务
  ├─ 受控任务服务
  └─ 平台能力注册中心
       ├─ Windows 实现：第一阶段
       └─ macOS 实现：只预留扩展位置
  │
  ▼
AIKB 共享 Python 核心
  ├─ Markdown / Git 事实源
  ├─ SQLite / FTS 派生索引
  ├─ Working State
  └─ JSONL 审计事实源
```

架构必须遵守以下边界：

- 浏览器只能调用后端 API，不能直接访问本地文件、SQLite 或执行脚本；
- Web 后端复用 `system/tools/aikb-mcp/aikb/` 的共享能力，不复制知识读取、索引、Working State 和审计逻辑；
- Markdown、Git 和工作状态文件继续作为各自领域的事实源，SQLite、统计结果和页面缓存均为可重建派生数据；
- 控制仓与独立知识仓保持分离，WebUI 不能把两个 Git 根合并为一个工作区；
- WebUI 运行数据进入 `workspace/runtime/web/`，审计数据继续进入 `workspace/audit/`，不得写入源码目录或正式知识目录；
- 所有有副作用的动作必须经过动作注册、参数校验、风险确认、受控执行和审计记录。

## 技术框架

### 前端

- React；
- TypeScript；
- Vite；
- Ant Design；
- TanStack Query；
- React Router；
- Markdown 渲染组件；
- Monaco Editor，仅在规则审阅和修改阶段使用；
- SSE，用于任务输出和状态推送；
- Vitest、Testing Library 和 Playwright。

Ant Design 用于管理终端所需的表格、筛选、树形导航、抽屉、对话框、表单和状态标签，避免重复开发管理后台基础组件。

### 后端

- Python；
- FastAPI；
- Pydantic；
- Uvicorn；
- SQLite/FTS；
- AIKB 现有 Python 核心；
- 封装 `subprocess` 的受控任务执行器；
- SSE 任务事件流；
- pytest 和 Ruff；
- mypy 可在接口稳定后逐步启用。

普通查询采用 REST JSON，长任务输出采用 SSE。第一阶段不引入 WebSocket、独立消息队列或微服务拆分。

## 统一目录规划

Web 专用前端、后端、配置、脚本、测试和文档统一放在控制仓：

```text
system/tools/aikb-web/
├─ README.md
├─ pyproject.toml
├─ package.json
├─ backend/
│  ├─ aikb_web/
│  │  ├─ main.py
│  │  ├─ api/
│  │  │  ├─ knowledge.py
│  │  │  ├─ search.py
│  │  │  ├─ checkpoints.py
│  │  │  ├─ audit.py
│  │  │  ├─ rules.py
│  │  │  ├─ actions.py
│  │  │  ├─ tasks.py
│  │  │  └─ system.py
│  │  ├─ application/
│  │  ├─ domain/
│  │  ├─ infrastructure/
│  │  └─ platform/
│  │     ├─ base.py
│  │     ├─ detector.py
│  │     ├─ registry.py
│  │     ├─ windows/
│  │     │  ├─ actions.py
│  │     │  ├─ environment.py
│  │     │  ├─ paths.py
│  │     │  └─ process.py
│  │     └─ macos/
│  │        └─ README.md
│  └─ tests/
│     ├─ unit/
│     ├─ contract/
│     ├─ integration/
│     └─ windows/
├─ frontend/
│  ├─ src/
│  │  ├─ app/
│  │  ├─ pages/
│  │  ├─ features/
│  │  │  ├─ dashboard/
│  │  │  ├─ knowledge/
│  │  │  ├─ search/
│  │  │  ├─ checkpoints/
│  │  │  ├─ actions/
│  │  │  ├─ tasks/
│  │  │  ├─ rules/
│  │  │  └─ audit/
│  │  ├─ components/
│  │  ├─ api/
│  │  ├─ hooks/
│  │  ├─ types/
│  │  └─ styles/
│  └─ tests/
├─ config/
│  ├─ actions/
│  │  ├─ common/
│  │  └─ windows/
│  └─ schemas/
├─ scripts/
│  ├─ start-aikb-web.ps1
│  ├─ build-aikb-web.ps1
│  └─ validate-aikb-web.ps1
├─ tests/
│  ├─ fixtures/
│  └─ e2e/
└─ docs/
   ├─ architecture.md
   ├─ api.md
   ├─ security.md
   ├─ action-model.md
   ├─ ui-design.md
   └─ platform-extension.md
```

共享核心继续位于 `system/tools/aikb-mcp/aikb/`。正式规则、Agent adapters、知识仓和运行数据均不迁入 `aikb-web`，以免形成重复事实源或混淆职责。

## macOS 扩展预留

初期只实现以下预留：

- `platform/base.py` 定义平台执行契约；
- 平台模型能够表达 `windows` 和 `macos`；
- 后端提供平台和能力发现接口；
- `platform/macos/README.md` 记录待实现契约；
- `docs/platform-extension.md` 记录接入步骤；
- 契约测试覆盖“不支持平台必须明确返回原因”。

初期不创建未经验证的 `.sh` 安装脚本、macOS 环境变量持久化代码、macOS Hook 生成器、macOS 进程终止实现或声称可用但实际只抛异常的假实现。

检测到 macOS 时应返回明确的能力状态，而不是尝试执行 Windows 脚本：

```json
{
  "platform": "macos",
  "supported": false,
  "reason": "macOS implementation is reserved but not yet available"
}
```

公共 API、数据库和审计记录使用逻辑路径，例如 `content/knowledge/example.md`，不得把盘符、反斜杠或无条件小写路径作为跨平台协议。平台相关的环境变量持久化、Hook 生成、进程树终止和文件系统语义由平台适配器负责。

## 页面信息架构

整体采用桌面管理终端布局：

```text
┌─────────────────────────────────────────────────────────┐
│ 顶栏：AIKB 状态 / 当前仓库 / 索引时间 / 全局搜索 / 设置 │
├──────────────┬──────────────────────────────────────────┤
│ 左侧导航     │ 主内容区                                 │
│ 总览         │ 页面标题、状态和操作区                   │
│ 知识库       │                                          │
│ 检查点       │ 页面主体                                 │
│ 控制中心     │                                          │
│ 规则管理     │                                          │
│ 审计日志     │                                          │
│ 系统状态     │                                          │
└──────────────┴──────────────────────────────────────────┘
```

### 总览页

- 正式知识数量、最近更新时间、索引状态和当前检查点；
- 控制仓、知识仓状态；
- 知识分类分布和最近更新；
- 最近任务、审计事件、异常和待处理事项。

### 知识库页面

采用“知识目录树 / Markdown 阅读区 / 文档信息及关联知识”三栏布局，提供面包屑、文档内目录、代码高亮、来源路径、Git 提交信息和关联知识。第一阶段严格只读。

### 搜索页面

采用“筛选条件 / 结果列表 / 内容预览”布局，可按关键词、分类、标签、文件路径、更新时间、内容状态和仓库来源筛选。结果显示标题、命中片段、来源、时间和打开入口。

### 检查点页面

采用列表和详情抽屉，展示工作目标、状态、最近更新时间、下一步、阻塞项、涉及仓库、检查点内容和相关审计事件。

### 控制中心

动作卡片只展示后端允许的预定义动作。执行详情展示任务摘要、进度、实时输出、结构化结果、错误和审计信息。

### 规则管理

采用“规则目录 / 文本审阅或编辑 / 差异与校验结果”布局。开放修改前必须具备修改前快照、差异预览、结构校验、并发修改检测、人工确认和失败保护。

### 审计日志

支持按时间、事件类型、动作 ID、执行状态、发起来源和风险等级筛选，展示脱敏后的结构化详情。JSONL 继续作为事实源，页面统计为派生视图。

## 受控动作模型

前端只能提交稳定的语义动作 ID 和经过约束的参数，不能提交脚本路径或 Shell 文本：

```yaml
id: validate_structure
title: 结构校验
description: 校验 AIKB 控制仓结构
risk_level: low
supported_platforms:
  - windows
required_capabilities:
  - python
  - git
confirmation:
  required: false
timeout_seconds: 300
concurrency_group: validation
parameters:
  - name: scope
    type: enum
    allowed_values:
      - control
      - knowledge
```

执行链路为：

```text
前端选择动作
  → 后端查询动作注册表
  → 校验平台和能力
  → 校验参数
  → 判断风险等级
  → 必要时二次确认
  → 生成任务 ID
  → 启动受控执行器
  → SSE 推送进度
  → 写入结构化审计
  → 返回结果摘要
```

建议将动作划分为：

| 等级 | 示例 | 控制要求 |
|---|---|---|
| `read` | 状态检查、日志查询 | 直接执行 |
| `low` | 结构校验、索引检查 | 参数白名单 |
| `medium` | 重建索引、安装配置 | 二次确认 |
| `high` | 覆盖规则、清理运行数据 | 差异预览、明确确认、完整审计 |

第一阶段不提供任意 Shell、任意脚本路径、任意工作目录、未注册环境变量注入或无审计后台执行。

## 分阶段实施计划

### 阶段 0：架构契约和工程骨架

目标是建立后续控制层、规则层和 macOS 扩展不需要返工的基础。

实施内容：

- 创建统一 `aikb-web` 子工程；
- 建立前后端基础构建；
- 定义 REST API 版本和统一错误模型；
- 定义逻辑路径、平台能力、动作、任务和审计数据结构；
- 建立 Windows 平台注册机制；
- 建立 macOS 扩展说明，不开发实现；
- 建立基础测试框架、主布局和页面路由。

验收门槛：

- 前端不能直接访问事实源；
- API 不出现硬编码盘符；
- 公共动作模型不包含 Shell 命令字段；
- 平台检测结果可查询；
- macOS 被识别为已预留但未支持，而不是运行失败。

### 阶段 1：知识读取 MVP

实施知识总览、目录树、Markdown 阅读、全文搜索、筛选、文档元数据、Git 来源信息和索引状态。

验收门槛：

- 不允许修改或删除知识；
- 搜索结果与现有索引行为一致；
- Markdown 链接和代码块正确展示；
- 中文路径、空格路径和 UTF-8 内容正确；
- 控制仓和知识仓边界清晰；
- 页面标明内容来源和更新时间。

### 阶段 2：运行状态与审计

实施 Working State、检查点、双仓 Git 状态、审计查询、异常提示、系统健康和平台环境信息。

验收门槛：

- 不猜测缺失的 session 或身份信息；
- JSONL 审计仍为事实源；
- 页面报告可以重建；
- 审计展示遵守脱敏和长度限制；
- Working State 与正式知识明确分区；
- 数据库不可用时能够降级说明。

### 阶段 3：受控校验与任务中心

第一批动作建议包括结构校验、知识索引检查、配置诊断、控制仓状态检查、知识仓状态检查和审计报告生成。

任务分发前置契约已经固定在 [第三阶段受控动作前置基线](phase-3-controlled-actions-preconditions.md)。2026-08-29 已按契约实现结构校验和双仓状态三项只读动作、任务中心、REST/SSE、Windows Job Object 与审计 v3，详见[第三阶段实现基线](phase-3-controlled-actions-implementation.md)。索引检查、配置诊断和审计报告仍未准入。

实施动作注册表、参数 Schema、Windows 执行器、任务状态机、SSE 输出、超时、取消、并发分组、风险确认和审计记录。

验收门槛：

- 未注册动作不能执行；
- 参数不能越过白名单；
- 每次执行都有任务 ID 和审计事件；
- 超时和取消可以收敛子进程；
- 页面刷新后仍可查看任务最终状态；
- 中文输出不出现乱码或 U+FFFD；
- 失败原因不能只依赖退出码。

### 阶段 4：规则治理与高风险动作

在审计和任务边界稳定后，再实施规则修改、差异预览、保存前校验、Git 冲突检查、原子写入、安装和修复动作。

阶段 4 拆为 4A 规则治理和 4B 安装修复。4A 的任务分发条件已经固定在[第四阶段规则治理前置基线](phase-4-rule-governance-preconditions.md)：首批四项规则可审阅，但只有 `USER_RULES.md` 可受控修改；控制仓必须全仓干净，应用使用候选覆盖校验、完整 diff、单次令牌、专用事务、跨进程锁、原子替换、回滚和审计 v4。当前已完成波次 0～3，并通过经授权的真实 checkout 等价规则往返终验。

4B 不直接复用一键安装脚本。其独立安全边界和验收结论已经固定在[第四阶段安装与修复前置基线](phase-4b-install-repair-preconditions.md)：首版只允许逐个处理当前 AIKB 用户环境、Codex 和 Claude Code 三个静态目标；浏览器不提交路径或配置正文；跨多个用户文件和环境值的修改使用持久化日志与补偿回滚；卸载和 macOS 不进入首版。波次 0～5 已完成，并通过真实 HKCU、真实 Agent handler、真实浏览器和三个用户目标的完整往返验收。

验收门槛：

- 默认先显示差异再允许写入；
- 工作区冲突或文件已变化时拒绝覆盖；
- 校验失败时不写入正式文件；
- 写入失败时原文件可恢复；
- 不自动提交或推送 Git；
- 不提供正式知识修改能力；
- 所有修改行为进入审计事实源。

### 阶段 5：Windows 实际使用持续优化

阶段 4B 完成后，Windows 版本进入反馈驱动的持续优化阶段。该阶段不提前假设问题或一次性扩大功能范围，只接收能够描述使用场景、期望行为和实际现象的反馈；优先通过当前代码、审计、任务投影和可控复现建立证据，再决定是否修改。

反馈按以下顺序分级：

1. 安全、数据恢复、配置覆盖和审计失真问题；
2. 功能正确性、AIKB 控制层兼容性和 Windows 环境兼容问题；
3. 页面可用性、错误提示、性能和运行稳定性问题；
4. 新功能建议，仅在不突破既有事实源和安全边界时进入后续波次。

每个优化波次必须保持范围可审查：先记录复现条件和影响面，再确定修复边界；实现后执行受影响模块测试和项目一体化门禁。涉及真实 HKCU、Agent 用户配置、规则文件或其他高风险目标时，仍须逐次列明目标、备份和恢复方式并取得明确授权。完成波次后同步控制仓实现、知识仓结论和适用版本，不把未经验证的现场现象直接写成正式知识。

阶段 5 的常规验收门槛：

- 原问题在修复前可复现，或存在足够的代码和审计证据；
- 修复不扩大浏览器可提交的命令、路径、配置正文或任意参数边界；
- 受影响功能具有自动化回归，页面问题补充真实浏览器检查；
- 涉及进程、Handler、编码或用户配置的问题执行真实边界回归；
- MCP、后端、前端、类型检查、Lint、构建和结构校验按影响范围通过；
- 双仓状态、提交、推送和遗留限制分别报告。

### 阶段 6：macOS 实现

本阶段只保留计划，待 macOS 实体设备就位后启动。启动前需要明确处理器架构、目标 Agent、Agent 配置目录和 Hook 行为，并准备独立测试数据。

届时实现 macOS 路径策略、环境变量配置、Agent 配置发现、Hook 启动器、进程组终止、文件权限和符号链接处理、安装诊断以及 macOS 专属集成测试。

验收必须在真实设备上执行生成后的 Handler，覆盖中文路径、空格路径、UTF-8、大小写敏感路径、符号链接、可执行权限、长任务取消和子进程清理。完成验证前，页面持续标记 macOS 为不支持。

## 建议追加能力

以下能力应在架构中留出位置，但不挤入第一版：

- 数据来源追踪：仓库、相对路径、Git commit、内容哈希、索引时间和一致性状态；
- 只读维护模式：禁止控制动作或规则写入；
- 动作预演：展示语义步骤、目标文件和预计修改；
- 搜索解释信息：命中字段、片段、来源、搜索模式和索引版本；
- 状态异常中心：聚合索引过期、双仓异常、环境变量错误、审计失败和 Agent 适配异常。

## 初期明确排除

- macOS 实际实现；
- 多用户和复杂 RBAC；
- 公网部署；
- 移动端专项适配；
- 任意 Shell 终端；
- 正式知识在线编辑和删除；
- 自动 Git commit 或 push；
- 自动晋升候选知识；
- 自建新的全文检索引擎；
- WebSocket；
- Redis、Celery、Kafka 等任务基础设施；
- 微服务拆分；
- 将 Backstage、OliveTin、MkDocs 或 Gollum 作为项目运行依赖；
- 容器化作为唯一运行方式。

## 开源方案吸收边界

- OliveTin：吸收预定义动作、参数表单、安全确认、超时和受控执行思路，不吸收任意 Shell 暴露方式，也不将其作为 AIKB 控制层依赖；
- Backstage：吸收统一门户、功能导航和任务状态组织方式，不引入其庞大的服务目录和插件平台；
- MkDocs：吸收 Markdown 目录导航和阅读体验，不替换 AIKB 的 Markdown/Git/SQLite 事实与派生层；
- Gollum：吸收 Git 历史和差异查看思路，不开放 Wiki 式的正式知识任意修改。

## 发布顺序

```text
架构契约
  ↓
知识只读 MVP
  ↓
检查点与审计
  ↓
低风险控制动作
  ↓
规则修改与安装动作（阶段 4A/4B 已完成）
  ↓
Windows 首版
  ↓
Windows 实际使用持续优化
  ↓
macOS 设备就位
  ↓
macOS 平台实现与真实回归
```

阶段 1 已交付 Windows 本地知识读取 MVP；阶段 2 已增加活动 Working State、检查点、双仓安全摘要和审计查询；阶段 3 首批三项只读动作、任务中心、实时事件和 Windows 进程树收敛也已实现；阶段 4A 已完成规则治理闭环；阶段 4B 已完成用户环境、Codex、Claude Code 安装修复闭环和真实往返验收。当前进入阶段 5，根据 Windows 实际使用反馈按小波次持续优化。知识、其他正式规则与 Git 元数据保持既定只读边界，macOS 继续只保留扩展位置，不声称支持。

## 验证

本计划来自用户需求确认和当前 AIKB 代码边界核对。2026-09-01 阶段 4B 独立验收完成后，重新核对目录、技术框架、安全边界和 macOS 预留策略仍适用，并将下一阶段调整为 Windows 实际使用反馈驱动的持续优化。

React/FastAPI、共享 Python 核心、Markdown/JSONL 事实源和 SQLite 可重建派生层已经在阶段 0～4B 的实现及验收中得到验证。当前 Agent adapters、环境变量安装器和 Hook 命令仍以 Windows 为边界，因此 macOS 只保留扩展契约，不能作为已实现能力。

## 适用范围

适用于 AIKB 管理 WebUI 从方案确认、Windows 首版交付到反馈驱动持续优化的架构和范围控制。阶段 0～4B 已实现并验收，阶段 5 根据 Windows 实际使用逐波优化；本文不代表卸载、任意知识或规则写入、远程管理、macOS 兼容或未经验收的新能力已经开放，也不对阶段 6 工期作出承诺。

当 WebUI 开始实际开发、事实源或仓库边界变化、动作风险模型调整，或者 macOS 设备就位准备开发时，必须复核本计划并按当前代码、Agent 和操作系统行为更新。

## 关联信息

- AIKB 控制仓计划源码位置：`system/tools/aikb-web/`；
- AIKB 共享 Python 核心：`system/tools/aikb-mcp/aikb/`；
- WebUI 运行数据规划位置：`workspace/runtime/web/`；
- AIKB 审计运行面：`workspace/audit/`；
- OliveTin：<https://github.com/OliveTin/OliveTin>；
- Backstage：<https://github.com/backstage/backstage>；
- MkDocs：<https://github.com/mkdocs/mkdocs>；
- Gollum：<https://github.com/gollum/gollum>。
