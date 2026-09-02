---
id: aikb:projects:aikb-web:webui-deferred-maintenance-review
type: project-memory
status: verified
governance_version: 2
change_class: factual-update
authority: "用户于 2026-09-02 先决定延期评估，数据维护落地后又明确要求修复终态私有材料、事务摘要增长和自动清理；WarningBar、公共 chunk 与无时间条件审计提前停止继续延期"
preparer: codex
reviewer: user-liuwenlong
reviewed_at: 2026-09-02
approval_status: not-required
tags: [aikb-web, webui, performance, maintainability, audit, data-maintenance]
applicable_versions: AIKB WebUI at control commits cf12301253c15613012fb2e75a1e1d6929fe2e46 and b2a03e3833e84486dd8150787cf72aace330fc01
last_verified: 2026-09-02
review_when: WarningBar出现第四个消费者或语义分叉、公共chunk超过800kB或首屏性能可测量恶化、清理类别或保留期需要产品化配置、审计查询引入索引或游标契约时
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
  - kind: commit
    ref: "cf12301253c15613012fb2e75a1e1d6929fe2e46"
    result: "维护事务 succeeded/rolled_back 终态摘要可靠落盘后清理私有材料，启动恢复重试终态清理；recovery_required 和未知材料继续保留。"
    date: 2026-09-02
  - kind: commit
    ref: "b2a03e3833e84486dd8150787cf72aace330fc01"
    result: "数据维护扩展到规则/维护终态事务摘要，默认保留90天；启动恢复完成后等待5分钟、每6小时自动执行固定策略，并增加共享锁、陈旧候选和Windows重解析点保护。"
    date: 2026-09-02
relations:
  - type: related_to
    target: aikb:projects:aikb-web:management-webui-implementation-plan
  - type: related_to
    target: aikb:projects:aikb-web:phase-2-runtime-audit
---

# WebUI 延后维护项与复核门槛

## 当前决策

下列项目在 2026-09-02 的复核后仍没有构成当前功能、安全或稳定性阻断，因此继续延期：

- 三处页面内 `WarningBar` 的重复实现；
- Vite 公共依赖 chunk 超过默认 500 kB 提示；
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

## 已落地的 workspace 收敛

workspace 清理已经采用固定类别、安全状态白名单和服务端保留期：审计 90 天、终态归档任务 180 天、终态 Web 任务 30 天、终态规则与维护事务摘要 90 天。用户仍可通过短期预览和确认令牌显式清理；后台任务在规则/维护恢复收敛后等待 5 分钟，随后每 6 小时按相同固定默认策略执行。共享维护锁繁忙、恢复门禁阻断、对象变化、未知材料、非安全终态或 Windows 重解析点都会跳过而不扩大删除面。

维护事务的私有材料在 `succeeded`/`rolled_back` 终态摘要原子保存后立即清理，启动恢复及自动周期只重试已声明材料；`recovery_required`、非终态及未知内容永久保持保护，必须由后续显式恢复或人工处置。当前修复解决了已纳入五个固定类别的无界增长，不代表提供任意目录、容量阈值或自定义保留策略。

## 继续延期的审计查询项

无时间条件的审计任意筛选无法在保持精确 `total` 和稳定分页语义的同时安全提前停止；提前停止会让用户误以为结果已经完整。该项等待数据维护/索引能力落地后重新评估，优先考虑时间分片索引、稳定游标或明确的 `total=null`/“结果未穷尽”契约，而不是静默截断扫描。

## 后续验收门槛

- WarningBar 抽取后，各页面现有告警文案、严重度和显示条件保持一致；
- chunk 优化以真实产物与冷启动测量为依据，不以消除构建提示作为唯一目标；
- 新增清理类别或可配置策略仍须保护活动任务、有效事务、未知材料和重解析点，并维持陈旧候选拒绝；
- 审计提前停止必须在 API 和 UI 中显式表达结果是否完整，不能返回看似精确但实际截断的总数。
