# 多需求管理

当 **spec/** 下存在多个需求目录（如 `spec/2025-03-01/req-a/`、`spec/2025-03-02/req-b/`）时，通过**当前需求**与**显式指定**避免操作错目录。

## 当前需求与需求状态（spec/.state.json）

- 文件 **`spec/.state.json`** 是多需求管理的单一状态源，使用 JSON 结构化记录：

```json
{
  "active": "spec/2025-03-02/user-center-refactor",
  "requirements": [
    {
      "path": "spec/2025-03-02/user-center-refactor",
      "status": "in-delivery",
      "open_clarifications": 3,
      "shared_modules": ["user-core", "auth-gateway"]
    }
  ]
}
```

- 字段约定：
  - `active`：当前操作的默认需求路径，必须是 `requirements[].path` 之一。
  - `requirements[].path`：需求目录的相对路径，推荐带 `spec/` 前缀，如 `spec/2025-03-02/user-center-refactor`。
  - `requirements[].status`：需求状态，枚举值：`planning` / `in-doc-chain` / `in-delivery` / `delivered` / `archived`。
  - `requirements[].open_clarifications`：该需求当前未关闭澄清项数量（可用于状态仪表盘）。
  - `requirements[].shared_modules`：该需求声明依赖/影响的共享模块标识（如 `user-core`、`billing-engine`），用于检测跨需求潜在冲突。
- 所有「生成、级联更新、澄清同步、查看状态」等操作在**未显式指定需求**时，以 `spec/.state.json` 中的 `active` 路径为默认目标。
- 任何写入 `spec/.state.json` 的操作必须在写前校验：`active` 与所有 `requirements[].path` 所指目录真实存在；若路径不存在，则视为状态无效，拒绝静默操作并在回复中提示需修正状态文件或创建对应目录。

## 目标需求解析顺序（强制）

执行任一会写文档或读需求目录的操作时，按以下顺序确定**目标需求目录**：

1. **用户显式指定**：用户消息中给出需求名称（如「对 user-center-refactor 做级联更新」）或路径（如「spec/2025-03-02/req-a」）→ 用该需求。
2. **当前编辑文件所在目录**：若用户正在编辑某需求下的文档（如 `spec/2025-03-02/req-a/02-prd.md`）且未提其他需求 → 用该文档所在需求目录。
3. **spec/.state.json**：若存在且有效（`active` 指向的目录存在）→ 用该路径。
4. **仅有一个需求目录**：若 spec 下仅有一个日期/名称子目录 → 用该目录。
5. **否则**：向用户确认「要对哪个需求操作？」并列出 `spec/` 下已有需求目录，待用户指定后再执行。

多需求时**优先让用户显式指定**或先切换当前需求，避免歧义。

## 切换当前需求（set-active）

- **何时用**：用户说「切换到 xxx」「当前需求设为 2025-03-02/req-b」「以后默认操作 req-a」等。
- **操作**：更新 `spec/.state.json` 中的 `active` 字段，并确保存在对应的 `requirements[]` 条目：
  - 若该需求已在 `requirements` 中：仅更新 `active`。
  - 若为新需求：在确认目录存在后追加一条 `requirements` 记录（初始 `status` 可设为 `planning` 或 `in-doc-chain`）。
- **路径格式**：与 path-convention 一致，如 `spec/<YYYY-MM-DD>/<需求名称>`；若用户只给需求名称，可结合最近日期或询问日期后写入。

## 需求目录识别规则（list / set-active / 目标解析共用）

以下为 list、set-active 及目标需求解析的**权威约定**，所有识别与解析均据此执行。

- **目录模式**：`spec/<YYYY-MM-DD>/<需求名称>/`。日期推荐四位数年+两位月+两位日（如 2025-03-02）；需求名称建议英文或拼音，可含连字符。
- **认定条件**：上述路径下存在以下**任意一个**约定文件，即视为有效需求目录：`01-analysis.md`、`02-prd.md`、`03-tech.md`、`04-acceptance.md`、`00-clarifications.md`。
- **spec/.state.json 无效时**：若 `active` 字段指向的路径不存在或已被删除，视为「未设置当前需求」，按目标解析顺序落到「仅有一需求则用该目录」或「向用户确认并列出需求」，并在回复中注明 state 需修复。

## 列出需求（list）

- **何时用**：用户问「有哪些需求」「列出所有需求目录」「现在在哪个需求」等。
- **操作**：按上述「需求目录识别规则」扫描 **spec/** 下所有有效需求目录，列出路径与可选摘要（如首行标题）；若存在 **spec/.state.json**，标出当前激活需求（`active`）；若 `active` 指向路径不存在，在列表中注明「state 中 active 路径无效」。
- **不写文件**，仅只读并回复。

## 建议用法

- 新建需求并准备在该需求上连续操作前：生成文档后**顺手更新 spec/.state.json** 中的 `active` 与对应 `requirements[]` 条目，或让用户说一句「当前需求设为刚建的 xxx」。
- 多需求并行时：每次做级联、澄清同步等前，**先确认目标**（「对 req-a 还是 req-b？」）或先执行「切换到 xxx」再操作。
- 若其它工具也维护 `spec/.state.json`（或仍使用历史的 `spec/.active`），本插件以 `spec/.state.json` 为主，必要时可兼容只读 `.active` 作为回退（但不再写入 .active）。
