# Reasoning（推理）

> 相关：[chain-of-thought.md](chain-of-thought.md) · [planner.md](planner.md) · [agent-loop.md](agent-loop.md)

---

## 定义

**Reasoning** 是在「当前目标、记忆、检索结果、工具清单、上轮观察」之间，决定**下一步做什么、是否调用工具、如何填参、是否结束**的决策过程。通常由 LLM 完成，可辅以规则/分类器。

---

## 解决什么

- 连接感知与行动  
- 在多工具、多约束下做权衡  
- 为 Reflection 提供「当时为何如此决策」的依据（若留下思维摘要）

---

## 怎么用

1. 把决策所需事实放进上下文（Memory / RAG / 观察）  
2. 用结构化输出（JSON / tool call）降低胡话  
3. 高风险决策叠加 [guardrails-hitl](guardrails-hitl.md) 或外部判定器  
4. 需要可展示过程时用 [CoT](chain-of-thought.md)，但注意隐私与泄露

---

## 常见坑

- 上下文缺失却强行推理 → 幻觉行动  
- 推理与执行耦合过死，无法单测  
- 把「模型说得像在想」当成可靠推理

---

## 本仓关联

会话层（pi 等）与编排层都依赖可靠 tool-calling 推理；门禁决策可交给专用判定，不必全靠同一生成模型「自觉」。
