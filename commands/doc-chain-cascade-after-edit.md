---
name: doc-chain-cascade-after-edit
description: 用户刚编辑了某一前置文档，级联更新其后置文档并同步澄清文档
---

# 编辑后级联更新

执行以下流程：

1. 确认用户刚编辑的文档（分析报告 / PRD / 系统技术设计 / 验收清单）及路径；**目标需求目录**按 reference/multi-requirement.md 解析（当前编辑文件所在目录优先）。
2. 调用 **cascade-update** 技能：根据前置→后置关系更新所有受影响的后续文档；该技能内部会对每个更新后的文档调用 **extract-clarification** 提取新待澄清项。
3. 回复：已级联更新的文件列表、澄清文档完整路径及新增的待澄清项（如有），**以及建议的下一步**（如「请在 `spec/.../00-clarifications.md` 填写第 X 条结论后说一声同步回 PRD」），推动澄清循环。
