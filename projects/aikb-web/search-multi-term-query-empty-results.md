---
id: aikb:projects:aikb-web:search-multi-term-query-empty-results
type: project-memory
status: verified
governance_version: 2
change_class: factual-update
authority: "控制仓提交 f4c9015 的共享 KnowledgeService.search() 多词 AND 查询实现、核心/Web 全量测试与真实派生索引复验"
preparer: codex
reviewer: user-liuwenlong
reviewed_at: 2026-09-02
approval_status: not-required
tags: [aikb-web, aikb-mcp, knowledge-service, search, fts, trigram, sqlite, multi-term, webui]
applicable_versions: "AIKB 共享核心 system/tools/aikb-mcp；控制仓 main@f4c9015 及以后，直至检索契约再次变化"
last_verified: 2026-09-02
review_when: "KnowledgeService.search() 的分词、跨词语义、FTS tokenizer、字段匹配、排序合并或 Web/MCP 搜索入口变化时"
supersedes: []
duplicate_check_statuses: [verified, candidate, deprecated]
evidence:
  - kind: commit
    ref: "f4c9015"
    result: "共享检索按空白拆词，元数据路径采用词内字段 OR、跨词 AND；FTS 路径使用逐词短语 AND，并为 trigram 短词追加 LIKE 约束。"
    date: 2026-09-02
  - kind: test
    ref: "python -m unittest discover -s tests -p 'test_*.py'（cwd=system/tools/aikb-mcp）"
    result: "95 项通过、1 项按环境跳过；新增多词顺序无关、trigram 短词兜底和不存在词保持 0 条的回归覆盖。"
    date: 2026-09-02
  - kind: test
    ref: "pwsh -NoProfile -File system/tools/aikb-web/scripts/validate-aikb-web.ps1"
    result: "共享核心 95 项、Web 后端 289 项、前端 62 项通过；typecheck、lint、build 与 286 个 Markdown/55 个知识文件结构校验通过。"
    date: 2026-09-02
  - kind: command
    ref: "真实派生索引调用 KnowledgeService.search()：WebUI 总览 / WebUI 管理终端 总览 / WebUI 不存在词"
    result: "前两项分别命中 3 条和 2 条；包含不存在词的查询为 0 条，证明修复后召回恢复且保持 AND 语义。"
    date: 2026-09-02
relations:
  - type: related_to
    target: aikb:projects:aikb-web:phase-1-read-only-mvp
  - type: related_to
    target: aikb:projects:aikb-web:management-webui-implementation-plan
---

# AIKB 共享检索服务多词 AND 查询修复

## 背景

AIKB 的 Web 搜索页面（`/knowledge/search`）和 MCP `search_knowledge` 都调用共享核心 `system/tools/aikb-mcp/aikb/knowledge.py` 的 `KnowledgeService.search()`。控制仓 `6384480` 及以前把含空格查询当作一个连续短语，导致“单词能搜到、多词搜不到”。控制仓提交 `f4c9015` 已在共享核心修复该问题，两种入口无需分别修改。

## 问题

修复前，`KnowledgeService.search()` 把整个查询字符串（含空格）当作一个整体处理，不拆分检索词：

1. **元数据路径**（`knowledge.py:358-372`）：`LIKE '%<整串>%'` 同时匹配 id、标题、路径、正文块和标签，要求整串作为连续子串出现。
2. **FTS 路径**（`knowledge.py:379-401`）：`tokenizer == "trigram"` 时把整串拼成带引号短语 `"<整串>"` 交给 FTS5 trigram tokenizer，要求整串（含空格）的连续 trigram 序列出现。

结果是含空格的自然多词查询几乎必然返回 0 条，即使每个词单独都能命中。索引本身健康，缺陷位于查询构造层。

## 解决方案

`f4c9015` 在共享 `KnowledgeService.search()` 中实施以下契约：

- 按空白拆分检索词，跨词采用 AND 语义；词的输入顺序不要求在正文中连续出现。
- 元数据路径中，每个词可命中 ID、标题、逻辑路径、当前正文块或标签，五类字段取 OR，多个词组取 AND；精确 ID/标题排序仍按完整原查询计算。
- FTS 路径把可索引词构造成 `"词1" AND "词2"`。trigram tokenizer 无法表达不足 3 个字符的短词，因此短词作为 LIKE 条件继续约束同一个 FTS 候选，不能因长词命中而降级成 OR。
- 元数据与 FTS 候选继续沿用既有分数、去重、标签过滤、数量和摘要预算；Web 与 MCP API 参数及返回结构不变。

当前 AND 语义作用于同一索引 chunk 可见的文档元数据、标签和正文；本修复不改变 chunk 划分、跨 chunk 聚合、LIKE 通配符或排序权重。

## 验证

在控制仓提交 `f4c9015` 上完成以下验证：

- 隔离回归：`search("SQLite 中文")` 命中缓存条目，`search("缓存 SQLite")` 顺序反转仍命中，`search("SQLite 不存在")` 返回 0 条。
- 真实派生索引：`search("WebUI 总览")` 命中 3 条，`search("WebUI 管理终端 总览")` 命中 2 条，`search("WebUI 不存在词")` 返回 0 条。
- 共享核心测试 95 项通过、1 项跳过；Web 后端 289 项通过、16 项跳过；前端 62 项通过，typecheck、lint、build 和结构校验通过。

验证使用的真实索引 tokenizer 为 `trigram` 且 `rebuilt=false`，说明结果变化来自查询构造修复，而不是重建或替换索引。

## 适用范围

适用于控制仓 `f4c9015` 及以后、检索契约未再次变化时的 AIKB 共享知识搜索。影响 Web 搜索页面与 MCP `search_knowledge` 的多词召回，但不改变 verified/candidate 状态过滤、标签过滤、索引事实源、读写权限或安全边界。

## 关联信息

- 修复提交：控制仓 `f4c9015`；
- 检索实现与回归：`system/tools/aikb-mcp/aikb/knowledge.py`、`system/tools/aikb-mcp/tests/test_core.py`；
- MCP 搜索入口：`system/tools/aikb-mcp/aikb/server.py`；
- Web 搜索入口：`system/tools/aikb-web/backend/aikb_web/api/v1/knowledge.py`；
- 相关阶段基线：[第一阶段只读知识 MVP 实现基线](phase-1-read-only-mvp.md)、[AIKB 管理 WebUI 分阶段建设计划](management-webui-implementation-plan.md)。
