---
name: doc-chain-generate-all
description: 从分析报告开始按前置顺序一次性生成分析报告、PRD、系统技术设计、验收清单与澄清文档
---

# 一次性生成完整文档链

执行以下流程：

1. 确认需求名称与日期，输出目录**默认**为 **spec/<YYYY-MM-DD>/<需求名称>/**。
2. 若存在 `spec/00-global-memory.md`，先读取并作为全局约束。
3. 调用 **generate-doc-chain** 技能：从分析报告开始，按顺序生成 `01-analysis.md`、`02-prd.md`、`03-tech.md`、`04-acceptance.md`；每生成一份文档后调用 **extract-clarification** 提取待澄清项写入 `00-clarifications.md`（若 00 已存在则追加）。每份文档更新时在修订记录表中追加一行。
4. 完成后列出所有生成的文件路径、**待澄清项数量及澄清文档完整路径**（如 `spec/2025-03-18/xxx/00-clarifications.md`），并提示用户可进行的下一步（在澄清文档中填写结论后说「把澄清写回文档」），推动澄清循环。

若用户已提供需求概要或现有材料，将其作为分析报告与 PRD 的输入依据。
