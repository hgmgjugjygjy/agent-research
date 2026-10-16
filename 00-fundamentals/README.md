# 00 · Fundamentals：Agent 入门

**受众**：产品 / 架构 / 安全工程（零 Agent 基础也可读）  
**定位**：本仓的**入门层**——先弄清「Agent 是什么、由哪些组成」，再读竞品与框架选型。  
**约束**：只讲概念与中性示例（竞品分析、查库、发 Slack）；**不含** exploit / PoC。  
**路径**：`00-fundamentals/`（英文目录；正文中文）

---

## 结论先行

| 要点 | 说明 |
|---|---|
| **Agent ≠ 一次问答** | 目标驱动的「感知 → 决策 → 行动 → 观察 → 再决策」闭环；与 chatbot / 单次 LLM 调用本质不同 |
| **核心组成** | LLM 推理核 · Planner · Tools/Function calling · Memory · **RAG** · MCP/连接器 · Orchestration（图/状态机）· Reflection/Eval · Guardrails/HITL · Observability |
| **典型循环** | ReAct / Plan-Execute / 多 Agent 协作 |
| **本仓选型挂钩** | 编排 [LangGraph](../02-frameworks/langgraph.md) · 会话/压缩 [pi](../02-frameworks/pi.md) · 门禁 HITL · 质量流水线 YoungFlow/VH |

---

## 阅读顺序（建议）

1. **[01-what-is-an-agent.md](01-what-is-an-agent.md)** — 定义、与 chatbot 差异、核心组成、典型循环、与本仓选型关系  
2. **[03-large-language-models.md](03-large-language-models.md)** — LLM 入门：定义、能力、训练、参数规模、上下文窗口、与 Agent 的关系、各家官方描述与价格表、技术实现原理  
3. **[02-agent-architecture.md](02-agent-architecture.md)** — 工程架构走读（含自写 Mermaid；外部示意图作附录参考）  
4. **[05-multi-agent-collaboration.md](05-multi-agent-collaboration.md)** — 五种多 Agent 协作模式、上下文爆炸、何时值得  
5. **[06-context-engineering.md](06-context-engineering.md)** — 上下文工程：预算、分区、压缩、隔离、常见失败  
6. **[glossary/](glossary/README.md)** — 关键名词专篇（含 RAG、编排、门禁、多 Agent、上下文压缩等）

不必一次读完所有名词；遇到主文加粗词时点进专篇即可。

---

## 本目录结构

```
00-fundamentals/
  README.md                 ← 本页
  01-what-is-an-agent.md
  02-agent-architecture.md
  03-large-language-models.md   ← 阅读顺序排在 01 之后
  05-multi-agent-collaboration.md
  06-context-engineering.md
  glossary/                 ← 关键名词专篇（英文文件名，含技术实现原理）
  assets/                   ← 自绘图 *.svg / *.png；外部参考 *.jpg
    src/                    ← Mermaid 源码 *.mmd
```

---

## 图谱索引

全部为本仓自绘：Mermaid 源码在 `assets/src/*.mmd`，经 `@mermaid-js/mermaid-cli`（mmdc）+ Noto Sans CJK 渲染为 SVG（主）与 PNG（2x，便于手机预览）。

