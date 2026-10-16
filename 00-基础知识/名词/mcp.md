# MCP（Model Context Protocol）

> 图位：完整架构右侧 · Tool 的统一连接层 · [架构图](../assets/agent-complete-architecture.jpg)  
> 相关：[tool.md](tool.md) · [execution.md](execution.md)

---

## 结论先行

**MCP（Model Context Protocol）** 是一种让 AI 应用用**统一协议**连接外部系统（检索、GitHub、Notion、数据库、Slack 等）的方式。图2将其画在 Tool 外侧：Tool 是能力，MCP 是「怎么接上这些系统」的协议层。

价值：少写 N 套私有插件，多一份可插拔生态。

---

## 是什么

直观理解：

```
Agent 核心循环
    → Action 选「能力」
    → 经 MCP 连接到具体系统（GitHub / Notion / Slack / …）
    → 返回结果给 Reflection / 下一轮 Thinking
```

图中示例连接：Google 搜索、GitHub、Notion、数据库、Slack、其他系统。

MCP 不替代 Planner / Reasoning；它替代的是「每个 SaaS 一套一次性集成代码」的碎片化。

---

## 应用场景

- 同一 Agent 要同时用文档库 + 聊天通知 + 代码托管  
- 企业内多系统、希望权限与审计收口  
- 希望工具实现与编排内核解耦  

---

## 效益

| 效益 | 说明 |
|---|---|
| 可插拔 | 换系统不必重写编排 |
| 生态 | 社区 / 厂商可提供 MCP Server |
| 治理 | 连接层可统一鉴权与日志 |

---

## 注意

- MCP 解决「连接」，不自动解决「该不该调」（仍需门禁）  
- 协议与具体产品版本会演进，选型时核对官方规范与实现成熟度  
- 本调研文档不展开协议报文细节；产品落地以当时官方文档为准  

---

## 与本仓挂钩

目标架构适配层常见写法：**LiteLLM / LangChain + MCP + Skills + 沙箱运行时**。编排仍建议 LangGraph；会话执行参考 pi——MCP 挂在工具平面。
