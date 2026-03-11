---
name: spec-delivery-run
description: 交付闭环入口：基于已完成 spec 文档执行开发-验收-修复循环直至全部验收项通过，并持续落盘交付日志
---

# 交付闭环（开发 → 验收 → 修复 → 通过）

执行以下流程：

1. 解析目标需求目录（遵循 `reference/multi-requirement.md`）。
2. 读取 `02-prd.md`、`03-tech.md`、`04-acceptance.md`（若有 `00-clarifications.md` 仅采纳已确认结论）。
3. 调用并遵循 **spec-delivery-chat**：按验收项推进实现，真实执行测试/验收，若不通过则定位根因并修复，回到验收重复直到全部通过。
4. 在目标需求目录写入或追加 `05-delivery-log.md`（模板见 `templates/05-delivery-log.md`），每轮都记录证据与结论。
5. 全部通过后输出最终交付总结与通过依据（逐项 A-xxx 证据）。