| 图名 | 文件 | 用在哪篇 |
|---|---|---|
| 系统架构示意图 | [system-architecture.svg](assets/system-architecture.svg) · [png](assets/system-architecture.png) · [src](assets/src/system-architecture.mmd) | 01 §2.0 · 02 §1.1 |
| 端到端数据流图 | [dataflow.svg](assets/dataflow.svg) · [png](assets/dataflow.png) · [src](assets/src/dataflow.mmd) | 01 §2.0 · 02 §2 |
| 组件关系图 | [component-relations.svg](assets/component-relations.svg) · [png](assets/component-relations.png) · [src](assets/src/component-relations.mmd) | 02 §1.2 |
| RAG 检索链路图 | [rag-pipeline.svg](assets/rag-pipeline.svg) · [png](assets/rag-pipeline.png) · [src](assets/src/rag-pipeline.mmd) | glossary/rag |
| 向量库写入/查询流 | [vector-database.svg](assets/vector-database.svg) · [png](assets/vector-database.png) · [src](assets/src/vector-database.mmd) | glossary/vector-database |
| ReAct 循环图 | [react-loop.svg](assets/react-loop.svg) · [png](assets/react-loop.png) · [src](assets/src/react-loop.mmd) | 01 §3.1 · glossary/agent-loop |
| Plan-Execute 循环图 | [plan-execute.svg](assets/plan-execute.svg) · [png](assets/plan-execute.png) · [src](assets/src/plan-execute.mmd) | 01 §3.2 · glossary/agent-loop |
| 多 Agent 主管协作图 | [multi-agent.svg](assets/multi-agent.svg) · [png](assets/multi-agent.png) · [src](assets/src/multi-agent.mmd) | 01 §3.3 · glossary/multi-agent |
| HITL 门禁时序图 | [hitl-sequence.svg](assets/hitl-sequence.svg) · [png](assets/hitl-sequence.png) · [src](assets/src/hitl-sequence.mmd) | glossary/guardrails-hitl |
| 记忆分层图 | [memory-layers.svg](assets/memory-layers.svg) · [png](assets/memory-layers.png) · [src](assets/src/memory-layers.mmd) | glossary/memory · glossary/context-compaction |
| 工具调用 / MCP 时序图 | [tool-calling-mcp.svg](assets/tool-calling-mcp.svg) · [png](assets/tool-calling-mcp.png) · [src](assets/src/tool-calling-mcp.mmd) | glossary/tool · glossary/mcp |
| 编排状态机图 | [orchestration-graph.svg](assets/orchestration-graph.svg) · [png](assets/orchestration-graph.png) · [src](assets/src/orchestration-graph.mmd) | glossary/orchestration |
| 多 Agent 模式总览 | [ma-overview.svg](assets/ma-overview.svg) · [png](assets/ma-overview.png) · [src](assets/src/ma-overview.mmd) | 05 §0 |
| Pipeline 模式 | [ma-pipeline.svg](assets/ma-pipeline.svg) · [png](assets/ma-pipeline.png) · [src](assets/src/ma-pipeline.mmd) | 05 §1 |
| Hierarchical 模式 | [ma-hierarchical.svg](assets/ma-hierarchical.svg) · [png](assets/ma-hierarchical.png) · [src](assets/src/ma-hierarchical.mmd) | 05 §2 |
| Peer 协商模式 | [ma-peer.svg](assets/ma-peer.svg) · [png](assets/ma-peer.png) · [src](assets/src/ma-peer.mmd) | 05 §3 |
| Blackboard 模式 | [ma-blackboard.svg](assets/ma-blackboard.svg) · [png](assets/ma-blackboard.png) · [src](assets/src/ma-blackboard.mmd) | 05 §4 |
| Router 模式 | [ma-router.svg](assets/ma-router.svg) · [png](assets/ma-router.png) · [src](assets/src/ma-router.mmd) | 05 §5 |
| 上下文组成与预算 | [context-budget.svg](assets/context-budget.svg) · [png](assets/context-budget.png) · [src](assets/src/context-budget.mmd) | 06 §1 |
| 上下文治理策略 | [context-strategies.svg](assets/context-strategies.svg) · [png](assets/context-strategies.png) · [src](assets/src/context-strategies.mmd) | 06 §3 |
| 上下文常见失败 | [context-failures.svg](assets/context-failures.svg) · [png](assets/context-failures.png) · [src](assets/src/context-failures.mmd) | 06 §4 |
| 上下文爆炸成因与解法 | [context-explosion.svg](assets/context-explosion.svg) · [png](assets/context-explosion.png) · [src](assets/src/context-explosion.mmd) | glossary/context-explosion |
| Transformer 前向与自注意力 | [llm-transformer.svg](assets/llm-transformer.svg) · [png](assets/llm-transformer.png) · [src](assets/src/llm-transformer.mmd) | 03 §9 |
| LLM 训练流程 | [llm-training.svg](assets/llm-training.svg) · [png](assets/llm-training.png) · [src](assets/src/llm-training.mmd) | 03 §3 |
| 推理 prefill / decode / KV Cache | [llm-inference.svg](assets/llm-inference.svg) · [png](assets/llm-inference.png) · [src](assets/src/llm-inference.mmd) | 03 §9.6 |
| LLM 在 Agent 中的位置 | [llm-in-agent.svg](assets/llm-in-agent.svg) · [png](assets/llm-in-agent.png) · [src](assets/src/llm-in-agent.mmd) | 03 §6 · glossary/llm |

重新渲染：`mmdc -i assets/src/X.mmd -o assets/X.svg -b white` 与 `-o assets/X.png -s 2`（需 CJK 字体）。

---

## 图示素材（附录参考）

两张外部示意图**不是**本目录唯一骨架；主文以工程理解 + 上方自绘图为主。图保留作对照：

| 文件 | 内容 | 来源标注 |
|---|---|---|
| [assets/agent-chain-of-thought-overview.jpg](assets/agent-chain-of-thought-overview.jpg) | 七层能力 + 反馈 + 四特征 | 微信公众号「**雪峰大数据**」 |
| [assets/agent-complete-architecture.jpg](assets/agent-complete-architecture.jpg) | Planner→Reasoning→Loop→Reflection + Memory/Tool/MCP | 改编自小红书「**林纾**」思路 |

本仓**未声称自绘**上述原图。

---

## 与本仓后续章节的挂钩（一句）

| Fundamentals 概念 | 本仓落地章 |
|---|---|
| 编排 / 阶段状态机 | [LangGraph](../02-frameworks/langgraph.md)、Aurora 战役图 |
| 会话内核 / tool loop / 压缩 | [pi](../02-frameworks/pi.md)、[dsh](../02-frameworks/dsh.md)、[context-compaction](glossary/context-compaction.md) |
| 工具适配 / 协议 | [LangChain](../02-frameworks/langchain.md)、[MCP](glossary/mcp.md) |
| 门禁 / HITL | [guardrails-hitl](glossary/guardrails-hitl.md)、Aurora 四层门禁 |
| 质量 / decide 螺旋 | YoungFlow / VulnHunter |
| LLM / 推理引擎 | [03-large-language-models](03-large-language-models.md) · [llm](glossary/llm.md) |
| 检索增强 | [rag](glossary/rag.md)（与 Memory 区分）· [vector-database](glossary/vector-database.md) |
| 上下文治理 | [06-context-engineering](06-context-engineering.md) · [context-explosion](glossary/context-explosion.md) |
| 多 Agent 协作 | [05-multi-agent-collaboration](05-multi-agent-collaboration.md) |

---

## glossary 说明

每篇统一结构：**定义 / 解决什么 / 怎么用 / 技术实现原理 / 常见坑 / 本仓关联**。索引含中英对照，见 [glossary/README.md](glossary/README.md)。
