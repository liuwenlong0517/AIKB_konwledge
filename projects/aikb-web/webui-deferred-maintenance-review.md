---
id: aikb:projects:aikb-web:webui-deferred-maintenance-review
type: project-memory
status: verified
governance_version: 2
change_class: factual-update
authority: "用户于 2026-09-02 明确决定暂缓 WarningBar 抽取、公共 chunk 优化、workspace 清理和审计任意筛选提前停止，并要求记录复核条件"
preparer: codex
reviewer: user-liuwenlong
reviewed_at: 2026-09-02
approval_status: not-required
tags: [aikb-web, webui, performance, maintainability, audit, data-maintenance]
applicable_versions: AIKB WebUI at control commits b6fe409b3f824bfc76ad83f317ae08532be1796f and de9a4c3bd6b4b4afd470106aa1ad4270d6188606
last_verified: 2026-09-02
review_when: WarningBar出现第四个消费者或语义分叉、公共chunk超过800kB或首屏性能可测量恶化、数据维护保护注册表和安全计划器落地、审计查询引入索引或游标契约时
supersedes: []
duplicate_check_statuses: [verified, candidate, deprecated]
evidence:
  - kind: command
    ref: "npm run build (cwd=system/tools/aikb-web/frontend)"
    result: "生产构建通过；路由拆分后入口约3.53kB，公共chunk约591.88kB、gzip约190.75kB，仍触发Vite默认500kB提示。"
    date: 2026-09-02
  - kind: command
    ref: "rg -n WarningBar system/tools/aikb-web/frontend/src/pages"
    result: "三处WarningBar为页面内局部实现，当前均消费ApiMeta并展示降级告警，未发现已证实的功能、安全或性能缺陷。"
    date: 2026-09-02
  - kind: user-confirmation
    ref: "当前任务用户消息"
    result: "用户要求WarningBar与chunk占用写入项目知识后视情况修补；workspace无界增长及无时间条件审计查询的任意筛选分页提前停止，等待数据维护功能落地后再评估。"
    date: 2026-09-02
relations:
  - type: related_to
    target: aikb:projects:aikb-web:management-webui-implementation-plan
  - type: related_to
    target: aikb:projects:aikb-web:phase-2-runtime-audit
---

# WebUI 延后维护项与复核门槛

## 当前决策

下列项目在 2026-09-02 的审查中确认存在优化空间，但没有构成当前功能、安全或稳定性阻断，因此不进入本轮修复：

- 三处页面内 `WarningBar` 的重复实现；
- Vite 公共依赖 chunk 超过默认 500 kB 提示；
- workspace 中任务、审计和事务材料的长期清理；
- 没有时间条件时，审计任意筛选分页是否允许提前停止扫描。

延期不等于关闭问题。重新评估时应使用本条目的触发条件和验收门槛，避免仅为了消除静态提示扩大改动面。

## WarningBar

`AuditPage`、`RuntimePage` 和 `TasksPage` 各有一个页面内局部 `WarningBar`。三者当前职责很小，且 `RuntimePage` 存在审计告警文案差异；立即抽取共享组件主要减少重复代码，不会直接改善用户可见行为。

暂不修复的理由：

- 当前只有三个消费者，代码量与分支都有限；
- 没有发现文案、严重度或可见条件不一致导致的缺陷；
- 为统一而抽取需要同时冻结告警展示契约，收益不足以覆盖回归面。

出现以下任一条件时重新评估：增加第四个消费者；告警种类或映射明显增长；各页面的严重度、文案、排序产生分叉；告警需要携带结构化操作、详情链接或统一可访问性行为。届时优先抽取纯展示组件和共享映射，不改变 API 响应结构。

## 公共依赖 chunk

路由级代码分割后，生产入口已经降到约 3.53 kB；公共依赖 chunk 约 591.88 kB，gzip 后约 190.75 kB。Vite 的提示是体积启发式告警，不是构建失败。当前 WebUI 运行在本机回环地址，公共依赖可被浏览器缓存，尚无首屏加载或解析卡顿的测量证据，因此可接受该提示。

不要只提高 `chunkSizeWarningLimit` 来隐藏提示。出现以下任一条件时再优化：公共 chunk 增长到约 800 kB 至 1 MB；冷启动性能测量显示下载、解析或执行成为主要瓶颈；部署范围扩展到远程网络或低性能设备；发现只被少数路由使用的大型依赖被错误并入公共 chunk。优化前先用构建可视化或浏览器性能数据定位依赖，再决定 `manualChunks`、替换依赖或进一步延迟加载。

## 与数据维护能力绑定的延期项

workspace 清理涉及任务记录、审计 JSONL、规则草稿和维护事务材料。它不能以目录年龄或简单数量阈值直接删除，必须先有保护注册表、引用关系判断、安全预览、确认令牌、可审计执行和失败恢复。数据维护功能具备这些边界后，再设计保留期、容量阈值和分批回收策略。

无时间条件的审计任意筛选无法在保持精确 `total` 和稳定分页语义的同时安全提前停止；提前停止会让用户误以为结果已经完整。该项等待数据维护/索引能力落地后重新评估，优先考虑时间分片索引、稳定游标或明确的 `total=null`/“结果未穷尽”契约，而不是静默截断扫描。

## 后续验收门槛

- WarningBar 抽取后，各页面现有告警文案、严重度和显示条件保持一致；
- chunk 优化以真实产物与冷启动测量为依据，不以消除构建提示作为唯一目标；
- 清理功能必须先预览、后确认，并保护活动任务、有效事务和审计事实；
- 审计提前停止必须在 API 和 UI 中显式表达结果是否完整，不能返回看似精确但实际截断的总数。
