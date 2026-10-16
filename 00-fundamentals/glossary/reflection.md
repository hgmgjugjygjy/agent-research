# Reflection（反思）

> 相关：[check-and-correction.md](check-and-correction.md) · [evaluation-trace.md](evaluation-trace.md) · [agent-loop.md](agent-loop.md) · [planner.md](planner.md)

---

## 定义

**Reflection** 是在行动之后，对结果是否满足目标/约束做**检查**，并决定：继续 Loop、重规划、请求人工、或交付。可以是同一模型的自评，也可以是独立审查 Agent / 规则集 / 测试套件。

---

## 解决什么

- 单轮成功≠任务成功  
- 发现遗漏、矛盾、策略违规  
- 把「失败」变成可回流的信号，而不是静默烂结果

---

## 怎么用

1. 定义可检验的验收条件（清单、测试、评分阈值）  
2. 在阶段末或关键 tool 后插入反思节点  
3. 失败分类：可重试 / 需重规划 / 需 HITL / 硬失败  
4. 与 [Eval/Trace](evaluation-trace.md) 结合：反思结论要落盘

---

## 常见坑

- 无标准的「你觉得好不好」→ 自我感觉良好  
- 反思死循环（一直不满意）无上限  
- 用反思代替门禁（安全问题应硬拦）

---

## 本仓关联

VulnHunter / YoungFlow 的 decide 螺旋、双向论证，是 Reflection/Eval 产品化形态；Aurora 阶段门与之互补（一偏质量论证，一偏安全门禁）。
