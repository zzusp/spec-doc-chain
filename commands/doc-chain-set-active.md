---
name: doc-chain-set-active
description: 设置当前默认需求目录（写入 spec/.state.json 的 active 字段），后续未显式指定需求的操作将针对该目录
---

# 设置当前需求

执行以下流程：

1. 从用户消息中解析目标需求：**需求名称**（如 `user-center-refactor`）或**完整路径**（如 `spec/2025-03-02/user-center-refactor`）。若仅给名称，可结合 spec/ 下已有目录匹配（如最近日期下的同名目录），或向用户确认日期。
2. 确认该需求目录存在（按 `reference/multi-requirement.md` 的「需求目录识别规则」验证）；若不存在可提示用户先创建或生成该需求文档。
3. 更新 **spec/.state.json**：
   - 若文件不存在，按以下最小结构创建：`{"active": "<路径>", "requirements": [{"path": "<路径>", "status": "planning", "open_clarifications": 0, "shared_modules": []}]}`
   - 若文件已存在：更新 `active` 字段；若该需求路径尚不在 `requirements` 数组中，追加一条记录（初始 `status` 可设为 `planning` 或 `in-doc-chain`）
   - 写入前校验：`active` 所指目录真实存在；若路径不存在，拒绝写入并提示修正
4. 回复确认：「当前需求已设为 `spec/YYYY-MM-DD/<名称>`，后续生成、级联更新、澄清同步等未指定需求的操作将默认针对该目录。」并可选列出该目录下已有文档。

与其它使用 `spec/.state.json` 的工具共用工作区时，同一文件可被共同更新。外部引用见 [reference/spec-agent-reference.md](reference/spec-agent-reference.md)。
