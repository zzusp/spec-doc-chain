# 术语表

本页统一插件内常用术语，便于人与 agent 快速对齐用语。详细约定见对应 reference 与技能说明。

| 术语 | 说明 |
|------|------|
| **待确认 / 已确认** | 澄清文档中每条澄清项的**状态**。仅可填写此二值：`待确认` 表示待用户填写结论；`已确认` 表示用户已填写并可回写至其它文档。见 [templates/00-clarifications.md](../templates/00-clarifications.md)、[sync-clarification](../skills/sync-clarification/SKILL.md)。 |
| **C-xxx** | 澄清项 ID，如 C-001、C-002。由 agent 在向澄清文档新增行时分配，同需求目录内取已有最大编号 +1；若用户手改编号，以不重复 ID 为原则分配。见 [sync-clarification](../skills/sync-clarification/SKILL.md)。 |
| **CC-xxx** | Context Collector 专用临时澄清 ID，如 CC-001、CC-002。仅出现在 `context-input.md` 的「待澄清项」表，用于标记从外部原始输入中识别出的多义/缺失点；后续由文档链流程在提取待澄清时**转换为正式 C-xxx 写入 `00-clarifications.md`**，转换规则为：在目标需求目录下读取当前 `00-clarifications.md` 中最大 C 号 +1 作为新 C-xxx，将对应 CC-xxx 的问题迁移为该 C-xxx，并在 `context-input.md` 中注明对应关系。见 [context-collector-agent](../agents/context-collector-agent.md)。 |
| **A-xxx** | 验收项 ID，如 A-001、A-002。出现在 `04-acceptance.md` 中，每条对应可执行、可断言的验收条目。交付流程以「全部 A-xxx 在本流程内通过」为完成标准。见 [doc-types-and-order.md](doc-types-and-order.md)、[spec-delivery-chat](../skills/spec-delivery-chat/SKILL.md)。 |
| **R-xx** | 需求编号（可选）。用于在 PRD/技术设计/验收清单中做需求追溯，使 A-xxx 能对应到具体需求与设计。见 [doc-types-and-order.md](doc-types-and-order.md)、[spec-agent-reference.md](spec-agent-reference.md)。 |
| **级联更新** | 前置文档（分析报告、PRD、技术设计、验收清单之一）被更新后，按 01→02→03→04 的前置→后置关系依次更新其后置文档，并在每份更新后执行澄清提取。见 [cascade-update](../skills/cascade-update/SKILL.md)、[flow-overview.md](flow-overview.md)。 |
| **澄清回写** | 将澄清文档中状态为「已确认」且已填结论的项，按归属文档与关联章节写回 01/02/03/04；回写后须 whole-document review，并对更新后的文档再做澄清提取，必要时触发级联。见 [sync-clarification](../skills/sync-clarification/SKILL.md) 方向二。 |
| **提取待澄清** | 从 01/02/03/04 中识别需用户确认或二义性点，在澄清文档中新增行（C-xxx），供用户填写。见 [sync-clarification](../skills/sync-clarification/SKILL.md) 方向一。 |
| **目标需求解析顺序** | 写文档或读需求目录时确定**目标需求目录**的优先级：用户显式指定 > 当前编辑文件所在目录 > spec/.active > 仅有一需求目录 > 否则向用户确认并列出需求。见 [multi-requirement.md](multi-requirement.md)。 |
| **whole-document review** | 每份文档更新时须**整体回顾整篇文档**，不仅改直接对应段落；有矛盾可自行收敛则修正，需用户决策则追加到澄清文档。见 [flow-overview.md](flow-overview.md)、各技能中的「必须」步骤。 |
| **spec/.active** | 保存当前默认需求目录路径的单行文件，位于 `spec/.active`。未显式指定需求时，生成、级联、澄清、list 等以该路径为默认目标；若指向的目录不存在则视为未设置。见 [multi-requirement.md](multi-requirement.md)。 |
| **需求目录** | 符合 `spec/<YYYY-MM-DD>/<需求名称>/` 且其下存在至少一个约定文件（01/02/03/04/00）的目录。list、set-active 与目标解析均据此识别。见 [multi-requirement.md](multi-requirement.md)「需求目录识别规则」。 |
