# spec-doc-chain 插件：全面分析报告

本文档对当前插件进行系统化、专业化分析，涵盖结构完整性、设计一致性、可优化点与建议追加能力。分析基准：Cursor 插件规范、插件自述的「文档链 + 澄清双向同步 + 统一入口」目标；外部约定引用见 reference/spec-agent-reference.md。

---

## 一、整体评价

| 维度         | 评价 | 说明 |
|--------------|------|------|
| 结构完整性   | 良好 | 具备 plugin.json、skills、rules、commands、agents、reference、templates，符合 Cursor 插件常规结构 |
| 文档与约定   | 良好 | reference 与 README 清晰，路径/多需求/澄清流程有据可查 |
| 技能与编排   | 良好 | 五技能 + 编排 agent + 统一入口，覆盖生成、分析、级联、澄清、多需求 |
| 一致性       | 需改进 | 澄清状态用词、spec-doc-chat 描述与调度表存在小幅不一致 |
| 可发现性/体验 | 可增强 | 缺「需求目录」判定算法、错误与边界说明、版本/变更说明 |

---

## 二、结构与规范符合度

### 2.1 符合项

- **插件清单**：`.cursor-plugin/plugin.json` 存在，含 name、displayName、version、description、keywords 等。
- **目录布局**：`commands/`、`agents/`、`skills/`、`rules/` 位于插件根目录，符合 Cursor 约定（非嵌套在 .cursor-plugin 内）。
- **技能形态**：各技能为独立目录 + `SKILL.md`，frontmatter 含 `name`、`description`。
- **规则形态**：`.mdc` 含 `description`、`globs`/`alwaysApply`，与 Cursor rules 兼容。
- **命令形态**：commands 为 `.md`，含 frontmatter `name`、`description`。

### 2.2 建议补充（非强制）

- **plugin.json**：可增加 `repository`、`homepage` 或 `bugs`（若开源），便于追溯与反馈。
- **README 安装**：可补充「如何从 Cursor 设置中启用插件、选择项目级/用户级」的简短步骤，降低首次使用门槛。

---

## 三、设计一致性

### 3.1 澄清状态用词不统一（建议修复）

| 位置 | 当前表述 | 建议 |
|------|----------|------|
| `templates/00-clarifications.md` | 状态仅可填写：`待确认`、`已确认` | 作为**规范用词**，保持不变 |
| `skills/sync-clarification/SKILL.md` 方向二 | 状态为「已确认」 | 与模板一致 ✓ |
| `skills/sync-clarification/SKILL.md` 建议结构 | 状态（待澄清/已闭合） | 改为「待确认/已确认」，与模板统一 |
| `rules/clarification-sync.mdc` | 标记该条为已处理；已闭合 | 统一为「标记为已确认」；「已闭合」改为「已确认」或注明「即已确认」 |
| `commands/doc-chain-sync-clarification.md` | 已闭合的项 | 改为「已确认的项」 |

**理由**：模板是生成与解析澄清文档的契约，状态列应唯一规范为「待确认 | 已确认」，避免 agent 或脚本解析时出现多种别名导致判断分支复杂或遗漏。

### 3.2 spec-doc-chat 描述与调度表

- **frontmatter description**：当前只写了「调度 generate-doc-chain、cascade-update、sync-clarification」，未提 **analyze-existing-project**。
- **建议**：在 description 中增加「及 analyze-existing-project」，与技能内「支持的操作与对应技能」表一致，便于在技能列表中一眼看到「分析已有项目」也由此入口调度。

---

## 四、逻辑与边界

### 4.1 需求目录的识别算法（建议显式成文）

- **现状**：`doc-chain-list`、`multi-requirement.md` 等多处约定「符合 spec/<日期>/<名称>/ 且目录内存在 01-analysis.md 或 00-clarifications.md 等约定文件」。
- **缺失**：未在单一处给出**可执行定义**，例如：
  - 日期格式是否严格 `YYYY-MM-DD`（四位数年、两位月日）？
  - 需求名称是否允许包含哪些字符（如仅英文/数字/连字符/拼音）？
  - 认定「需求目录」的充分条件：是「至少存在 01-analysis.md 或 00-clarifications.md 之一」即可，还是必须同时满足目录层级为 `spec/<date>/<name>/`？
- **建议**：在 `reference/multi-requirement.md` 或新建 `reference/requirement-directory-detection.md` 中增加一小节「需求目录识别规则」，写明：
  - 目录模式：`spec/<YYYY-MM-DD>/<需求名称>/`（可注明日期、名称的推荐格式）。
  - 认定条件：该目录下存在以下任意一个约定文件即视为有效需求目录：`01-analysis.md`、`02-prd.md`、`03-tech.md`、`04-acceptance.md`、`00-clarifications.md`。
  - 这样 list/set-active 及「仅有一个需求目录则用该目录」的解析可被 agent 或后续脚本一致实现。

### 4.2 spec/.active 的格式与错误处理

