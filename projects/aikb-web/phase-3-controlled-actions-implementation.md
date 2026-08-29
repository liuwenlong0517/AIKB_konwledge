---
id: aikb:projects:aikb-web:phase-3-controlled-actions-implementation
type: project-memory
status: verified
tags: [aikb-web, webui, phase-3, task-center, sse, windows, job-object, audit, security]
applicable_versions: AIKB WebUI phase 3 first Windows controlled-action release
last_verified: 2026-08-29
review_when: 动作白名单、任务状态机、确认协议、Windows执行器、审计schema、监听范围或平台支持发生变化时
supersedes: []
relations:
  - type: implements
    target: aikb:projects:aikb-web:management-webui-implementation-plan
  - type: depends_on
    target: aikb:projects:aikb-web:phase-3-controlled-actions-preconditions
  - type: depends_on
    target: aikb:projects:aikb-web:phase-2-runtime-audit
---

# AIKB WebUI 阶段 3 受控动作与任务中心实现基线

## 结论

阶段 3 第一小版本已在 Windows 本机实现并验证。WebUI 只开放 `validate.structure`、`repository.status.control`、`repository.status.knowledge` 三项无参数只读动作；知识、规则和 Git 事实源仍不可通过 Web 修改。macOS 只有平台扩展位置，能力保持不支持。

## 实现边界

- 静态 Python 注册表固定动作 ID、空参数 Schema、风险、副作用、超时和并发组，浏览器不能提交命令、脚本路径、工作目录、环境或 stdin；
- 预览生成规范化摘要和五分钟单次令牌，令牌绑定动作、参数、风险与摘要，进程重启后失效，并设置进程内容量上限；
- `events.jsonl` 是任务事实源，`snapshot.json` 是可原子替换和重建的投影；非终态历史任务在启动恢复时进入 `interrupted`；
- REST 提供动作、预览、任务列表、详情和取消，SSE 提供单调事件 ID、断点回放、重置快照、心跳和终态关闭；前端在 SSE 失败时轮询非终态任务；
- 审计 schema v3 增加 `source=web`、任务关联和取消、超时、中断状态，读端保持 v1/v2 兼容；
- 知识阅读页把传入、传出方向和七种关系类型映射为中文，未知值使用中文兜底，不回显内部英文枚举。

## Windows 进程与环境

执行器使用 `STARTUPINFOEX` 创建挂起进程，只继承 NUL stdin、stdout 和 stderr 句柄；进程在恢复前必须关联启用 kill-on-close 的 Job Object。关联失败即拒绝启动，取消、超时和服务关闭统一终止完整 Job。

Git 与 PowerShell 从机器级安装注册表解析绝对程序位置，不信任当前目录或用户 PATH。子环境从空白字典构造，只加入 AIKB 双仓、UTF-8、临时目录、用户目录和 Windows 启动所需变量；受控 PATH 仅含服务端 Python 与系统目录。该约束避免 PowerShell 把缺失的系统盘变量展开为工作目录下的字面量 `%SystemDrive%` 缓存路径。

## 安全投影

任务输出按 UTF-8 单块、单行和总量预算保存；密钥、认证字段、Windows/Unix 路径及 `file:///` URI 在进入事实源前脱敏。结构化结果对 token、secret、authorization、command、argv、环境、诊断和路径键递归清理；任务接口必须显式允许安全结果，其他 API 仍默认丢弃 `result`。

仓库状态动作不转发 Git porcelain 原文或变更文件名，只返回分支、短提交、dirty 和变更数量；结构校验也只返回固定完成提示与安全终态。

变更请求只接受 JSON、`X-AIKB-Request: 1` 和匹配本机 Host/Origin；开发端口只在显式模式允许。服务继续只监听 `127.0.0.1`，不提供局域网、登录、多用户、任意 Shell、安装、清理、索引重建、规则编辑或知识写入。

## 验证

- 隔离测试覆盖静态注册、令牌单次消费与容量、JSONL 重建、状态机、输出预算、脱敏、同源负向、REST/SSE、审计关联和能力状态；
- Windows 真实父、子、孙进程树覆盖正常、取消、超时、两个并发任务取消和服务关闭，均以 PID 探测确认无残留；
- 三项真实动作均成功，执行前后控制仓与知识仓的 branch、revision、status 完全一致，且异常 `%SystemDrive%` 缓存未再生成；
- 前端通过类型检查、Lint、组件测试和生产构建，浏览器覆盖任务中心、深链、终态恢复和关联知识中文显示；
- 正式结构校验覆盖控制仓和独立知识仓。

## 后续边界

`index.inspect`、`config.doctor` 和审计报告生成仍未准入；规则治理、安装修复、索引重建与任意写入属于后续显式阶段。macOS 必须等待实体设备就位后实现独立进程、路径、权限和编码回归，完成前不得声明兼容。
