# Reasoning（推理）

> 图位：完整架构主轴第 2 步 · [总览图](../assets/agent-complete-architecture.jpg)  
> 相关：[chain-of-thought.md](chain-of-thought.md) · [agent-loop.md](agent-loop.md) · [planner.md](planner.md)

---

## 结论先行

**Reasoning** 是计划与执行之间的**决策层**：根据当前目标、记忆与上一步结果，决定「下一步做什么、用什么方式做」。它不是最终答案生成器，而是**调度判断**。

---

## 是什么

完整架构中，Planner 给出步骤列表后，Reasoning：

- 选择当前应推进哪一步  
- 判断信息是否足够进入 Action  
- 在工具失败时决定重试、换工具或回 Planner  

与核心循环的关系：Reasoning 常包裹或紧邻 Thinking；产品实现里二者可能合并进同一 LLM 调用，但概念上 **Reasoning = 决策职责**，**Thinking = 单次循环内的分析**。

---

## 应用场景

| 场景 | Reasoning 的典型判断 |
|---|---|
| 资料收集 | 先搜网页还是先读已有附件 |
| 发送通知前 | 纪要是否达标；频道是否正确 |
| 工具报错 | 换参数、换工具，还是升级为人工 |

---

## 与 Chain of Thought

| | Reasoning | Chain of Thought |
|---|---|---|
| 角色 | 架构职责：决策下一步 | 技法 / 图1层：显式中间推理步骤 |
| 可无 CoT？ | 可以（直接出决策） | CoT 是提升复杂推理质量的常用手段 |

详见 [chain-of-thought.md](chain-of-thought.md)。

---

## 效益

- 把「计划」变成「当前可执行意图」  
- 为 Reflection 提供可追溯的决策点  
- 便于日志审计：「为何选了 Slack 而不是邮件」

---

## 与本仓挂钩

LangGraph 节点内的路由函数、pi tool loop 中的「模型选 tool」都是 Reasoning 的工程形态。门禁应限制 Reasoning **不可覆盖**的确定性规则（如 scope）。
