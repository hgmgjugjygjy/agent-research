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

![Function calling 与 MCP 时序](../assets/tool-calling-mcp.svg)

*图：工具调用时序。运行时把工具 JSON Schema 交给模型 → 模型返回结构化 `tool_call` → 运行时校验并执行 → 结果以 `role=tool` 回填 → 模型继续。工具可来自本地注册，也可经 MCP 动态发现。PNG：[tool-calling-mcp.png](../assets/tool-calling-mcp.png)*

## 技术实现原理

**数据结构**：工具定义 `{name, description, parameters: JSON Schema}`；模型输出 `tool_call{id, name, arguments(JSON 字符串)}`；回填消息 `{role:"tool", tool_call_id, content}`。

**流程（function calling）**：
1. 注册：每个函数生成 JSON Schema（可由 Pydantic / 类型注解自动生成）。
2. 调用 LLM 时附带 `tools=[...]`；模型经过专门训练，在需要时输出结构化 tool_call 而非文本。
3. 运行时：解析 JSON → Schema 校验 → 权限/门禁检查 → 执行（超时、重试、沙箱）→ 结果截断/摘要后回填。
4. 循环直到模型不再请求工具。多个独立 tool_call 可并行执行。

```python
tools = [{"name": "get_weather", "description": "查询城市天气",
          "parameters": {"type": "object",
                         "properties": {"city": {"type": "string"}},
                         "required": ["city"]}}]
resp = llm(messages, tools=tools)
for call in resp.tool_calls:
    args = validate(json.loads(call.arguments), tools_by_name[call.name])
    out = REGISTRY[call.name](**args)
    messages.append({"role": "tool", "tool_call_id": call.id, "content": str(out)[:4000]})
```

**为什么**：让模型只负责「决定调用什么、参数是什么」，执行由确定性代码完成——可校验、可审计、可加权限；Schema 是模型与代码之间的契约。

## 本仓关联

LangChain 工具适配面广；pi/dsh 强调会话内 tool loop；MCP 提供协议化连接。平台要维护**工具白名单与风险分级**。
