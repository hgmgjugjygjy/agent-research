# Chain of Thought（思维链，CoT）

> 相关：[reasoning.md](reasoning.md) · [prompt-system-prompt.md](prompt-system-prompt.md)

---

## 定义

**Chain of Thought** 是一种提示/解码技法：要求模型把答案拆成**显式中间步骤**再给结论。它是实现 Reasoning 可观察性的常用手段，**不是**独立系统组件。

---

## 解决什么

- 提高多步算术/逻辑类任务正确率（视模型与任务而定）  
- 便于调试「错在哪一步」  
- 产品上可展示「思考过程」（需脱敏）

---

## 怎么用

- 提示：「一步步想」「先列要点再结论」  
- 或使用提供「思考通道」的模型 API（若可用）  
- 与 tool call 结合：想清楚再选工具，避免无意义调用

---

## 常见坑

- 暴露 CoT 导致提示注入或隐私泄漏  
- 冗长 CoT 挤占窗口 → 需 [compaction](context-compaction.md)  
- 以为开了 CoT 就等于有 Planner / 门禁

---

## 本仓关联

教学配图常把「思维链」画成能力层；工程上请落到 Reasoning + 可审计 trace，而不是只加一句「请逐步思考」。
