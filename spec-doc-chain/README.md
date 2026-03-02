# 规范文档链 (spec-doc-chain)

按前置关系维护**分析报告、PRD、系统技术设计、验收清单、澄清**等多份文档，支持按序生成、前置更新后置自动更新、澄清与其它文档双向同步；支持**对已有项目进行分析并生成分析报告、PRD、技术设计**；并通过**统一对话入口**完成所有操作。

## 核心理念

**通过用户不断的澄清，从而不断完善所有工程文档。** 文档链的生成与级联更新会自然产生待确认、待决策点；这些点会进入澄清文档，由用户补充或确认；用户的澄清结论回写到分析报告、PRD、技术设计、验收清单后，文档得以迭代完善。该「生成/更新 → 提取待澄清 → 用户澄清 → 回写文档」的循环是文档质量持续提升的主线，澄清文档是连接用户与整套工程文档的枢纽。详见 [reference/core-philosophy.md](reference/core-philosophy.md)。

**与 [spec-agent](https://github.com/cursor/plugins) 兼容**：默认使用相同路径与文件名约定（`spec/` 根路径 + `01-analysis.md`、`02-prd.md`、`03-tech.md`、`04-acceptance.md`、`00-clarifications.md`），可与 spec-agent 工作区共用或迁移；澄清文档本插件仅维护 .md，.json 由 spec-agent 按需维护。撰写前若存在 `spec/00-global-memory.md` 会作为全局约束纳入。

## 文档类型与顺序

| 顺序 | 文档       | 文件名 | 说明 |
|------|------------|--------|------|
| 1    | 分析报告   | `01-analysis.md` | 需求/现状分析、结论与建议 |
| 2    | PRD        | `02-prd.md` | 产品需求文档 |
| 3    | 系统技术设计 | `03-tech.md` | 技术方案/架构与实现 |
| 4    | 验收清单   | `04-acceptance.md` | 可验收条目（A-xxx） |
| 5    | 澄清       | `00-clarifications.md` | 待澄清项与澄清结论（本插件仅维护 .md；与 spec-agent 共用时 .json 由 spec-agent 按需维护） |

详见 [reference/doc-types-and-order.md](reference/doc-types-and-order.md)。流程环节与流转衔接见 [reference/flow-overview.md](reference/flow-overview.md)；**防死循环兜底**见 flow-overview 第八节「流程兜底」。

## 文档生成路径（按日期 + 需求名称）

所有生成的文档**默认**放在（与 spec-agent 一致）：

```
spec/<YYYY-MM-DD>/<需求名称>/
```

例如：`spec/2025-03-02/user-center-refactor/`。生成前若未提供需求名称或日期，会向用户确认。详见 [reference/path-convention.md](reference/path-convention.md)。可选使用 `docs/` 根路径与长文件名，见 path-convention。

**多需求时**：通过 **spec/.active** 记录当前默认需求目录；未显式指定需求时，操作目标按「用户指定 > 当前编辑文件所在目录 > spec/.active > 仅有一需求则用该目录」解析，否则向用户确认。可对话中说「切换到 xxx」「列出所有需求」或使用命令 `doc-chain-set-active`、`doc-chain-list`。详见 [reference/multi-requirement.md](reference/multi-requirement.md)。

## 文档模板

各文档类型均有统一模板框架（与 spec-agent 章节对齐），生成时按模板填充。模板位于 `templates/` 目录：

- 分析报告 → [templates/01-analysis.md](templates/01-analysis.md)
- PRD → [templates/02-prd.md](templates/02-prd.md)
- 系统技术设计 → [templates/03-tech.md](templates/03-tech.md)
- 验收清单 → [templates/04-acceptance.md](templates/04-acceptance.md)
- 澄清 → [templates/00-clarifications.md](templates/00-clarifications.md)

## 与 spec-agent 的兼容性

- **路径**：默认 `spec/YYYY-MM-DD/<需求名称>/`，与 spec-agent 的 task 路径一致。
- **文件名**：`01-analysis.md`、`02-prd.md`、`03-tech.md`、`04-acceptance.md`、`00-clarifications.md`，与 spec-agent 一致（本插件不生成 `00-clarifications.json`，由 spec-agent 脚本按需维护）。
- **全局记忆**：若存在 `spec/00-global-memory.md`，撰写任一份文档前会读取并作为全局约束；若不存在可先创建并写入最小模板（见 [reference/00-global-memory-example.md](reference/00-global-memory-example.md)）。单次需求范围/结论不写入全局记忆。
- **澄清**：仅维护 `00-clarifications.md`；表格列与 spec-agent（ID、状态、优先级、影响范围、归属文档、关联章节、问题/待确认点、用户确认/补充、解决方案）一致。
- **修订记录**：每份文档含 `## 修订记录` 表，每次更新追加一行。
- **DEPENDENCY-SIGNATURE**：02/03/04 模板中含依赖签名区块占位，哈希由 spec-agent 脚本或流程更新；本插件不修改签名内容。若需 final-check、subagent、sync-memory 等，请配合使用 spec-agent。

## 使用方式

### 统一对话入口（推荐）

在 Cursor 对话中直接说明你的需求即可，例如：

- 「从分析报告开始，按顺序生成整套文档」（需确认需求名称，输出到 `spec/日期/需求名称/`）
- 「分析当前项目，生成分析报告、PRD 和技术设计」
- 「我改完了 PRD，请把技术设计和验收清单同步更新一下」
- 「把澄清文档里第 2 条的结论写回 PRD」
- 「从技术设计里把需要我确认的点抽到澄清文档」

Agent 会解析意图并调度对应技能完成操作。

### 技能（可用 `/技能名` 调用）

| 技能 | 作用 |
|------|------|
| `generate-doc-chain` | 按前置顺序逐个生成文档（spec/ + 01-04-00 路径与模板） |
| `analyze-existing-project` | 分析已有代码库，生成分析报告、PRD、系统技术设计（可选验收清单与澄清） |
| `cascade-update`     | 前置文档更新后级联更新后置文档 |
| `sync-clarification` | 澄清文档与其它文档双向同步（提取待澄清 / 澄清回写） |
| `spec-doc-chat`      | 统一入口：解析意图并调度上述操作 |

### 命令

| 命令 | 作用 |
|------|------|
| `doc-chain-generate-all`       | 一次性从分析报告生成到澄清文档（spec/ + 01-04-00） |
| `doc-chain-analyze-project`    | 分析当前或指定项目，生成分析报告、PRD、技术设计（可选完整文档链） |
| `doc-chain-cascade-after-edit` | 编辑某文档后级联更新后置并同步澄清 |
| `doc-chain-sync-clarification` | 执行澄清双向同步 |
| `doc-chain-set-active`         | 设置当前默认需求（写入 spec/.active） |
| `doc-chain-list`               | 列出 spec/ 下所有需求目录及当前激活需求 |

### Agent

- **doc-chain-orchestrator**：编排 agent，通过统一入口理解意图并调度生成、级联、澄清同步，保证文档链与澄清一致。

## 规则

- **doc-dependency**：文档前置关系与级联更新规则（在匹配的文档路径下生效）。
- **clarification-sync**：澄清文档与其它文档的写入/回写规则（在澄清相关文件下生效）。
- **delivery**：实现阶段的完整交付闭环（开发→测试/验收→修复→再验收直至通过）；仅写需求文档时不强制。
- **coding**：代码复用、文件约束、注释、根因修复与 YAGNI；仅编辑纯需求文档（.md）时可酌情适用。

## 安装

将本目录作为 Cursor 插件安装（项目级或用户级），确保 `.cursor-plugin/plugin.json` 存在。安装后规则与技能会出现在 Cursor 设置中，统一对话入口通过编排 agent 与 `spec-doc-chat` 技能提供。

## 许可证

MIT
