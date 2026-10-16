# Check & Correction（检查与修正）

> 与 [reflection.md](reflection.md) 同族：外部七层图常用「检查与修正」一词；本仓主口径用 Reflection，本篇作对照。

---

## 定义

对中间/最终结果做**自检**，发现偏差后**纠偏**（改参重试、回退到规划、请求人工）。即 Reflection 的操作化说法。

---

## 解决什么

- 反馈回路：失败 → 调整计划，而非直接放弃或硬交付错结果

---

## 怎么用

见 [reflection.md](reflection.md)；纠偏动作要有上限与预算。

---

## 常见坑

- 无限修正；或修正时丢掉原约束

---

## 技术实现原理

**流程**：产出 → 校验器 → 若失败则把错误信息作为反馈重试（上限 N 次）。校验器优先用**确定性检查**（JSON Schema、单元测试、编译、正则、引用存在性），其次用 LLM-as-judge 打分。

```python
for i in range(3):
    out = llm(task + feedback)
    errs = schema_check(out) + run_tests(out)
    if not errs: break
    feedback = f"\n上次输出的问题：{errs}，请修正。"
```

**为什么**：模型「自己说自己对」不可靠；外部可验证信号（测试/编译/Schema）才能形成真正的纠错闭环。重试上限防止无限循环与成本失控。

## 本仓关联

与 Reflection、Eval、门禁一起构成「质量 + 安全」双闭环。
