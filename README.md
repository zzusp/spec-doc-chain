# 规范文档链 (spec-doc-chain)

> **30 秒极速上手**：直接在 Cursor 对话中说：
> - 「新建需求 xxx，生成整套文档」→ 从零开始
> - 「分析当前项目，生成 PRD 和技术设计」→ 已有代码
> - 「我改了 PRD，同步后面的文档」→ 级联更新
> - 「按验收清单开始交付，跑到全过」→ 开发验收
> - 「这个需求现在什么状态」→ 查看进度
>
> 术语速查见 [glossary.md](reference/glossary.md)，5 分钟完整指南见 [quick-start.md](reference/quick-start.md)。

---

按前置关系维护**分析报告、PRD、系统技术设计、验收清单、澄清**等多份文档，支持按序生成、前置更新后置自动更新、澄清与其它文档双向同步；支持**对已有项目进行分析并生成分析报告、PRD、技术设计**。提供两条主流程入口：**文档链撰写与同步**（`spec-doc-chat`，只写文档不改代码）与**实现与验收闭环**（`spec-delivery-chat`，按 PRD/技术设计/验收清单开发→验收→修复直至全部 A-xxx 通过并落盘交付日志）；二者通过**统一对话**或对应命令/编排 agent 使用。

## 核心理念

**通过用户不断的澄清，从而不断完善所有工程文档。** 文档链的生成与级联更新会自然产生待确认、待决策点；这些点会进入澄清文档，由用户补充或确认；用户的澄清结论回写到分析报告、PRD、技术设计、验收清单后，文档得以迭代完善。该「生成/更新 → 提取待澄清 → 用户澄清 → 回写文档」的循环是文档质量持续提升的主线，澄清文档是连接用户与整套工程文档的枢纽。详见 [reference/core-philosophy.md](reference/core-philosophy.md)。

默认使用 `spec/` 根路径与固定文件名：文档链为 `01-analysis.md`、`02-prd.md`、`03-tech.md`、`04-acceptance.md`、`00-clarifications.md`，交付阶段另有 `05-delivery-log.md` 与 `delivery-rounds/round-<NN>.md`；澄清文档本插件仅维护 .md。撰写前若存在 `spec/00-global-memory.md` 会作为全局约束纳入；多需求时通过 `spec/.state.json` 管理当前默认需求。外部引用记录见 [reference/spec-agent-reference.md](reference/spec-agent-reference.md)。

## 文档类型与顺序

| 顺序 | 文档       | 文件名 | 说明 |
|------|------------|--------|------|
| 1    | 分析报告   | `01-analysis.md` | 需求/现状分析、结论与建议 |
| 2    | PRD        | `02-prd.md` | 产品需求文档 |
| 3    | 系统技术设计 | `03-tech.md` | 技术方案/架构与实现 |
| 4    | 验收清单   | `04-acceptance.md` | 可验收条目（A-xxx） |
| 5    | 澄清       | `00-clarifications.md` | 待澄清项与澄清结论（本插件仅维护 .md） |
| 6    | 交付日志   | `05-delivery-log.md` | 交付闭环过程总览：首页仪表盘 + 验收项状态总表（A-xxx）+ 各轮次索引 |
| 7    | 交付轮次（每轮详情） | `delivery-rounds/round-<NN>.md`（模板 `templates/05-delivery-round.md`） | 每轮验收的详细记录与可见证据，主日志引用 |

详见 [reference/doc-types-and-order.md](reference/doc-types-and-order.md)（该页仅含 01–00 文档链；交付日志与轮次见上表及 [skills/spec-delivery-chat/SKILL.md](skills/spec-delivery-chat/SKILL.md)）。流程环节与流转衔接见 [reference/flow-overview.md](reference/flow-overview.md)；**防死循环兜底**见 flow-overview 第八节「流程兜底」。术语统一说明见 [reference/glossary.md](reference/glossary.md)。

## 文档生成路径（按日期 + 需求名称）

所有生成的文档**默认**放在：

