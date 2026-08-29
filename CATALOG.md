# AIKB 完整知识目录

本文件由知识仓独立维护，集中登记知识仓中的全部内容，便于浏览、查找和写入前去重。Agent 首次接入时不默认加载；新增、移动、重命名或删除知识内容时必须同步维护对应条目。控制仓规则、模板和测试不在本目录登记。

## 内容入口

- [README.md](README.md)：知识内容面的范围、边界和分类入口。

## 通用知识

- [knowledge/README.md](knowledge/README.md)：通用知识的组织和写入规则。
- [knowledge/engineering/README.md](knowledge/engineering/README.md)：通用工程知识分类索引。
  - [架构](knowledge/engineering/architecture/README.md)：架构知识主题索引，暂无具体条目。
  - [编码风格](knowledge/engineering/coding-style/README.md)：编码风格知识主题索引，暂无具体条目。
  - [设计模式](knowledge/engineering/design-patterns/README.md)：设计模式知识主题索引，暂无具体条目。
  - [代码评审](knowledge/engineering/review-guide/README.md)：代码评审知识主题索引，暂无具体条目。
- [knowledge/languages/README.md](knowledge/languages/README.md)：编程语言知识分类索引。
  - [Java](knowledge/languages/java/README.md)：Java 知识主题索引，暂无具体条目。
  - [Python](knowledge/languages/python/README.md)：Python 知识主题索引。
    - [以项目为边界管理 Python 运行时、虚拟环境与依赖](knowledge/languages/python/python-runtime-project-environments.md)：解释器、项目虚拟环境与依赖的边界，以及 uv、PyCharm、VS Code 和版本升级迁移的工作模型。
  - [TypeScript](knowledge/languages/typescript/README.md)：TypeScript 知识主题索引，暂无具体条目。
- [knowledge/frameworks/README.md](knowledge/frameworks/README.md)：框架与平台知识分类索引。
  - [Docker](knowledge/frameworks/docker/README.md)：Docker 知识主题索引，暂无具体条目。
  - [React](knowledge/frameworks/react/README.md)：React 知识主题索引，暂无具体条目。
  - [Spring](knowledge/frameworks/spring/README.md)：Spring 知识主题索引，暂无具体条目。
- [knowledge/tools/README.md](knowledge/tools/README.md)：工程工具知识分类索引。
  - [Git](knowledge/tools/git/README.md)：Git 知识主题索引，暂无具体条目。
  - [IDE](knowledge/tools/ide/README.md)：IDE 知识主题索引，暂无具体条目。
  - [Linux](knowledge/tools/linux/README.md)：Linux 知识主题索引，暂无具体条目。

## 经验沉淀

- [experience/README.md](experience/README.md)：经验内容总入口。
- [候选知识](experience/inbox/README.md)：尚未验证或尚未归类的候选内容，暂无具体条目。
- [解决方案](experience/solutions/README.md)：已经验证的问题解决方案。
  - [统一 VS Code user-data-dir 修复重启后登录会话丢失](experience/solutions/vscode-mixed-user-data-dir-auth-loss.md)：Windows Installer 版混用默认与自定义用户数据目录时，统一启动入口并通过两次冷启动验证登录会话持久化。
  - [PowerShell profile 中实现 Linux 风格列表并区分预测建议与 Tab 补全](experience/solutions/powershell-profile-psreadline-completion.md)：修复 `ll` 的错误参数用法，并通过 PSReadLine 配置区分行内预测建议与命令补全菜单。
- [工程陷阱](experience/pitfalls/README.md)：已经验证的陷阱与规避方式。
  - [Windows Agent Hook 必须分别固定 Shell 解释器与 UTF-8 进程边界](experience/pitfalls/windows-agent-hook-shell-encoding-boundaries.md)：区分 PowerShell 命令被 Git Bash 误解析与 Python/PowerShell 标准流编码冲突，并给出真实 handler 的诊断、修复和回归方式。
  - [PowerShell 脚本中文输出被按 UTF-8 解码成乱码：须在脚本顶部固定输出编码](experience/pitfalls/powershell-script-output-encoding-redirected-capture.md)：维护脚本在重定向/非控制台环境默认输出 GBK，被按 UTF-8 解码的消费者捕获时乱码；固定编码必须放在脚本顶部首个输出之前。
- [工程决策](experience/decisions/README.md)：保留背景和取舍理由的工程决策，暂无具体条目。

## 工作流

- [workflows/README.md](workflows/README.md)：工作流总入口。
- [开发流程](workflows/development.md)：开发任务流程，待定义。
- [调试流程](workflows/debugging.md)：故障和回归问题调试流程，待定义。
- [代码评审流程](workflows/code-review.md)：代码评审流程，待定义。
- [发布流程](workflows/release.md)：发布准备、执行和复盘流程，待定义。

## 项目知识

- [projects/README.md](projects/README.md)：项目级知识的组织规则与项目索引。
- [AIKB Web](projects/aikb-web/README.md)：AIKB 管理 WebUI 的项目知识索引。
  - [AIKB 管理 WebUI 分阶段建设计划](projects/aikb-web/management-webui-implementation-plan.md)：统一前后端目录、页面布局、技术框架、受控动作模型、Windows 首发范围和 macOS 扩展预留方案。
  - [第一阶段只读知识 MVP 实现基线](projects/aikb-web/phase-1-read-only-mvp.md)：共享知识查询、FastAPI/React 只读实现、安全边界和 Windows 验证基线。
  - [第二阶段运行状态与审计实现基线](projects/aikb-web/phase-2-runtime-audit.md)：活动 Working State、检查点、双仓安全摘要、脱敏审计和 Windows 只读管理终端验证基线。
  - [第三阶段受控动作前置基线](projects/aikb-web/phase-3-controlled-actions-preconditions.md)：首批动作准入、任务 JSONL、SSE、Windows 进程树取消、同源确认和审计关联规划基线。
  - [第三阶段受控动作与任务中心实现基线](projects/aikb-web/phase-3-controlled-actions-implementation.md)：三项 Windows 只读动作、任务事实源、REST/SSE、Job Object、审计 v3 和安全验收基线。
- [Local Code RAG](projects/local-code-rag/README.md)：本地代码索引与检索基础设施的项目知识索引。
  - [本地生成式模型退出 RAG 主链路](projects/local-code-rag/local-llm-exclusion.md)：GPU 资源优先投入 embedding、索引和检索，生成与推理交给外部 Agent。
  - [异步索引任务队列与重启状态边界](projects/local-code-rag/async-index-task-queue.md)：索引任务的持久化状态、单工作线程约束、MCP 查询方式和自动恢复边界。
  - [索引一致性恢复、任务可观测性与自动项目注册](projects/local-code-rag/index-integrity-observability-registration.md)：三层索引检查、最小恢复、任务事件指标和只读项目自动登记。
  - [P1：安全上下文、容器监听与多项目检索基准](projects/local-code-rag/p1-safe-context-watcher-retrieval.md)：安全快照读取、任务进度/取消、按需监听、端口迁移和双项目评测结果。
  - [本地检索缓存、上下文预算与 MCP 检索契约](projects/local-code-rag/retrieval-cache-context-mcp.md)：本地 SQLite 缓存、索引 revision 失效、预算化上下文组装、来源排名和 MCP 显式参数。
- [ToolBox](projects/toolbox/README.md)：原生 JavaScript 开发者工具箱的项目知识索引。
  - [项目架构与功能全景](projects/toolbox/project-overview.md)：当前基线的技术形态、扩展机制、20 个工具、外部依赖、运行验证与已知边界。
