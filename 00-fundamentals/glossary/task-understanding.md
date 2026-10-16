# Task Understanding（任务理解）

> 相关：[planner.md](planner.md) · [prompt-system-prompt.md](prompt-system-prompt.md) · [guardrails-hitl.md](guardrails-hitl.md)

---

## 定义

**Task Understanding** 是把用户自然语言或上游事件，解析为：**意图、对象、约束、成功标准、禁止事项**。常含澄清问句（缺槽则问）。

---

## 解决什么

- 避免一上来就 tool 乱打  
- 把模糊目标变成可规划输入  
- 及早识别越权或范围外请求

---

## 怎么用

1. 抽槽：目标、范围、截止时间、输出格式  
2. 缺关键约束 → 澄清或拒绝  
3. 输出结构化 Task Spec 再交给 Planner

---

## 常见坑

- 过度澄清骚扰用户  
- 静默脑补高风险约束（如默认可外发）

---

## 本仓关联

接入层就应做权限与范围归一；「理解」失败时 fail-closed，而不是乐观执行。