```
spec/<YYYY-MM-DD>/<需求名称>/
├── 00-clarifications.md
├── 01-analysis.md
├── 02-prd.md
├── 03-tech.md
├── 04-acceptance.md
├── 05-delivery-log.md
├── 06-discussion-log.md
├── 07-retrospective.md
└── delivery-rounds/
    └── round-<NN>.md    # 每轮详情，模板见 templates/05-delivery-round.md
```

**spec 根下**还可有：`spec/00-global-memory.md`（全局约束，撰写 01–04 时若存在则纳入）、`spec/.state.json`（多需求状态管理，含 `active` 当前默认需求路径与 `requirements[]` 数组）。旧版 `spec/.active`（单行文本）仅作兼容只读回退，不再写入。

例如：`spec/2025-03-02/user-center-refactor/`。生成前若未提供需求名称或日期，会向用户确认。详见 [reference/path-convention.md](reference/path-convention.md)。可选使用 `docs/` 根路径与长文件名，见 path-convention。

**多需求时**：通过 **spec/.state.json** 的 `active` 字段记录当前默认需求目录；未显式指定需求时，操作目标按「用户指定 > 当前编辑文件所在目录 > spec/.state.json active > 仅有一需求则用该目录」解析，否则向用户确认。可对话中说「切换到 xxx」「列出所有需求」或使用命令 `doc-chain-set-active`、`doc-chain-list`。详见 [reference/multi-requirement.md](reference/multi-requirement.md)。

## 文档模板

各文档类型均有统一模板框架，生成时按模板填充。模板位于 `templates/` 目录：

- 分析报告 → [templates/01-analysis.md](templates/01-analysis.md)
- PRD → [templates/02-prd.md](templates/02-prd.md)
- 系统技术设计 → [templates/03-tech.md](templates/03-tech.md)
- 验收清单 → [templates/04-acceptance.md](templates/04-acceptance.md)
- 澄清 → [templates/00-clarifications.md](templates/00-clarifications.md)
- 交付日志 → [templates/05-delivery-log.md](templates/05-delivery-log.md)
- 交付轮次 → [templates/05-delivery-round.md](templates/05-delivery-round.md)（每轮验收详细记录与证据，主日志引用）
- 讨论记录 → [templates/06-discussion-log.md](templates/06-discussion-log.md)（多 agent 分歧、重大假设与被否决方案的决策过程，由 `record-discussion` 写入）
- 交付复盘 → [templates/07-retrospective.md](templates/07-retrospective.md)（单个需求的交付复盘与可复用约定候选，由 `retrospective-agent` 写入）

## 使用方式

### 统一对话入口（推荐）

在 Cursor 对话中直接说明你的需求即可，例如：

**文档链（撰写/同步，不改代码）**
- 「从分析报告开始，按顺序生成整套文档」（需确认需求名称，输出到 `spec/日期/需求名称/`）
- 「分析当前项目，生成分析报告、PRD 和技术设计」
- 「我改完了 PRD，请把技术设计和验收清单同步更新一下」
- 「把澄清文档里第 2 条的结论写回 PRD」「从技术设计里把需要我确认的点抽到澄清文档」

**交付闭环（实现 + 验收，改代码、跑测试、写日志）**
- 「按文档开始开发，跑验收直到全部通过」「交付这一需求，修到验收全过再停」
- 「执行 spec-delivery-run / 用 spec-delivery-chat 跑完当前需求」

Agent 会解析意图并调度对应技能（文档链 → `spec-doc-chat`，交付 → `spec-delivery-chat` / delivery-orchestrator）完成操作。

### 技能（可用 `/技能名` 调用）

