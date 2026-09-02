# AIKB 完整知识目录

本文件由知识仓独立维护，集中登记知识仓中的全部内容，便于浏览、查找和写入前去重。Agent 首次接入时不默认加载；新增、移动、重命名或删除知识内容时必须同步维护对应条目。控制仓规则、模板和测试不在本目录登记。

## 内容入口

- [README.md](README.md)：知识内容面的职责与边界说明。

## 通用知识

- [knowledge/INDEX.md](knowledge/INDEX.md)：通用知识的组织和写入规则。
- [knowledge/engineering/INDEX.md](knowledge/engineering/INDEX.md)：通用工程知识分类索引。
  - [架构](knowledge/engineering/architecture/INDEX.md)：架构知识主题索引，暂无具体条目。
  - [编码风格](knowledge/engineering/coding-style/INDEX.md)：编码风格知识主题索引，暂无具体条目。
  - [设计模式](knowledge/engineering/design-patterns/INDEX.md)：设计模式知识主题索引，暂无具体条目。
  - [代码评审](knowledge/engineering/review-guide/INDEX.md)：代码评审知识主题索引，暂无具体条目。
- [knowledge/languages/INDEX.md](knowledge/languages/INDEX.md)：编程语言知识分类索引。
  - [Java](knowledge/languages/java/INDEX.md)：Java 知识主题索引，暂无具体条目。
  - [Python](knowledge/languages/python/INDEX.md)：Python 知识主题索引。
    - [以项目为边界管理 Python 运行时、虚拟环境与依赖](knowledge/languages/python/python-runtime-project-environments.md)：解释器、项目虚拟环境与依赖的边界，以及 uv、PyCharm、VS Code 和版本升级迁移的工作模型。
  - [TypeScript](knowledge/languages/typescript/INDEX.md)：TypeScript 知识主题索引，暂无具体条目。
- [knowledge/frameworks/INDEX.md](knowledge/frameworks/INDEX.md)：框架与平台知识分类索引。
  - [Docker](knowledge/frameworks/docker/INDEX.md)：Docker 知识主题索引，暂无具体条目。
  - [React](knowledge/frameworks/react/INDEX.md)：React 知识主题索引，暂无具体条目。
  - [Spring](knowledge/frameworks/spring/INDEX.md)：Spring 知识主题索引，暂无具体条目。
- [knowledge/tools/INDEX.md](knowledge/tools/INDEX.md)：工程工具知识分类索引。
  - [Git](knowledge/tools/git/INDEX.md)：Git 知识主题索引，暂无具体条目。
  - [IDE](knowledge/tools/ide/INDEX.md)：IDE 知识主题索引，暂无具体条目。
  - [Linux](knowledge/tools/linux/INDEX.md)：Linux 知识主题索引，暂无具体条目。

## 候选知识

- [全局 Inbox](inbox/INDEX.md)：尚未验证或尚未归类的统一候选数据来源，暂无具体条目。

## 经验沉淀

- [experience/INDEX.md](experience/INDEX.md)：经验内容总入口。
- [解决方案](experience/solutions/INDEX.md)：已经验证的问题解决方案。
  - [统一 VS Code user-data-dir 修复重启后登录会话丢失](experience/solutions/vscode-mixed-user-data-dir-auth-loss.md)：Windows Installer 版混用默认与自定义用户数据目录时，统一启动入口并通过两次冷启动验证登录会话持久化。
  - [PowerShell profile 中实现 Linux 风格列表并区分预测建议与 Tab 补全](experience/solutions/powershell-profile-psreadline-completion.md)：修复 `ll` 的错误参数用法，并通过 PSReadLine 配置区分行内预测建议与命令补全菜单。
- [工程陷阱](experience/pitfalls/INDEX.md)：已经验证的陷阱与规避方式。
  - [Windows Agent Hook 必须分别固定 Shell 解释器与 UTF-8 进程边界](experience/pitfalls/windows-agent-hook-shell-encoding-boundaries.md)：区分 PowerShell 命令被 Git Bash 误解析与 Python/PowerShell 标准流编码冲突，并给出真实 handler 的诊断、修复和回归方式。
  - [PowerShell 脚本中文输出被按 UTF-8 解码成乱码：须在脚本顶部固定输出编码](experience/pitfalls/powershell-script-output-encoding-redirected-capture.md)：维护脚本在重定向/非控制台环境默认输出 GBK，被按 UTF-8 解码的消费者捕获时乱码；固定编码必须放在脚本顶部首个输出之前。
- [工程决策](experience/decisions/INDEX.md)：保留背景和取舍理由的工程决策，暂无具体条目。

## 工作流

