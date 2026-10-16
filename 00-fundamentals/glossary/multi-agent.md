# Multi-agent（多 Agent）

> 相关：[orchestration.md](orchestration.md) · [memory.md](memory.md) · [agent-loop.md](agent-loop.md)

---

## 定义

**Multi-agent** 指多个具有不同角色/工具集的 Agent 协作完成同一目标。常见拓扑：Orchestrator–Worker、流水线接力、评议/辩论（生成 + 审查）。

---

## 解决什么

- 单一巨型提示难维护时的职责拆分  
- 并行加速（多路检索/分析）  
- 用「审查者」角色降低单模型自我偏好

---

## 怎么用

1. 先定**共享状态**（黑板、图、工件目录），再定消息协议  
2. Orchestrator 负责任务分解与汇合；Worker 窄工具集  
3. 避免人人可写全局状态 → 冲突与污染  
4. 成本：多 Agent ≠ 更强，常更贵；有指标再拆

---

## 常见坑

- 「多智能体」只是串行 prompt 换人设，无隔离工具与状态  
- 无编排的群聊式互调 → 死锁/循环调用  
- 共享记忆无权限，审查者看不到完整证据

---

![多 Agent 协作（主管模式）](../assets/multi-agent.svg)

*图：Supervisor 主管模式。主管拆解并分派子任务给专家 Agent，结果写入共享状态，主管汇总；专家间也可直接 handoff。更多模式（流水线、对等协商、黑板、路由）见主文 [../05-multi-agent-collaboration.md](../05-multi-agent-collaboration.md)。PNG：[multi-agent.png](../assets/multi-agent.png)*

## 技术实现原理

**数据结构**：`AgentSpec{name, system_prompt, tools, model}`；消息/任务 `Task{id, parent_id, assignee, input, status, result}`；共享状态或 mailbox（每个 Agent 一个收件队列）。

**流程**：
1. 主管 LLM 以「可调用的子 Agent 列表」作为工具（`transfer_to_research(task)`），选择分派对象。
2. 子 Agent 在自己的上下文中跑 ReAct，只返回摘要结果（隔离上下文，避免主上下文爆炸）。
3. 并发 worker 从队列 `claim` 任务；状态机跟踪 pending/running/done/failed；父任务等待子任务（join）或级联取消。
4. 设预算：最大轮数、最大 token、最大并发。

```python
def supervisor(goal):
    while not done:
        d = llm(SUP_PROMPT, goal, board.summary(), tools=[to(a) for a in agents])
        if d.finish: return d.answer
        res = agents[d.target].run(d.task)   # 子上下文独立
        board.write(d.target, summarize(res))
```

**为什么**：专业化 prompt/工具集更聚焦、上下文隔离、可并行；代价是通信与协调成本，所以先证明单 Agent 不够再拆。

## 相关

- 五种协作模式详解：[../05-multi-agent-collaboration.md](../05-multi-agent-collaboration.md)

## 本仓关联

Strix、DeepAudit 提供多 Agent 协作样板；落地时仍建议 Orchestration 托底，而不是纯自由群聊。
