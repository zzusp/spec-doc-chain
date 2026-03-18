---
name: doc-chain-status
description: 输出当前需求文档链与交付进度的 ASCII 仪表盘摘要，便于用户快速了解「文档完成度、待澄清数量、交付到哪一轮、各 A-xxx 状态」。
---

# 文档链与交付状态快照（doc-chain-status）

## 定位与边界

- **只读汇总，不修改任何文件**：本技能只读取 spec 目录下的文档与日志，汇总为一屏可读的 ASCII 仪表盘，不对文档或代码做任何修改。
- **面向单一需求**：一次只针对一个需求目录输出状态；目标需求解析遵循 `reference/multi-requirement.md`。
- **聚合多个来源**：综合 01–04 文档存在性、00 澄清、05 交付日志与可选 round 文件的状态，给出当前需求的整体健康度快照。

## 何时使用

- 用户问「这个需求的文档链完成度如何」「现在交付进行到哪一步了」「还有多少澄清没关」等。
- 作为其它技能的前置自检（如 `spec-doc-chat`、`spec-delivery-chat` 在执行前先打印当前状态）。

## 输入与读取范围

1. **解析目标需求目录**：按 `reference/multi-requirement.md` 的优先级确定目标目录（用户显式指定 > 当前编辑文件所在目录 > `spec/.state.json` active 字段 > spec 下唯一需求目录 > 否则提示用户选择）。
2. 从该需求目录读取（存在则读取，不存在则记为缺失）：
   - `01-analysis.md`
   - `02-prd.md`
   - `03-tech.md`
   - `04-acceptance.md`
   - `00-clarifications.md`
   - `05-delivery-log.md`
   - `delivery-rounds/round-*.md`（仅用于推断最近轮次编号与结论，不展开详细内容）
3. 可选读取 `spec/.state.json`，补充该需求在全局中的 `status` 与 `open_clarifications` 等元信息。

## 输出结构

输出一段 ASCII 仪表盘，示例：

```text
需求: user-center-refactor (2025-03-02)
状态: in-delivery

文档链:
  01-analysis       ✅
  02-prd            ✅
  03-tech           ✅
  04-acceptance     ✅
  00-clarifications ⚠️ 未关闭 3 条

交付:
  交付轮次: Round 2 进行中
  验收项: A-001 ✅  A-002 M(mock-blocked)  A-003 🔄  A-004 ⏳
```

要求：

- **文档链行**：对每个文档展示「存在/不存在」与必要时的简单提示（如 04 存在但无任何 A-xxx 时可标注为「⚠️ 未定义验收项」）。
- **澄清行**：如果存在 `00-clarifications.md`，根据表格中 `状态` 列统计「待确认」/「已确认」数量，并提示待确认条数。
- **交付行**：
  - 若存在 `05-delivery-log.md`，解析验收项状态总表，统计各状态计数（未开始/不通过/通过/阻塞/`MOCK-BLOCKED`）；
  - 推断最近的交付轮次编号及其结论（可结合轮次索引或 `delivery-rounds/round-*.md` 文件名）。
  - 对每个 A-xxx 给出简要状态（如「✅ / ⚠️ / ⏳ / ⛔ / M（MOCK-BLOCKED）」），在总行中以压缩形式展示。

## 强制约束

- **不得对文档或日志做任何修改**：本技能只读；如发现明显结构问题（如 state 无效、日志缺表），只在输出中提出提示与建议。
- **状态以事实为准，不做推测**：例如，未找到某 round 文件时只提示「未找到轮次详情」，不得猜测轮次结论。
- **必须显示 `MOCK-BLOCKED`**：若在 `05-delivery-log.md` 中存在 `MOCK-BLOCKED` 状态的 A-xxx，必须在交付状态中单独列出（或用清晰标记），提醒当前交付仍依赖 mock 需要回归真实环境。

