---
name: sync-clarification
description: 澄清文档与其它规范文档的双向同步；已拆分为 extract-clarification（方向一：提取）和 writeback-clarification（方向二：回写）两个独立技能，本技能作为兼容入口按方向分发。
---

# 澄清文档双向同步（分发入口）

> 此技能已拆分为两个职责更清晰的独立技能：
> - **方向一（提取）**：从文档提取待澄清项 → 调用 `extract-clarification`
> - **方向二（回写）**：将澄清结论写回文档 → 调用 `writeback-clarification`

## 方向识别与分发

| 用户意图 | 调用 |
|---------|------|
| 「从 PRD 提取待澄清」「把待澄清项加到澄清文档」 | `extract-clarification` |
| 「把澄清结论写回 PRD」「根据澄清更新文档」 | `writeback-clarification` |
| 「双向同步」「两者都做」| 先 `extract-clarification`，再 `writeback-clarification` |
| cascade-update / generate-doc-chain 调用澄清提取 | `extract-clarification` |
| orchestrator 执行澄清回写 | `writeback-clarification` |

完整逻辑详见各技能文件。