| 技能 | 作用 |
|------|------|
| `generate-doc-chain` | 按前置顺序逐个生成文档（spec/ + 01-04-00 路径与模板） |
| `analyze-existing-project` | 分析已有代码库，生成分析报告、PRD、系统技术设计（可选验收清单与澄清） |
| `cascade-update`     | 前置文档更新后级联更新后置文档 |
| `extract-clarification` | 从 01/02/03/04 提取待澄清项写入 `00-clarifications.md` |
| `writeback-clarification` | 将澄清结论回写到 01/02/03/04（含 doc-agent-stance + discussion 流水线） |
| `spec-doc-chat`      | 文档链统一入口：解析意图并调度生成、级联、澄清提取与回写（仅文档，不改代码） |
| `spec-delivery-chat` | **交付流程入口**：基于已完成 spec（PRD/技术设计/验收清单）执行”开发→验收→不通过→修复→再验收…”直至**全部 A-xxx 在本流程内执行并通过**，并强制落盘 `05-delivery-log.md`（可拆分 round 详情）；验收责任不可转嫁、不得遗留”需用户执行”的验证。详见 [skills/spec-delivery-chat/SKILL.md](skills/spec-delivery-chat/SKILL.md) |

### 命令

| 命令 | 作用 |
|------|------|
| `doc-chain-generate-all`       | 一次性从分析报告生成到澄清文档（spec/ + 01-04-00） |
| `doc-chain-analyze-project`    | 分析当前或指定项目，生成分析报告、PRD、技术设计（可选完整文档链） |
| `doc-chain-cascade-after-edit` | 编辑某文档后级联更新后置并同步澄清 |
| `doc-chain-sync-clarification` | 执行澄清双向同步 |
| `doc-chain-set-active`         | 设置当前默认需求（写入 spec/.state.json 的 active 字段） |
| `doc-chain-list`               | 列出 spec/ 下所有需求目录及当前激活需求 |
| `spec-delivery-run`            | 交付闭环入口：基于已完成 spec 执行开发→验收→修复直至全部通过，并落盘交付日志 |

### Agent

- **doc-chain-orchestrator**：编排 agent，通过统一入口理解意图并调度生成、级联、澄清同步，保证文档链与澄清一致。
- **delivery-orchestrator**：交付闭环编排 agent；调度 `spec-delivery-chat` 执行实现与验收循环直至通过，并记录交付日志。
- **context-collector-agent**：需求来源注入 agent；在进入文档链主流程前，将 Jira ticket、会议纪要、外部 PRD、口述转录等原始材料清洗成结构化的 `context-input.md`，供后续文档生成消费。
- **spec-drift-detector**：文档代码漂移检测 agent；在交付轮次结束后或按需触发，对比 02/03/04 与实际代码，输出 `spec-drift-report.md`，提示「文档有但代码无」「代码有但文档无」「两者不一致」等漂移类型。
- **retrospective-agent**：交付复盘与知识沉淀 agent；在全部 A-xxx 通过后生成 `07-retrospective.md` 复盘摘要，并在用户确认后将可复用约定候选沉淀到 `spec/00-global-memory.md`。
- **test-runner**：测试自动化，主动跑测、分析失败并修复。
- **verifier**：任务完成后校验实现是否真实可用、是否遗漏边界。
- **debugger**：错误与失败根因分析，最小修复并验证。

> 说明：`doc-agent-stance`、`doc-agent-discussion`、`record-discussion` 等技能为编排内部的子技能，用于多 agent 表态、讨论与决策过程记录（`06-discussion-log.md`），通常由 orchestrator 间接调用，无需用户直接记忆技能名。

## 规则

- **doc-dependency**：文档前置关系与级联更新规则（在匹配的文档路径下生效）。
- **clarification-sync**：澄清文档与其它文档的写入/回写规则（在澄清相关文件下生效）。
- **delivery**：实现阶段的完整交付闭环（开发→测试/验收→修复→再验收直至通过）；仅写需求文档时不强制。
- **coding**：代码复用、文件约束、注释、根因修复与 YAGNI；仅编辑纯需求文档（.md）时可酌情适用。

## 在 Claude Code 命令行中使用

> 重要：本仓库当前是 **Cursor 插件项目**，不是 Claude Code 原生插件包。

### 当前现状

