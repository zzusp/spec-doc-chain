# 外部引用：spec-agent

本插件在路径、文件名、澄清表格结构等方面参考了 [spec-agent](https://github.com/cursor/plugins)，便于工作区共用或迁移。**仅在此记录引用关系，插件主文档（如 README）不展开 spec-agent 说明。**

## 兼容性对照（引用记录）

- **路径**：默认 `spec/YYYY-MM-DD/<需求名称>/`，与 spec-agent 的 task 路径一致。
- **文件名**：`01-analysis.md`、`02-prd.md`、`03-tech.md`、`04-acceptance.md`、`00-clarifications.md` 与 spec-agent 一致；本插件不生成 `00-clarifications.json`，由 spec-agent 脚本按需维护。
- **全局记忆**：若存在 `spec/00-global-memory.md`，撰写任一份文档前会读取并作为全局约束；若不存在可先创建并写入最小模板（见 [00-global-memory-example.md](00-global-memory-example.md)）。单次需求范围/结论不写入全局记忆。
- **澄清**：本插件仅维护 `00-clarifications.md`；表格列与 spec-agent（ID、状态、优先级、影响范围、归属文档、关联章节、问题/待确认点、用户确认/补充、解决方案）一致。
- **修订记录**：每份文档含 `## 修订记录` 表，每次更新追加一行。
- **DEPENDENCY-SIGNATURE**：02/03/04 模板中含依赖签名区块占位，哈希由 spec-agent 脚本或流程更新；本插件不修改签名内容。若需 final-check、subagent、sync-memory 等，请配合使用 spec-agent。
