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
2. **[02-agent-architecture.md](02-agent-architecture.md)** — 工程架构走读（含自写 Mermaid；外部示意图作附录参考）  
3. **[glossary/](glossary/README.md)** — 关键名词专篇（含 RAG、编排、门禁、多 Agent、上下文压缩等）

不必一次读完所有名词；遇到主文加粗词时点进专篇即可。

---

## 本目录结构

```
00-fundamentals/
  README.md                 ← 本页
  01-what-is-an-agent.md
  02-agent-architecture.md
  glossary/                 ← 关键名词专篇（英文文件名）
  assets/                   ← 外部示意图参考（非本仓自绘）
```

---

## 图示素材（附录参考）

两张外部示意图**不是**本目录唯一骨架；主文以工程理解 + 自写 Mermaid 为主。图保留作对照：

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
| 检索增强 | [rag](glossary/rag.md)（与 Memory 区分） |

---

## glossary 说明

每篇统一结构：**定义 / 解决什么 / 怎么用 / 常见坑 / 本仓关联**。索引含中英对照，见 [glossary/README.md](glossary/README.md)。
