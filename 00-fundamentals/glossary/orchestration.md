# Orchestration（编排 / 状态机）

> 相关：[agent-loop.md](agent-loop.md) · [planner.md](planner.md) · [guardrails-hitl.md](guardrails-hitl.md) · [multi-agent.md](multi-agent.md)  
> 框架：[../../02-frameworks/langgraph.md](../../02-frameworks/langgraph.md)

---

## 定义

**Orchestration** 是在「单次会话热循环」之上，用**图、状态机或工作流**管理：阶段顺序、分支、并行、重试、持久化状态、跨 Agent 派发与中断恢复。LangGraph 一类框架是其常见实现；本质是**显式状态 + 显式转移**。

---

## 解决什么

- 多阶段战役（侦察→分析→报告）不能靠无限 ReAct 硬撑  
- 需要 checkpoint、从失败节点恢复  
- 需要在**节点边界**插入 HITL，而不是淹没在 token 流里  
- 多 Agent 时要有「谁先谁后、如何汇合」

---

## 怎么用

1. 把业务阶段建成图节点；边表示转移条件  
2. 状态 schema 明确（Pydantic/TypedDict 等）  
3. 长副作用节点配 timeout / 补偿  
4. `interrupt` / 人工节点：对外发送、高风险操作  
5. 节点**内部**可再跑 Agent Loop（编排套会话）

---

## 常见坑

- 用纯聊天 Agent「假装」有阶段，实际无持久化  
- 图过细变成难维护的流程图地狱  
- 编排与会话职责反转：编排里塞 prompt 细节、会话里塞全局拓扑

---

## 本仓关联

**优先 LangGraph 做战役编排**；pi 做阶段内会话。Aurora 的 Kill-Chain 图、YoungFlow 的 flow runner 都是编排样板。详见框架对比章。
