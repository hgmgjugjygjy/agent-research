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

## 技术实现原理

**执行器职责**：接收已校验的 tool_call → 在隔离环境运行 → 返回结构化结果。关键机制：
- **沙箱**：容器/microVM/受限子进程，限制文件系统、网络、CPU/内存。
- **超时与重试**：指数退避，只对幂等操作自动重试。
- **幂等键**：有副作用的调用带 `idempotency_key`，防止重试导致重复执行。
- **结果规范化**：`{ok, data, error, truncated}`，错误也作为观察回填给模型，让其自我修正。

```python
def execute(call):
    try:
        with timeout(30), sandbox(net=call.allow_net):
            data = REGISTRY[call.name](**call.args)
        return {"ok": True, "data": truncate(data)}
    except Exception as e:
        return {"ok": False, "error": type(e).__name__ + ": " + str(e)[:500]}
```

**为什么**：决策（模型）与执行（代码）分离，副作用集中在一处便于加权限、审计与回滚。

## 本仓关联

pi/dsh 的 tool 执行与 LangGraph 节点执行都属于本层；沙箱策略见 VH `sandbox_cfg` 一类设计。
