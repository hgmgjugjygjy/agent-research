# Tool / Function calling（工具与函数调用）

> 相关：[mcp.md](mcp.md) · [agent-loop.md](agent-loop.md) · [guardrails-hitl.md](guardrails-hitl.md) · [execution.md](execution.md)

---

## 定义

**Tool** 是 Agent 可调用的外部能力（检索、HTTP、数据库、Slack、Shell…）。**Function calling** 是让 LLM 以结构化方式（通常 JSON）提出「调用哪个函数、参数是什么」的机制；运行时校验后真正执行。

---

## 解决什么

- 突破模型纯文本边界，产生真实世界副作用或取数  
- 把能力模块化、可鉴权、可审计

---

## 怎么用

1. 为每个 tool 提供清晰 **name / description / JSON schema**  
2. 运行时：解析调用 → schema 校验 → [门禁](guardrails-hitl.md) → 执行 → 观察写回  
3. 幂等与超时；对外发送类默认 HITL  
4. 大批工具时做检索式 tool 选择，避免 schema 撑爆窗口

---

## 常见坑

- description 含糊 → 乱选工具  
- 无校验执行 → 注入参数  
- 只读与可写工具未分权  
- 把 MCP 服务器能力未映射成可门禁的 tool 面

---

## 本仓关联

LangChain 工具适配面广；pi/dsh 强调会话内 tool loop；MCP 提供协议化连接。平台要维护**工具白名单与风险分级**。
