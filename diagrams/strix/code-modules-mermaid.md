# strix · 代码模块拓扑（代码视角）

> 依据：`01-competitors/strix.md` + `_trees/strix.json`；仅模块/调用关系。

```mermaid
flowchart TB
  subgraph entry["入口 / UI"]
    CLI["CLI / pyproject 入口"]
    TUI["interface/tui Go Bubble Tea\nagents.go / agents_graph.go"]
    VIEW["interface/viewer\nAgentGraph / Transcript"]
    CLOUD["interface/cloud/session.py"]
  end

  subgraph core["strix/core 协调内核"]
    AG["core/agents.py\nAgentCoordinator\nstatuses/parent_of/mailbox/wake"]
    SESS["core/sessions.py"]
  end

  subgraph agents["strix/agents"]
    FACT["agents/factory.py"]
    PROMPT["agents/prompt.py\nprompts/scope.jinja"]
  end

  subgraph graphtools["多 Agent 工具面"]
    AGT["tools/agents_graph/tools.py\ncreate_agent / send_message\nwait_for_agents / agent_finish"]
    MCP["tools/mcp/agent_tools.py\nallowed_tools 白名单"]
    BR["tools/agent_browser/"]
  end

  subgraph llm["上下文"]
    COMP["llm/compaction.py\nmaybe_compact"]
    BUD["llm/context_budget.py"]
  end

  subgraph runtime["运行时 / 沙箱"]
    SM["runtime/session_manager.py"]
    DEF["runtime/deferred_session.py"]
    DOCKER["containers/ Docker 沙箱镜像"]
  end

  subgraph skills["skills/"]
    ROOT["coordination/root_agent.md"]
    VULN["vulnerabilities/*"]
    PROT["protocols/graphql.md"]
  end

  subgraph report["报告"]
    ST["report/state.py\ncoverage / finish gate"]
  end

  CLI --> FACT --> AG
  TUI --> AG
  VIEW --> AG
  CLOUD --> SESS
  AG --> AGT
  AG --> SESS
  FACT --> PROMPT
  FACT --> skills
  AGT --> AG
  AG --> COMP --> BUD
  AG --> SM --> DOCKER
  SM --> DEF
  MCP --> DOCKER
  BR --> DOCKER
  AG --> ST
```

## 说明

1. **AgentCoordinator**（`core/agents.py`）是图心脏：statuses / parent_of / mailbox / wake Event。
2. 子 Agent 经 `tools/agents_graph/tools.py` 派生；Root 用 `finish_scan`，子用 `agent_finish` 回父 inbox。
3. **安全压缩**：`compaction.py` 产出 `<conversation-checkpoint>`，模板强制保留 Findings/Credentials，禁止合并去重发现。
4. **预算门**：`budget_paused/stopped` + `test_budget_pause_policy.py`；完成门 `finish_coverage_gate`。
5. 执行面在 **Docker 沙箱** + MCP `allowed_tools`；无 Aurora 级不可逆分级默认链。
6. Skills 建议每子 Agent 挂 1–3 个专家技能。
