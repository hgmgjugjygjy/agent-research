# 关键名词索引（Glossary）

> 每篇独立短文，结构统一：**定义 / 解决什么 / 怎么用 / 技术实现原理 / 常见坑 / 本仓关联**；关键篇含配图。  
> 路径：`00-fundamentals/glossary/`（英文文件名；正文中文）  
> 主文：[../01-what-is-an-agent.md](../01-what-is-an-agent.md) · [../02-agent-architecture.md](../02-agent-architecture.md)

---

## 中英对照索引

| 中文 | English | 一句话 | 文档 |
|---|---|---|---|
| Agent 核心循环 / ReAct | Agent Loop / ReAct | Thinking→Action→Observation 热循环 | [agent-loop.md](agent-loop.md) |
| 规划器 / 规划 | Planner / Planning | 目标拆步或拆阶段 | [planner.md](planner.md) · [planning.md](planning.md) |
| 推理 | Reasoning | 决定下一步做什么、怎么做 | [reasoning.md](reasoning.md) |
| 思维链 | Chain of Thought | 显式中间推理步骤 | [chain-of-thought.md](chain-of-thought.md) |
| 任务理解 | Task Understanding | 意图、约束、澄清 | [task-understanding.md](task-understanding.md) |
| 记忆 | Memory | 短/长/工作记忆 | [memory.md](memory.md) |
| 检索增强生成 | **RAG** | 检索外部知识再生成；≠ Memory | [rag.md](rag.md) |
| 向量与嵌入 | Embedding / Vector Store | RAG 底层：向量化与近邻检索 | [embedding-vector-store.md](embedding-vector-store.md) |
| 向量数据库 | Vector Database | ANN 索引、度量、过滤、产品对比 | [vector-database.md](vector-database.md) |
| 工具 / 函数调用 | Tool / Function calling | 模型可触发的外部能力 | [tool.md](tool.md) |
| MCP | Model Context Protocol | 统一协议连接外部系统 | [mcp.md](mcp.md) |
| 编排 | Orchestration | 图/状态机/工作流 | [orchestration.md](orchestration.md) |
| 反思 | Reflection | 检查结果、回流或熔断 | [reflection.md](reflection.md) |
| 执行 | Execution | 真正发起调用与副作用 | [execution.md](execution.md) |
| 检查与修正 | Check & Correction | 自检纠偏（Reflection 近亲） | [check-and-correction.md](check-and-correction.md) |
| 门禁与人机 | Guardrails / HITL | 范围、权限、审批 | [guardrails-hitl.md](guardrails-hitl.md) |
| 多 Agent | Multi-agent | 多角色分工协作 | [multi-agent.md](multi-agent.md) |
| 上下文压缩 | Context compaction | 长会话压窗保关键 | [context-compaction.md](context-compaction.md) |
| 上下文爆炸 | Context explosion | token 持续增长的成因与解法 | [context-explosion.md](context-explosion.md) |
| 提示词 | Prompt / System prompt | 指令与角色边界 | [prompt-system-prompt.md](prompt-system-prompt.md) |
| 评测与追踪 | Evaluation / Trace | 质量门与可观测 | [evaluation-trace.md](evaluation-trace.md) |

---

## 相关主文

- [../05-multi-agent-collaboration.md](../05-multi-agent-collaboration.md) — 五种多 Agent 协作模式
- [../06-context-engineering.md](../06-context-engineering.md) — 上下文工程

## 阅读建议

- **主轴**：Planner → Reasoning → Agent Loop → Reflection  
- **侧翼**：Memory · **RAG** · Tool · MCP  
- **平台层**：Orchestration · Guardrails/HITL · Compaction · Observability  
- **协作**：Multi-agent  

## 刻意区分

| 对 | 为何分开 |
|---|---|
| Memory vs RAG | 任务状态 vs 外部知识取证；混用会导致「用聊天记录当知识库」或「检索结果当工作进度」 |
| Planner vs Planning | 模块 vs 能力层；选型时避免指错对象 |
| Reasoning vs CoT | 决策职责 vs 常用技法 |
| Reflection vs Check & Correction | 同族不同切面；图2/图1用语差异 |
| Orchestration vs Agent Loop | 战役/阶段 vs 会话热循环 |
