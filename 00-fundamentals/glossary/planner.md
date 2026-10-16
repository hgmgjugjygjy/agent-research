# Planner（规划器）

> 相关：[planning.md](planning.md) · [orchestration.md](orchestration.md) · [agent-loop.md](agent-loop.md) · [task-understanding.md](task-understanding.md)

---

## 定义

**Planner** 是把用户/系统**目标**转换成**可执行步骤序列或阶段图**的模块（或提示策略）。输出可以是 todolist、DAG、LangGraph 节点序列，或「先检索再写作」一类粗计划。

---

## 解决什么

- 目标太大，不能一步 tool 打穿  
- 需要并行、依赖、验收标准  
- 失败后要**重规划**而不是原地瞎试

---

## 怎么用

1. 先澄清约束（时限、权限、禁止动作）——见 Task Understanding  
2. 产出带验收条件的步骤；标出需 HITL 的步  
3. 执行中根据 Reflection / 门禁结果触发 **replan**  
4. 复杂战役：Planner 产出交给 [Orchestration](orchestration.md) 持久化，而不是只存在聊天里

---

## 常见坑

- 计划过细且无反馈 → 一变全废  
- 计划过粗 → Loop 无方向空转  
- 把 Planner 和「会写步骤的 prompt」等同，却无状态持久化

---

## 技术实现原理

**输出结构**：让 LLM 按 Schema 输出计划，`Plan{steps: [Step{id, description, depends_on: [id], tool_hint, done_criteria}]}`，形成 DAG。

**流程**：目标 + 可用工具列表 + 约束 → Planner prompt → 结构化计划 → 拓扑排序 → 无依赖步骤可并行执行 → 每步结果回到 Replanner 判断是否修改剩余计划。

```python
plan = llm(PLANNER_PROMPT.format(goal=goal, tools=tool_list), response_format=Plan)
for batch in topo_batches(plan.steps):
    results |= parallel(execute_step(s) for s in batch)
    plan = replan(goal, plan, results)
```

**为什么**：显式计划可被人审阅、可预估成本、可并行；Schema 化便于代码而非模型来驱动调度。

## 本仓关联

Aurora / YoungFlow 等把「阶段」做进图；ARTEX Planner–Worker 分工。选型问：计划是否可序列化、可审计、可中断恢复。