- `agents/`、`skills/`、`rules/`、`commands/`、`.cursor-plugin/plugin.json` 这一套结构，是**面向 Cursor 插件/规则体系**组织的。
- 其中 `.cursor-plugin/plugin.json` 是 **Cursor 插件元数据**，**不是 Claude Code CLI 的原生加载格式**。
- 因此，**直接在 Claude Code CLI 中打开本仓库，并不会自动把这些 agents / skills / rules 当成 Claude Code 原生能力加载**。

### 在 Claude Code 里怎么用这个项目

有两种方式：

#### 方式一：把它当作普通项目文档来使用（立即可用）

在 Claude Code 里进入本仓库目录后，可以直接让 Claude 读取本项目文档并按文档约定协作，例如：

- 「先读 README.md 和 reference/quick-start.md，告诉我这个项目怎么用」
- 「按 README 里的规范，帮我生成一套 spec 文档」
- 「根据 templates/ 和 reference/ 约定，帮我补全当前需求的 PRD / 技术设计 / 验收清单」
- 「按 `skills/spec-delivery-chat/SKILL.md` 的流程，帮我执行交付闭环」

也就是说：**把这些 README / reference / skills / agents 文件当成项目内的“流程说明书”来让 Claude 遵循**。

#### 方式二：迁移为 Claude Code 原生项目记忆/指令（推荐）

如果你希望在 Claude Code 中获得更接近 Cursor 插件的体验，建议把关键约束整理到仓库根目录的 `CLAUDE.md` 中，让 Claude Code 在进入仓库时自动读取。例如可以把这些内容迁移进去：

- 项目目标与核心理念（来自 `README.md` / `reference/core-philosophy.md`）
- 文档链顺序与路径约定（来自 `reference/doc-types-and-order.md` / `reference/path-convention.md`）
- 澄清回写、级联更新、交付闭环规则（来自 `skills/` 与 `rules/`）
- 常用操作示例（来自 `reference/quick-start.md`）

推荐最小做法：

1. 新建仓库根目录 `CLAUDE.md`
2. 写入：项目简介、文档路径规则、澄清流程、交付流程、常用指令示例
3. 在 Claude Code 中进入该仓库后，直接自然语言操作

### 建议的 Claude Code 使用姿势

如果你暂时不迁移为 `CLAUDE.md`，建议在 Claude Code 里这样开场：

```text
请先阅读 README.md、reference/quick-start.md、reference/flow-overview.md、reference/glossary.md，理解这个项目的文档链与交付流程。后续严格按这些约定协作。
```

然后再继续说你的目标，例如：

```text
请按本项目规范，新建需求 user-center-refactor，先生成分析报告、PRD、技术设计、验收清单，再生成澄清文档。
```

或：

```text
请按本项目的 spec-delivery-chat 规则，对当前需求执行开发→验收→修复→再验收，直到全部 A-xxx 通过。
```

### 如果想真正适配 Claude Code

后续可以做一轮专门适配，把当前 Cursor 结构整理成：

- `CLAUDE.md`：Claude Code 自动加载的项目说明
- 更精简的 `reference/`：用户手册
- 保留 `templates/`：继续作为产物模板
- 把当前 `skills/agents/rules` 中真正需要 Claude Code长期记住的约束，提炼进 `CLAUDE.md`

这样 Claude Code 的使用体验会明显更自然。


将本目录作为 Cursor 插件安装（项目级或用户级），确保 `.cursor-plugin/plugin.json` 存在。

- **启用方式**：在 Cursor 设置中打开插件/扩展，添加或启用本插件；可选择**项目级**（仅当前仓库）或**用户级**（全局可用）。
- 安装后规则与技能会出现在 Cursor 设置中；文档链统一入口由编排 agent 与 `spec-doc-chat` 提供，交付闭环由 `delivery-orchestrator` 与 `spec-delivery-chat` 提供。
- 首次使用建议先看 [reference/quick-start.md](reference/quick-start.md) 快速跑通一条文档链与交付。

## 许可证

MIT
