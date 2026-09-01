---
id: aikb:projects:aikb-web:search-multi-term-query-empty-results
type: project-memory
status: verified
governance_version: 2
change_class: factual-update
authority: "共享核心 KnowledgeService.search() 源码核对与真实派生索引命令复现；未做代码修复，仅记录缺陷事实"
preparer: claude-code
reviewer: user-liuwenlong
reviewed_at: 2026-09-02
approval_status: not-required
tags: [aikb-web, aikb-mcp, knowledge-service, search, fts, trigram, sqlite, multi-term, webui]
applicable_versions: "AIKB 共享核心 system/tools/aikb-mcp 当前实现；控制仓 main@6384480"
last_verified: 2026-09-02
review_when: "KnowledgeService.search() 的查询分词、FTS trigram 索引、Web 或 MCP 搜索入口变化，或完成多词查询修复时"
supersedes: []
duplicate_check_statuses: [verified, candidate]
evidence:
  - kind: command
    ref: "python -c \"from aikb.config import Settings; from aikb.knowledge import KnowledgeService; s=Settings.load(); svc=KnowledgeService(s); print('WebUI', svc.search('WebUI', limit=5)['count']); print('multi', svc.search('WebUI 总览', limit=5)['count'])\"（cwd=system/tools/aikb-mcp）"
    result: "真实派生索引中单词 'WebUI' 命中 5 条；含空格多词 'WebUI 总览' 命中 0 条。"
    date: 2026-09-02
  - kind: command
    ref: "对 workspace/db/aikb-knowledge.db 的 chunks_fts 执行 FTS 短语查询：\"WebUI\" / \"管理终端\" / \"AIKB WebUI\" / \"webui 管理终端 总览\""
    result: "短语 \"WebUI\" 命中 100 行、\"管理终端\" 命中 5 行、连续子串 \"AIKB WebUI\" 命中 56 行；含空格的整串短语 \"webui 管理终端 总览\" 与 \"WebUI 总览\" 均为 0 行。"
    date: 2026-09-02
  - kind: file
    ref: "system/tools/aikb-mcp/aikb/knowledge.py"
    result: "search() 不做查询分词：元数据路径用整串 LIKE 匹配 id/标题/路径/正文/标签（约 358-372 行），FTS 路径把整串拼成带引号短语交给 trigram tokenizer（约 379-401 行），任一含空格的多词查询都要求整串连续出现，几乎必然返回空。"
    date: 2026-09-02
relations:
  - type: related_to
    target: aikb:projects:aikb-web:phase-1-read-only-mvp
  - type: related_to
    target: aikb:projects:aikb-web:management-webui-implementation-plan
---

# AIKB 共享检索服务多词查询必然返回空（整串短语不做分词）

## 背景

AIKB 的 Web 搜索页面（`/knowledge/search`）和 MCP `search_knowledge` 都调用共享核心 `system/tools/aikb-mcp/aikb/knowledge.py` 的 `KnowledgeService.search()`。用户在 Web 搜索框输入"WebUI 安装修复""WebUI 总览"这类自然多词查询时，页面显示"无命中"，容易被误判为知识库没有相关内容。本项目条目记录缺陷事实、根因与复现方式，供后续修复前减少重复调查。

## 问题

`KnowledgeService.search()` 把整个查询字符串（含空格）当作一个整体处理，不拆分检索词：

1. **元数据路径**（`knowledge.py:358-372`）：`LIKE '%<整串>%'` 同时匹配 id、标题、路径、正文块和标签，要求整串作为连续子串出现。
2. **FTS 路径**（`knowledge.py:379-401`）：`tokenizer == "trigram"` 时把整串拼成带引号短语 `"<整串>"` 交给 FTS5 trigram tokenizer，要求整串（含空格）的连续 trigram 序列出现。

结果是：**任何含空格的多词查询几乎必然返回 0 条**，即使每个词单独都能命中。单词查询（如 `WebUI`、`管理终端`、`总览`）正常，形成"单词能搜到、多词搜不到"的反直觉表现。

## 解决方案

本条目只记录已确认的缺陷事实，不包含已实施的代码修复。修复方向建议（未实施，待决定）：

- 按空白把查询拆分为多个检索词，采用 AND 语义（文档需同时包含所有词）；
- 元数据路径：每个检索词各自 OR 五个字段的 `LIKE`，跨词取 AND，保留精确 id/标题优先排序；
- FTS 路径：对 `len >= 3`（trigram 可建索引）的检索词构造 `"词1" AND "词2"`，短词（如两字词）无法用 trigram 表达，交给元数据 LIKE 兜底；两条路径仍按分数合并去重。

修复时应同步补充多词查询的回归测试——现有测试（`system/tools/aikb-mcp/tests/test_core.py:148-154,325`）只断言单词查询，因此缺陷从未被测试覆盖。

## 验证

在控制仓 `main@6384480`、知识仓独立目录的真实派生索引上复现（`cwd=system/tools/aikb-mcp`）：

- `search('WebUI')` → 5 条；`search('管理终端')` → 2 条；`search('总览')` → 2 条；`search('安装修复')` → 5 条；
- `search('WebUI 总览')`、`search('WebUI 管理')`、`search('WebUI 管理终端 总览')` → 0 条；
- 直接查 `chunks_fts`：短语 `"WebUI"` → 100 行、`"管理终端"` → 5 行、连续子串 `"AIKB WebUI"` → 56 行；含空格的整串短语 → 0 行。

MCP 入口（`server.py` `search_knowledge`）与 Web 入口（`api/v1/knowledge.py` `GET /search`）均直接调用同一 `KnowledgeService.search()`，两处都受影响。索引 tokenizer 为 `trigram`，`rebuilt=false`，索引本身健康；缺陷在查询构造层。

## 适用范围

适用于 AIKB 共享知识检索服务的当前行为记录，影响 Web 搜索页面与 MCP `search_knowledge` 的多词查询召回。不表示检索服务已修复，也不改变任何正式知识、规则或安全边界；修复需另行决策并走既有验证门禁。

## 关联信息

- 检索实现：`system/tools/aikb-mcp/aikb/knowledge.py`；
- MCP 搜索入口：`system/tools/aikb-mcp/aikb/server.py`；
- Web 搜索入口：`system/tools/aikb-web/backend/aikb_web/api/v1/knowledge.py`；
- 相关阶段基线：[第一阶段只读知识 MVP 实现基线](phase-1-read-only-mvp.md)、[AIKB 管理 WebUI 分阶段建设计划](management-webui-implementation-plan.md)。
