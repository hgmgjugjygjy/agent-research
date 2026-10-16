# 关键名词索引

> 每篇独立短文；主文加粗词可点进此处。  
> 配图总览：[../assets/agent-chain-of-thought-overview.jpg](../assets/agent-chain-of-thought-overview.jpg) · [../assets/agent-complete-architecture.jpg](../assets/agent-complete-architecture.jpg)

---

## 中英对照 + 一句话

| 中文 | English | 一句话 | 文档 |
|---|---|---|---|
| 规划器 | Planner | 把用户目标拆成可执行步骤序列的模块 | [planner.md](planner.md) |
| 规划（能力层） | Planning | 「会做计划」这一能力层；与 Planner 模块区分 | [planning.md](planning.md) |
| 推理 | Reasoning | 在计划与现状之间决定「下一步做什么、怎么做」 | [reasoning.md](reasoning.md) |
| 思维链 | Chain of Thought (CoT) | 把推理显式拆成中间步骤的技法 / 层 | [chain-of-thought.md](chain-of-thought.md) |
| 任务理解 | Task Understanding | 识别意图、提取约束、澄清目标 | [task-understanding.md](task-understanding.md) |
| 记忆 | Memory | 长/短/工作记忆，支撑一致性与续跑 | [memory.md](memory.md) |
| 工具 | Tool | Agent 可调用的外部能力（检索、库、Slack…） | [tool.md](tool.md) |
| MCP | Model Context Protocol | 统一协议连接外部系统与工具 | [mcp.md](mcp.md) |
| 反思 | Reflection | 检查结果、发现问题、触发修正或回流 | [reflection.md](reflection.md) |
| Agent 核心循环 | Agent Loop | Thinking → Action → Action Input 的热循环 | [agent-loop.md](agent-loop.md) |
| 执行 | Execution | 真正发起调用、产生中间结果 / 副作用 | [execution.md](execution.md) |
| 检查与修正 | Check & Correction | 自检、纠偏、回退到规划（图1第七层） | [check-and-correction.md](check-and-correction.md) |

---

## 阅读建议

- **先主轴**：Planner → Reasoning → Agent Loop → Reflection  
- **再侧翼**：Memory · Tool · MCP  
- **再对齐图1**：Task Understanding · Planning · CoT · Execution · Check & Correction  

## 刻意不合并的说明

| 对 | 为何分开 |
|---|---|
| Planner vs Planning | 前者是架构模块；后者是图1「规划」能力层，产品文档常混用，专篇拆开避免选型时指错对象 |
| Reasoning vs Chain of Thought | 推理是决策职责；CoT 是常用实现技法 / 图1第二层 |
| Reflection vs Check & Correction | 图2用 Reflection；图1用检查与修正——同族不同切面，各留一篇并交叉引用 |

---

← [目录导读](../README.md) · [Agent 是什么](../01-agent是什么.md) · [完整架构](../02-agent完整架构.md)
