# 文档生成路径约定

规范文档链生成的文档按**日期 + 需求名称**存放，推荐使用 **spec/** 根路径与**编号文件名**。外部引用记录见 [spec-agent-reference.md](spec-agent-reference.md)。

## 推荐格式

**根路径**：`spec/`（位于用户项目/工作区根目录下，非插件目录）

**路径格式**：`spec/<YYYY-MM-DD>/<需求名称>/`

- **`<YYYY-MM-DD>`**：需求创建或文档生成日期，如 `2025-03-02`。
- **`<需求名称>`**：当前需求/迭代名称，建议简短、英文或拼音，如 `user-center-refactor`、`skill-verify-20260302`。

**目录内固定文件名**：

| 文档类型           | 文件名 | 说明 |
|--------------------|--------|------|
| 分析报告           | `01-analysis.md` | 需求/现状分析、结论与建议 |
| PRD                | `02-prd.md` | 产品需求文档 |
| 系统技术设计       | `03-tech.md` | 技术方案/架构与实现 |
| 验收清单           | `04-acceptance.md` | 可验收条目（A-xxx） |
| 澄清               | `00-clarifications.md` | 待澄清项与澄清结论，表格 + 修订记录 |
| 讨论记录           | `06-discussion-log.md` | 多 agent 分歧、重大假设与被否决方案的决策过程（由 `record-discussion` 写入） |
| 交付复盘           | `07-retrospective.md` | 单个需求的交付复盘与可复用约定候选（由 `retrospective-agent` 写入） |
| 上游需求输入摘要   | `context-input.md` | 外部原始输入（Jira、会议纪要等）经 `context-collector-agent` 清洗后的结构化需求上下文 |
| 文档代码漂移报告   | `spec-drift-report.md` | `spec-drift-detector` 生成的文档与代码漂移报告 |

## 可选格式（本插件原有）

若用户明确要求使用 `docs/` 根路径，则可采用：

- 路径：`docs/<YYYY-MM-DD>/<需求名称>/`
- 文件名：`analysis-report.md`、`prd.md`、`technical-design.md`、`acceptance-checklist.md`、`clarification.md`。

**默认**：未特别说明时，使用 **spec/** + **01/02/03/04/00** 文件名。

## 示例（spec 布局）

- `spec/2025-03-02/user-center-refactor/01-analysis.md`
- `spec/2025-03-02/user-center-refactor/02-prd.md`
- `spec/2025-03-02/user-center-refactor/03-tech.md`
- `spec/2025-03-02/user-center-refactor/04-acceptance.md`
- `spec/2025-03-02/user-center-refactor/00-clarifications.md`

## 全局记忆

- 项目级全局记忆文件：**`spec/00-global-memory.md`**（位于 spec 根下，非各需求目录内）。
- 撰写分析、PRD、技术设计、验收清单时，若存在该文件，须**先读取并作为全局约束**纳入（技术栈、部署、团队习惯等）；单次需求的范围/结论不得写入全局记忆，应写在需求文档或澄清文档中。
- **若该文件不存在**：首次在 spec 下生成文档时，可先创建 `spec/00-global-memory.md` 并写入最小模板（见 `reference/00-global-memory-example.md`），再继续撰写需求文档。

## 多需求时的目标解析

- 当 **spec/** 下存在多个需求目录时，目标需求按以下顺序确定：**(1) 用户显式指定需求名称或路径** → **(2) 当前编辑文件所在需求目录** → **(3) spec/.state.json 的 active 字段** → **(4) 若仅有一个需求目录则用该目录** → **(5) 否则向用户确认并列出已有需求**。详见 [multi-requirement.md](multi-requirement.md)。
- **spec/.state.json**：多需求管理的单一状态源，其 `active` 字段保存当前默认需求目录路径（如 `spec/2025-03-02/user-center-refactor`）。未显式指定需求时，读写文档以此为准；可通过「切换当前需求」更新。旧版 `spec/.active`（单行文本）仅作兼容只读回退。

## 使用说明

- 生成文档前：若用户未指定路径，向用户确认**需求名称**，日期取**当天**（或用户指定），根路径默认 **spec/**，再按上述格式创建目录并生成文件；生成后可更新 **spec/.state.json** 的 active 字段为该需求路径，便于后续默认操作该需求。
- 级联更新、澄清同步时：从**目标需求**（见上）或当前编辑的文档路径反推同一需求目录，同一目录下按**固定文件名**（01/02/03/04/00）定位其他文档。
- 澄清文档：仅维护 `00-clarifications.md`。
- **数据库 schema**：若需在分析报告中引用库表结构，可放在 `spec/db/` 下，如 `spec/db/{scheme}-{dbname}-schema.md`，与模板 01-analysis 中的引用约定一致。
