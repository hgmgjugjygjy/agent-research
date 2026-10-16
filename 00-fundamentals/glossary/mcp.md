# MCP（Model Context Protocol）

> 相关：[tool.md](tool.md) · [guardrails-hitl.md](guardrails-hitl.md)

---

## 定义

**MCP（Model Context Protocol）** 是一种让 LLM 应用以**统一协议**连接外部系统（数据源、工具、上下文服务）的开放协议思路：服务端暴露资源/工具，客户端（Agent 宿主）发现并调用。相对「每个集成手写一套插件 API」，MCP 强调标准化发现与调用。

---

## 解决什么

- 降低 N×M 集成成本（多个宿主 × 多个外部系统）  
- 让工具与上下文服务可插拔  
- 便于在宿主侧统一做鉴权与审计切面

---

## 怎么用

1. 外部系统侧实现 MCP server（工具与资源）  
2. Agent 宿主做 client：列举工具 → 映射进 Function calling  
3. **门禁仍在宿主**：协议不等于安全；调用前照样 scope / HITL  
4. 凭证留在 server/宿主密钥管理，不进提示词

---

## 常见坑

- 误以为「接了 MCP 就安全」  
- 把全部 server 工具无过滤暴露给模型  
- 协议版本/能力协商不一致导致运行期失败

---

## 本仓关联

架构图侧翼的「MCP」表示协议化连接器；落地时与 LangChain 工具、自研插件并列评估。本仓竞品分析不依赖某一 MCP 发行版细节。