- [workflows/INDEX.md](workflows/INDEX.md)：工作流总入口。
- [开发流程](workflows/development.md)：开发任务流程，待定义。
- [调试流程](workflows/debugging.md)：故障和回归问题调试流程，待定义。
- [代码评审流程](workflows/code-review.md)：代码评审流程，待定义。
- [发布流程](workflows/release.md)：发布准备、执行和复盘流程，待定义。

## 项目知识

- [projects/INDEX.md](projects/INDEX.md)：项目级知识的组织规则与项目索引。
- [AIKB 本体](projects/aikb/INDEX.md)：AIKB 自身控制仓、知识仓与运行面的项目知识索引。
  - [AIKB 本体审计覆盖面与治理结果（2026-08 审计波次）](projects/aikb/audit-coverage-2026-08.md)：五个维度的审计覆盖、发现→封堵→验证映射，以及剩余取舍项。
- [AIKB Web](projects/aikb-web/INDEX.md)：AIKB 管理 WebUI 的项目知识索引。
  - [AIKB 管理 WebUI 分阶段建设计划](projects/aikb-web/management-webui-implementation-plan.md)：统一前后端目录、页面布局、技术框架、受控动作模型、Windows 首发范围、阶段 5 反馈迭代和 macOS 扩展预留方案。
  - [第一阶段只读知识 MVP 实现基线](projects/aikb-web/phase-1-read-only-mvp.md)：共享知识查询、FastAPI/React 只读实现、安全边界和 Windows 验证基线。
  - [第二阶段运行状态与审计实现基线](projects/aikb-web/phase-2-runtime-audit.md)：活动与历史 Working State、检查点、双仓安全摘要、脱敏审计和 Windows 只读管理终端验证基线。
  - [第三阶段受控动作前置基线](projects/aikb-web/phase-3-controlled-actions-preconditions.md)：首批动作准入、任务 JSONL、SSE、Windows 进程树取消、同源确认和审计关联规划基线。
  - [第三阶段受控动作与任务中心实现基线](projects/aikb-web/phase-3-controlled-actions-implementation.md)：三项 Windows 只读动作、任务事实源、REST/SSE、Job Object、审计 v3 和安全验收基线。
  - [第四阶段规则治理前置基线](projects/aikb-web/phase-4-rule-governance-preconditions.md)：首批规则白名单、审计 v4、全仓冲突、完整差异预览、原子替换、恢复和 Windows 终验；阶段 4A 已完成。
  - [第四阶段安装与修复前置基线](projects/aikb-web/phase-4b-install-repair-preconditions.md)：用户环境、Codex、Claude Code 三个静态目标，以及可读漂移语义、安全预览、跨文件补偿事务、崩溃恢复和六个开发波次。
  - [所有权与知识治理兼容契约](projects/aikb-web/governance-ownership-compatibility.md)：Working State v2/v3、owner/author 分离、SessionStart/foreign task 边界、规则治理 v2 和 Web 只读投影兼容检查点。
  - [共享检索服务多词 AND 查询修复](projects/aikb-web/search-multi-term-query-empty-results.md)：`KnowledgeService.search()` 按空白拆词、跨词 AND，并以 LIKE 约束 trigram 短词的共享修复与验证。
  - [WebUI 延后维护项与复核门槛](projects/aikb-web/webui-deferred-maintenance-review.md)：WarningBar、公共依赖 chunk、workspace 清理和审计任意筛选提前停止的延期理由、复核条件与验收门槛。
- [Local Code RAG](projects/local-code-rag/INDEX.md)：本地代码索引与检索基础设施的项目知识索引。
  - [本地生成式模型退出 RAG 主链路](projects/local-code-rag/local-llm-exclusion.md)：GPU 资源优先投入 embedding、索引和检索，生成与推理交给外部 Agent。
  - [异步索引任务队列与重启状态边界](projects/local-code-rag/async-index-task-queue.md)：索引任务的持久化状态、单工作线程约束、MCP 查询方式和自动恢复边界。
  - [索引一致性恢复、任务可观测性与自动项目注册](projects/local-code-rag/index-integrity-observability-registration.md)：三层索引检查、最小恢复、任务事件指标和只读项目自动登记。
  - [P1：安全上下文、容器监听与多项目检索基准](projects/local-code-rag/p1-safe-context-watcher-retrieval.md)：安全快照读取、任务进度/取消、按需监听、端口迁移和双项目评测结果。
  - [本地检索缓存、上下文预算与 MCP 检索契约](projects/local-code-rag/retrieval-cache-context-mcp.md)：本地 SQLite 缓存、索引 revision 失效、预算化上下文组装、来源排名和 MCP 显式参数。
- [ToolBox](projects/toolbox/INDEX.md)：原生 JavaScript 开发者工具箱的项目知识索引。
  - [项目架构与功能全景](projects/toolbox/project-overview.md)：当前基线的技术形态、扩展机制、20 个工具、外部依赖、运行验证与已知边界。
