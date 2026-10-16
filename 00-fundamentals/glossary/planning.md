# Planning（规划能力层）

> 与 [planner.md](planner.md) 区分：本篇强调**能力**，Planner 强调**模块/角色**。

---

## 定义

**Planning** 指系统「会做计划」这一能力：分解目标、排序、设子目标、预留检查点。可以内嵌在同一 Agent 的提示里，也可外置为独立 Planner 服务。

---

## 解决什么

- 产品叙事上说明 Agent「不是乱试」  
- 评估竞品时判断其是否具备可观察的计划工件

---

## 怎么用

- 轻量：系统提示要求「先列计划再执行」  
- 重量：独立规划节点 + 人工可改计划  
- 与 [chain-of-thought](chain-of-thought.md) 配合：CoT 管一步内推理，Planning 管跨步结构

---

## 常见坑

- 文档写「具备规划」但运行时无计划对象可查  
- 与 Orchestration 混淆：有计划无状态机仍难做战役审批

---

## 技术实现原理

**常见规划算法族**：
- **一次性分解**（Plan-and-Solve）：先列步骤再执行。
- **交错式**（ReAct）：每步临时决定下一步。
- **层级分解**（HTN 思路）：目标→子目标→原子动作，递归展开。
- **搜索式**（Tree of Thoughts / LATS）：生成多个候选下一步，评估打分，按 BFS/DFS/MCTS 扩展最优分支；成本高，适合可评估的难题。

```python
def tot(state, depth):
    if depth == 0 or is_goal(state): return state
    cands = [llm_propose(state) for _ in range(3)]
    best = max(cands, key=llm_score)
    return tot(best, depth - 1)
```

**为什么**：任务越长、失败代价越高，越值得投入前置规划或搜索；简单任务用 ReAct 即可，避免过度规划。

## 本仓关联

读竞品时：既看有没有 Planner 模块，也看计划是否进入可持久化编排（LangGraph 等）。
