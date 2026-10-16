# Memory（记忆）

> 相关：[rag.md](rag.md) · [context-compaction.md](context-compaction.md) · [agent-loop.md](agent-loop.md) · [orchestration.md](orchestration.md)

---

## 定义

**Memory** 让 Agent 在多步、多轮、甚至跨任务中保持一致性。常见三分法：

| 类型 | 存什么 | 生命周期 |
|---|---|---|
| **长期记忆** | 用户偏好、角色、跨任务先验 | 跨会话 |
| **短期记忆** | 近端对话与本任务上下文 | 本会话 / 本任务 |
| **工作记忆** | 已完成步骤、中间工件、进度勾选 | 本任务执行中 |

形式可以是：消息缓冲、KV、文件工件、**图结构**（资产图/探索图）、checkpointer 状态等。

---

## 解决什么

- 防止每步失忆、重复劳动  
- 支持中断续跑  
- 把「关键事实」从纯聊天记录升级为可查询结构

---

## 怎么用

1. 明确三类记忆的读写点（谁写、何时写、谁有权改偏好）  
2. 短记忆触顶 → [compaction](context-compaction.md)，不要只靠更大窗口  
3. 长记忆可编辑、可遗忘；涉密字段脱敏  
4. 与 [RAG](rag.md) 分家：进度/偏好走 Memory，文档知识走检索

---

## 常见坑

- 用向量库冒充工作记忆（「检索到旧任务」≠「知道做到哪」）  
- 无写入规范 → 垃圾记忆污染后续  
- 仅依赖聊天窗口，无外置结构

---

![Memory 分层](../assets/memory-layers.svg)

*图：记忆分层。新信息先进入工作记忆（上下文窗口）；超阈值触发压缩；抽取器把稳定事实写入长期记忆；需要时按相似度+时间+重要性召回。PNG：[memory-layers.png](../assets/memory-layers.png)*

## 技术实现原理

**数据结构**：
- 短期/工作记忆：`messages: list[{role, content, tool_calls?}]`，即每次调用 LLM 时发送的数组本身。
- 长期记忆：`MemoryItem{id, user_id, text, vector, type(fact|preference|episode), importance, created_at, last_access}`，存向量库 + KV。
- 情景记忆：历史任务轨迹（目标、步骤、结果），用于「上次怎么做的」。

**流程**：
1. 每轮结束后跑「记忆抽取」prompt：从对话中抽取「值得长期记住的事实」，并与已有记忆比对做 ADD / UPDATE / DELETE / NOOP（去重与冲突消解）。
2. 召回打分：`score = α·cos(q, m) + β·recency(m) + γ·importance(m)`（Generative Agents 论文中的经典三因子思路）。
3. 召回结果以「已知用户信息：…」注入系统提示，不混入工具结果。

```python
def remember(turn):
    for fact in llm_extract_facts(turn):
        old = ltm.search(embed(fact), k=3, user=uid)
        op = llm_decide(fact, old)        # ADD/UPDATE/DELETE/NOOP
        ltm.apply(op, fact)
```

**为什么**：上下文窗口有限且按 token 计费；把「当前任务状态」与「跨会话事实」分开存，前者精确、后者可检索；抽取+去重避免记忆库膨胀和自相矛盾。

## 相关

- 相关：[context-explosion.md](context-explosion.md) · [../06-context-engineering.md](../06-context-engineering.md) · [vector-database.md](vector-database.md)

## 本仓关联

ARTEX **双图 + 冷 digest**、Aurora **世界模型 / engagement_memory** 是 Memory 的产品级形态。选型先问「是否外置、是否可审计」，再问「有没有向量库」。