- **现状**：约定为单行路径，如 `spec/2025-03-02/user-center-refactor`。
- **建议在 path-convention 或 multi-requirement 中明确**：
  - 是否允许带或不带 `spec/` 前缀（若允许两种，需约定解析规则，避免重复拼接）。
  - 当 .active 指向的目录不存在或已被删除时：list 已说明「注明 .active 指向路径无效」；建议在 set-active 或「目标解析」中注明：若当前 spec/.active 指向无效，应视为「未设置」，按解析顺序落到「仅有一需求」或「向用户确认」。
  - 可选：建议 set-active 在写入前校验目标目录存在（已有「若不存在可提示用户先创建或生成该需求文档」），保持即可。

### 4.3 级联与澄清的触发边界

- **cascade-update**：已明确「验收清单更新 → 无强制后置」「若与 PRD/技术设计不一致可提示」。
- **建议**：在 cascade-update 或 doc-dependency 中补一句：当**仅**修改了 04-acceptance 且未改前置文档时，是否仍建议执行「澄清提取」（方向一）。若希望行为一致，可写「仅更新验收清单时，仍检查是否有新待澄清项并写入 00-clarifications.md」。

---

## 五、技能与命令

### 5.1 技能覆盖

- 生成链：generate-doc-chain ✓  
- 分析已有项目：analyze-existing-project ✓  
- 级联更新：cascade-update ✓  
- 澄清双向同步：sync-clarification ✓  
- 统一入口：spec-doc-chat ✓  

技能与 README/agent 描述一致，无缺口。

### 5.2 命令与技能的对应

- `doc-chain-generate-all` → generate-doc-chain ✓  
- `doc-chain-analyze-project` → analyze-existing-project ✓  
- `doc-chain-cascade-after-edit` → cascade-update + sync-clarification 方向一 ✓  
- `doc-chain-sync-clarification` → sync-clarification 双向 ✓  
- `doc-chain-set-active` / `doc-chain-list` → 多需求管理 ✓  

建议在 README 的「命令」表中，对「级联」「澄清」两条注明「内部会再调用澄清提取/双向同步」，便于高级用户理解与调试。

### 5.3 从某一环开始生成（已有，可更显式）

- generate-doc-chain 已支持「从某一文档类型开始，按前置顺序生成到验收清单或澄清」。
- 建议在 **spec-doc-chat** 的「用户意图示例」表中增加一行，例如：「从 PRD 开始生成、从技术设计开始补全」→ 按序生成（起始点非分析报告），避免用户以为只能「从分析报告开始」。

---

## 六、规则（rules）

### 6.1 适用范围

- **delivery.mdc**：`alwaysApply: true`，但正文写明「仅撰写或更新需求文档时不强制本交付流程」。若 Cursor 对 alwaysApply 的规则会全局注入，建议在规则开头再次强调「本规则在实现阶段（按 spec 进行开发与验收）适用；仅编辑 spec 下 .md 时不强制交付闭环」，避免在纯写 PRD/分析报告时被强推「测试/验收」。
- **coding.mdc**：`alwaysApply: true`，已说明「仅编辑纯需求文档时可酌情适用」，合理。

### 6.2 doc-dependency 与 clarification-sync 的 glob

- doc-dependency 的 glob 已覆盖 `spec/**/0[0-4]-*.md`、`**/docs/**` 及若干中文名模式，能命中 spec 下约定文档。
- clarification-sync 的 glob 针对澄清文件与含「澄清」的文件，与职责一致。
- 无冲突，无需调整；若未来支持 `docs/` 根路径下的长文件名（如 `clarification.md`），可确认 clarification-sync 的 glob 是否已覆盖（当前 `**/*澄清*`、`**/clarification*` 应能覆盖）。

---

## 七、模板与 reference

### 7.1 模板

- 01/02/03/04/00 结构完整，与 reference 描述一致；02/03/04 含 DEPENDENCY-SIGNATURE 占位，约定见 reference/spec-agent-reference.md。
- 01-analysis 中 `spec/db/{scheme}-{dbname}-schema.md` 的引用在 path-convention 中未单独说明；若希望统一，可在 path-convention 增加一句「数据库 schema 等可放在 spec/db/ 下，分析报告可引用」，避免被当作未定义路径。

### 7.2 reference

- doc-types-and-order、path-convention、multi-requirement、00-global-memory-example 已形成闭环。
- 建议新增（可选）：
  - **reference/requirement-directory-detection.md**：需求目录识别规则（见 4.1）。
  - **reference/glossary.md**：简要术语表（如 待确认/已确认、C-xxx、A-xxx、R-xx、级联、澄清回写、目标需求解析顺序），便于新人或新 agent 快速对齐用语。

---

## 八、可追加的设计与能力

### 8.1 文档链健康检查（建议）

- **能力**：只读检查当前需求目录下 01–04–00 是否存在、修订记录是否连续、澄清项是否都有归属文档与关联章节、验收项是否含可执行步骤与可断言标准等。
- **触发**：命令如 `doc-chain-health-check`，或统一入口意图「检查当前文档链是否完整/一致」。
- **价值**：在交付或评审前快速发现缺失文件、断链或澄清未闭环，无需人工逐份打开。

