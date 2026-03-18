---
name: doc-chain-sync-clarification
description: 执行澄清文档双向同步——从其它文档提取待澄清项（extract-clarification），或根据澄清结论回写相关文档（writeback-clarification）
---

# 澄清文档双向同步

执行以下流程：

1. 判断用户需求是「提取待澄清项」还是「根据澄清结论回写」或「两者都做」。
2. **提取待澄清项**：调用 `extract-clarification`，对指定或最近变更的文档提取待澄清内容写入澄清文档。
3. **澄清回写**：调用 `writeback-clarification`，对澄清文档中状态为「已确认」且已填写结论的项回写至对应文档（含 agent 表态流程、whole-document review、回写后再提取、必要时级联）。
4. **回写后单轮边界**：本轮「回写 → 提取 → 级联（若需）」执行一轮即结束；新产生的待澄清项由用户填写后显式发起下一轮，避免死循环。
5. 回复：同步了哪些文件、澄清文档中新增/已确认的项、本次新产生的待澄清项（如有），以及建议的下一步。
