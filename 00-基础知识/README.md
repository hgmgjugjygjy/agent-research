# 00 · 基础知识：Agent 入门

**受众**：产品 / 架构 / 安全工程（零 Agent 基础也可读）  
**定位**：本仓的**入门层**——先弄清「Agent 是什么、由哪些组成」，再读竞品与框架选型。  
**约束**：只讲概念与中性示例（竞品分析、查库、发 Slack）；**不含** exploit / PoC。

---

## 结论先行

| 要点 | 说明 |
|---|---|
| **Agent ≠ 一次问答** | 是「目标驱动 + 规划 + 工具行动 + 记忆 + 反思迭代」的闭环系统 |
| **两张总览图** | 图1讲「七层能力与反馈」；图2讲「端到端完整架构（含 MCP）」 |
| **读完本目录** | 再进入 `01-competitors/`、`02-frameworks/` 才有共同语言 |

---

## 阅读顺序（建议）

1. **[01-agent是什么.md](01-agent是什么.md)** — 定义、与聊天机器人差异、图1/图2讲解、与本仓选型关系  
2. **[02-agent完整架构.md](02-agent完整架构.md)** — 以图2为主线端到端走读（含中性示例）  
3. **[名词/](名词/README.md)** — 按需查阅 Planner、Memory、MCP、Reflection 等关键名词专篇  

不必一次读完所有名词；遇到主文加粗词时点进专篇即可。

---

## 本目录结构

```
00-基础知识/
  README.md                 ← 本页
  01-agent是什么.md
  02-agent完整架构.md
  名词/                     ← 关键名词专篇（独立短文）
  assets/                   ← 两张总览图（非本仓自绘）
```

---

## 图示素材与来源

| 文件 | 内容 | 来源标注 |
|---|---|---|
| [assets/agent-chain-of-thought-overview.jpg](assets/agent-chain-of-thought-overview.jpg) | AI 智能体思维链：工作原理总览（七层 + 反馈 + 四特征） | 微信公众号「**雪峰大数据**」 |
| [assets/agent-complete-architecture.jpg](assets/agent-complete-architecture.jpg) | AI Agent 的完整架构（Planner→Reasoning→核心循环→Reflection + Memory/Tool/MCP） | 改编自小红书「**林纾**」思路（动态面板复刻版） |

本仓**未声称自绘**上述两张原图；文中嵌入仅作教学讲解。可选 Mermaid 核心循环图为本仓自写，见主文。

---

## 与本仓后续章节的挂钩（一句）

| 基础知识概念 | 本仓落地章 |
|---|---|
| 编排 / 阶段状态机 | [LangGraph](../02-frameworks/langgraph.md)、Aurora 战役图 |
| 会话内核 / tool loop | [pi](../02-frameworks/pi.md)、[dsh](../02-frameworks/dsh.md) |
| 工具适配 | [LangChain](../02-frameworks/langchain.md)、MCP |
| Reflection / 门禁产品化 | Aurora 四层门禁、VulnHunter decide/gate |

---

## 名词篇是否合并？

用户要求「每个关键名词做专门解释」。本目录中 **Planner** 与 **Planning（规划能力层）**、**Reasoning** 与 **Chain of Thought** 等保持**独立成篇**，避免混为一谈；索引表见 [名词/README.md](名词/README.md)。篇幅均为短文（约 80–200 行量级）。