### 8.2 从 PRD/技术设计单篇刷新（建议）

- **能力**：用户只改了 PRD 或只改了技术设计，希望「只重新生成 04-acceptance」或「只根据 PRD+技术设计刷新验收清单」，而不做全链级联。
- **实现**：可在 cascade-update 中明确支持「仅更新 04-acceptance（基于当前 02+03）」作为一种调用模式，或由 spec-doc-chat 解析「只刷新验收清单」并调用该模式。
- **价值**：大文档链下减少不必要的中间文档重写，加快迭代。

### 8.3 澄清项 ID（C-xxx）的生成与冲突

- **现状**：模板示例为 C-001，sync-clarification 要求「新增一行」「若尚未存在」；未明确 C-xxx 由谁生成、是否自增、多轮追加是否可能重复或冲突。
- **建议**：在 sync-clarification 或 reference 中约定：C-xxx 由 agent 在插入新行时分配，规则为同需求目录内已有最大编号 +1；若澄清文档被用户手改编号，agent 以「不重复 ID」为原则分配新号。避免两处同时插入都写成 C-002 等。

### 8.4 与外部工具的边界（文档化即可）

- **现状**：分工与兼容性已收敛到 reference/spec-agent-reference.md，主文档（如 README）不展开。
- **建议**：若需表格化「本插件负责 / 外部工具负责」，可在 reference/spec-agent-reference.md 中补充。

### 8.5 版本与变更说明（可选）

- **现状**：plugin.json 有 version，无 CHANGELOG。
- **建议**：增加 `CHANGELOG.md` 或至少在 README 末尾保留「版本历史」小节，记录 1.0.0 及之后的重要变更（如澄清状态用词统一、需求目录识别规则成文等），便于协作与升级判断。

---

## 九、总结与优先级建议

| 优先级 | 类型     | 项 |
|--------|----------|----|
| 高     | 一致性   | 澄清状态用词统一为「待确认/已确认」，并统一「已闭合/已处理」的表述 |
| 高     | 可维护性 | spec-doc-chat 的 description 中补充 analyze-existing-project |
| 中     | 逻辑边界 | 成文「需求目录识别规则」；明确 spec/.active 无效时的解析行为 |
| 中     | 体验     | spec-doc-chat 表增加「从 PRD/技术设计开始生成」的示例 |
| 中     | 可选能力 | 澄清项 C-xxx 的分配与冲突避免约定 |
| 低     | 增强     | 文档链健康检查命令或意图；「仅刷新验收清单」的级联模式 |
| 低     | 文档     | path-convention 提一句 spec/db/；分工表见 reference/spec-agent-reference.md；CHANGELOG/版本历史 |

按上述顺序实施，可先完成「高」与「中」项，再视需要做「低」项与设计追加，在保持约定一致（见 reference/spec-agent-reference.md）的前提下，提升插件一致性、可预测性与可维护性。

---

## 修复记录（已落实）

- **澄清状态**：全插件统一为「待确认/已确认」；rules/commands 中「已闭合/已处理」已改为「已确认」。
- **spec-doc-chat**：description 已补充 analyze-existing-project；表内已增加「从 PRD/技术设计开始生成」示例。
- **需求目录识别**：已在 reference/multi-requirement.md 成文「需求目录识别规则」与「spec/.active 无效时」处理。
- **spec/.active 格式**：已在 multi-requirement 中明确推荐带 `spec/` 前缀及不带前缀时的解析规则。
- **delivery.mdc**：开头已增加「仅编辑 spec 下 .md 时不触发本流程」的强调。
- **path-convention**：已补充 spec/db/ 与 01-analysis 中 schema 引用的说明。
- **C-xxx 分配**：已在 sync-clarification 方向一中约定「同需求目录内最大编号+1、不重复 ID」。
- **仅改 04 时的澄清**：cascade-update 与 doc-dependency 已注明「仅修改验收清单时仍建议执行澄清提取」。
- **第二轮检查**：doc-chain-list 与「需求目录识别规则」对齐（明确五类约定文件任一见一即有效）；doc-chain-set-active 补充推荐路径格式及 reference 引用；doc-chain-sync-clarification 回写步骤表述修正（回写对象本为已确认项，改为「该条保持已确认状态」）。
- **全插件检查与优化**：generate-doc-chain 步骤4 调整顺序（生成前读取→填写→whole-document review→写回并追加修订记录），步骤5 去掉重复的「每完成一份追加修订记录」；sync-clarification 方向二步骤4 后增加「完成步骤3、4 后写回该文档」；flow-overview 生成链步骤补充「写回前须 whole-document review」；doc-chain-sync-clarification 命令步骤3 明确回写含 whole-document review；orchestrator 职责3 增加「每份文档更新时须 whole-document review」；analyze-existing-project 注意中增加写回前须 whole-document review；doc-types-and-order 澄清段落补充 whole-document review 与回写后澄清提取。
