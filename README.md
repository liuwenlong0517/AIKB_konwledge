# AIKB 知识内容面

本仓库只存放能够被查询和复用的知识内容，不存放 AIKB 自身的接入规则、个人配置、模板或行为测试。它由独立 Git 管理，物理位置由 `AIKB_KNOWLEDGE_HOME` 指定，默认装配在 `%AIKB_HOME%\content`。正常知识写入只能发生在本仓库及其局部索引中；具体条目发生新增、移动、重命名或删除时，同时维护本仓库 `CATALOG.md`。

## 分类入口

- [knowledge/README.md](knowledge/README.md)：按工程主题、语言、框架和工具组织的通用知识。
- [experience/README.md](experience/README.md)：候选内容、已验证方案、工程陷阱和决策记录。
- [workflows/README.md](workflows/README.md)：可重复执行的开发、调试、评审和发布流程。
- [projects/README.md](projects/README.md)：仅对特定项目成立的事实、约束和解决方案。

项目知识与通用知识必须分开；能够跨项目复用的结论应进入 `knowledge/`、`experience/` 或 `workflows/`，不得长期留在 `projects/`。MCP 对外继续返回 `content/...` 逻辑路径，使知识仓移动不影响稳定引用。
