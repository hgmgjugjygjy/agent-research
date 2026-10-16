# Execution（执行）

> 近亲：[agent-loop.md](agent-loop.md) · [tool.md](tool.md) · [guardrails-hitl.md](guardrails-hitl.md)

---

## 定义

**Execution** 指把已决定的 Action / 图节点**真正落地**：发起 I/O、产生中间结果或副作用，并把观察返回运行时。是 Loop 中「Action Input → 结果」的那一截。

---

## 解决什么

- 区分「模型打算做」与「系统已做」  
- 统一超时、重试、幂等、沙箱

---

## 怎么用

1. 执行器与推理器进程/权限分离更佳  
2. 所有执行走同一闸门：校验 → 门禁 → 调用 → 度量  
3. 可回放：输入参数与结果摘要入库

---

## 常见坑

- 在提示词里「假装已执行」  
- 无超时的阻塞 tool 卡死会话

---

## 本仓关联

pi/dsh 的 tool 执行与 LangGraph 节点执行都属于本层；沙箱策略见 VH `sandbox_cfg` 一类设计。
