---
id: aikb:projects:aikb-web:phase-3-controlled-actions-preconditions
type: project-memory
status: verified
tags: [aikb-web, webui, phase-3, action-registry, task-center, sse, windows, audit, security]
applicable_versions: AIKB WebUI phase 3 planning baseline and first implemented action set
last_verified: 2026-08-29
review_when: 阶段3开始任务分发、动作白名单变化、任务事实源或SSE契约变化、Windows进程取消方案变化、开放新的写入动作时
supersedes: []
relations:
  - type: implements
    target: aikb:projects:aikb-web:management-webui-implementation-plan
  - type: depends_on
    target: aikb:projects:aikb-web:phase-2-runtime-audit
---

# AIKB WebUI 阶段 3 受控动作前置基线

## 结论

阶段 3 在分发实现任务前，先冻结动作白名单、任务事实源、Windows 取消、SSE、同源保护和审计关联。首批实现已在 2026-08-29 按此基线完成；本文继续作为后续动作准入契约，实际实现与验证见阶段 3 实现基线。

## 首批动作分层

可直接进入第一批实现的只读动作只有：

- `validate.structure`：固定调用结构校验脚本；
- `repository.status.control`：固定读取控制仓 Git 状态；
- `repository.status.knowledge`：固定读取知识仓 Git 状态。

以下动作必须先补专用安全入口：

- `index.inspect`：现有搜索会自动重建索引，必须新增不重建的只读检查；
- `config.doctor`：现有 doctor 会执行 MCP/hook probe 并写审计，必须提供无 probe 观察模式；
- `audit.report.generate`：只允许固定输出到 `workspace/audit/reports/`，属于派生写入并要求风险确认。

索引重建、安装、卸载、修复、清理、规则写入、任意 Shell、任意路径和 Git 写操作不进入阶段 3 第一小版本。

## 稳定契约

- 动作注册表由版本控制下的 Python 代码静态构造，前端只提交 `action_id` 和严格 Schema 参数；
- 外部进程使用固定程序、固定工作目录、参数数组和最小环境白名单，禁止 `shell=True`、字符串拼接命令和继承秘密/代理变量；
- 任务 `events.jsonl` 是事实源，`snapshot.json` 是可重建原子投影；
- 状态覆盖 queued、running、cancelling 及 succeeded、failed、cancelled、timed_out、interrupted 终态；
- 服务重启把遗留非终态任务标为 interrupted，不依据旧 PID 猜测或重新附着；
- Windows 必须使用带 kill-on-close 的 Job Object 收敛完整进程树，关联失败即拒绝执行；
- SSE 使用单调事件 ID、Last-Event-ID 回放、15 秒心跳和终态关闭；
- 变更请求只接受 JSON、同源 Origin/Host、自定义请求头和绑定预览摘要的 5 分钟单次确认令牌；
- 每个任务关联审计 invocation；阶段 3 使用兼容 v1/v2 的审计 v3，增加 `web` 来源、任务关联和取消/超时/中断终态，但不保存任务输出、完整参数、命令行、PID 或原始异常；
- macOS 仍标记为不支持，不创建未经真机验证的执行器。

## 运行预算

服务全局最多同时运行两个任务。仓库状态超时 15 秒，结构校验 120 秒，索引检查 30 秒，配置诊断 90 秒，审计报告 120 秒；同一高资源并发组默认只运行一个任务。

任务输出单块最多 8 KiB、单行最多 4 KiB、单任务持久化总量最多 2 MiB。超限后保留最终结构化结果并标记截断。任务目录沿用 `workspace/runtime/` 默认 30 天保留策略，清理前必须验证不会删除运行中任务和最新投影。

## 分发前门槛

具体开发任务必须能够引用并验证：静态注册表、严格参数模型、同源和确认协议、JSONL 状态机、SSE 回放、Windows 三代进程树取消、审计关联、临时 workspace 测试隔离。详细字段和路由契约位于 `system/tools/aikb-web/docs/phase-3-preconditions.md`。

## 当前边界

阶段 1、2 的知识读取与只读观察能力保持兼容。当前阶段 3 不授权规则或知识修改，也不会自动提交、推送 Git 或扩大监听范围。
