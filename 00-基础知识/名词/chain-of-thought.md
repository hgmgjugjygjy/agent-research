# Chain of Thought（思维链 / CoT）

> 图位：思维链总览第 2 层 · [总览图](../assets/agent-chain-of-thought-overview.jpg)  
> 相关：[reasoning.md](reasoning.md) · [task-understanding.md](task-understanding.md) · [planning.md](planning.md)

---

## 结论先行

**思维链（Chain of Thought, CoT）** 指把复杂问题的推理过程**显式拆成中间步骤**，再得出结论或行动。图1将其标为「思维链 / 推理拆解」层：问题分解、逻辑分析。

它是提升推理质量的**技法与能力层**，不等于完整 Agent；Agent 还需要规划、工具、记忆与反馈。

---

## 是什么

典型形态：

1. 重述问题与约束  
2. 列出中间判断  
3. 逐步收束到行动或答案  

在 Agent 中，CoT 常出现在：

- 任务理解之后、规划之前（把模糊目标拆清）  
- 核心循环的 Thinking 步骤内  
- Reflection 时的「为何认为结果不够好」

---

## 应用场景

- 多约束目标（时限 + 渠道 + 格式）  
- 需要向用户或审计展示「推理依据」  
- 减少一步跳结论导致的工具误用  

**注意**：过长的思维链会占上下文窗口；生产系统常用摘要 / 外置记忆控制长度（见 [memory.md](memory.md)、本仓 pi compaction）。

---

## 效益

| 效益 | 说明 |
|---|---|
| 可解释 | 中间步可展示或落日志 |
| 可纠错 | Reflection 可定位「哪一步错了」 |
| 可组合 | 与规划、工具选择衔接自然 |

---

## 与本仓挂钩

教学图1整张都以「思维链总览」命名，强调 Agent 的分步推理特征。工程落地上，CoT 提示只是一层；**可靠产品**仍依赖工具结果校验与门禁，而非更长的独白。
