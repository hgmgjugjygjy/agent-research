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

## 技术实现原理

**实现方式**：(1) 提示式——在指令中要求「先逐步分析再给结论」或给 few-shot 推理示例；(2) 原生推理模型——模型内部生成隐藏 reasoning token，API 只返回摘要或不返回；(3) Self-consistency——同一问题采样 n 条推理链（temperature>0），对最终答案多数投票。

```python
answers = [extract_final(llm(q + "\n请逐步推理，最后一行写『答案：』", temperature=0.7)) for _ in range(5)]
final = Counter(answers).most_common(1)[0][0]
```

**为什么有效**：自回归模型每个 token 的计算量固定，把中间步骤写出来等于「用更多 token 换更多计算」，并让后续 token 能条件于中间结果。工程上应把推理与最终输出分开（结构化字段），只把结论交给下游。

## 本仓关联

教学配图常把「思维链」画成能力层；工程上请落到 Reasoning + 可审计 trace，而不是只加一句「请逐步思考」。
