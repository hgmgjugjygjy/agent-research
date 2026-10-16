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

## 技术实现原理

**输出结构**：把自由文本请求解析为 `TaskSpec{intent, entities/slots, constraints, success_criteria, missing_info[]}`。

**流程**：意图分类（规则/小模型/LLM 结构化输出）→ 槽位抽取 → 缺失必填槽位时生成澄清问题（而非猜测）→ 规范化后的 TaskSpec 交给 Planner。

```python
spec = llm(UNDERSTAND_PROMPT, user_msg, response_format=TaskSpec)
if spec.missing_info:
    return ask_user(spec.missing_info)
plan = planner(spec)
```

**为什么**：越早把模糊需求变成结构化规格，后续规划和验收越稳定；「问一次」比「做错再返工」便宜。

## 本仓关联

接入层就应做权限与范围归一；「理解」失败时 fail-closed，而不是乐观执行。
