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

## 本仓关联

Strix、DeepAudit 提供多 Agent 协作样板；落地时仍建议 Orchestration 托底，而不是纯自由群聊。
