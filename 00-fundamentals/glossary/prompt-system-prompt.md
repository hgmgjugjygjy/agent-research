# Prompt / System prompt（提示词）

> 相关：[reasoning.md](reasoning.md) · [chain-of-thought.md](chain-of-thought.md) · [guardrails-hitl.md](guardrails-hitl.md)

---

## 定义

- **Prompt**：送给模型的输入文本（含用户话、工具结果、检索摘录、指令）。  
- **System prompt**：通常优先级最高的角色/规则层，定义身份、口吻、硬约束与工具使用方针。

---

## 解决什么

- 固定行为边界与输出格式  
- 注入任务局部指令而不改代码  
- 与 tool schema、RAG 摘录共同构成「可工作上下文」

---

## 怎么用

1. 系统层写清：角色、禁止事项、输出契约  
2. 业务可变部分用模板变量；密钥永不进提示  
3. 硬安全靠 [门禁](guardrails-hitl.md)，提示词只作辅助  
4. 版本管理 system prompt；变更可回滚

---

## 常见坑

- 用超长 system prompt 代替编排与记忆  
- 提示词与 tool 描述互相矛盾  
- 把用户可控内容拼进 system 层导致注入

---

## 本仓关联

各框架均依赖提示策略；评估竞品时看「提示 + 状态机 + 门禁」是否分层，而不是只抄一段 system 文案。
