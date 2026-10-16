# ARTEX · 代码模块拓扑（代码视角）

> 依据：`01-competitors/ARTEX.md` + `_trees/ARTEX.json`；仅模块/调用关系。不含利用步骤。

```mermaid
flowchart TB
  subgraph entry["入口"]
    CMD["cmd/artex 单体入口"]
    WEB["web/ Next.js 静态嵌入"]
  end

  subgraph server["server/ 控制面"]
    ORCH["server/orchestration.go"]
    SCH["server/scheduler.go\n+ db/schedules.go"]
    TCTL["server/task_control.go"]
    TINT["server/task_intercept.go"]
    FWF["server/finding_workflow.go"]
  end

  subgraph agentpkg["agent/ 调度内核"]
    PLAN["planner.go\n唯一意图生成者 + TodoStore"]
    WORK["worker.go xN\nclaimNext 领取意图"]
    MAIN["mainagent.go\n人在环路对话"]
    GOALS["goals.go 目标分解"]
    COMP["compaction.go\nOnPlannerRound"]
    COLD["coldgraph.go\n冷节点 digest"]
    TOOLS["tools.go + toolcatalog.go"]
    FIND["finding_recorder.go\nfinding_workflow.go"]
  end

  subgraph memory["双图记忆 PostgreSQL"]
    ASSET["db/task_assets.go\n资产图 root_domain→endpoint"]
    EXPL["db 探索图\ngoal/intent/fact/finding"]
    CTX["db/task_context.go"]
    SCOPE["db/task_scope.go\ndb/company_scope.go"]
  end

  subgraph gates["门禁"]
    GUARD["guard/guard.go\nPreToolUse/PostToolUse"]
    INTER["intercept/\nask→人工审批"]
  end

  subgraph runtime["运行时依赖"]
    NORMA["norma SDK\nagentcore/tool/permission/harness/memory"]
    LLM["llmpool/ 池化"]
    MCP["mcphttp/"]
  end

  CMD --> ORCH
  WEB --> ORCH
  ORCH --> PLAN
  ORCH --> WORK
  ORCH --> MAIN
  SCH --> ORCH
  TCTL --> ORCH
  PLAN --> GOALS
  PLAN --> COMP --> COLD
  PLAN --> EXPL
  WORK --> TOOLS
  WORK --> ASSET
  WORK --> EXPL
  TOOLS --> GUARD --> INTER
  TINT --> INTER
  FIND --> FWF
  WORK --> NORMA
  PLAN --> NORMA
  PLAN --> LLM
  ASSET --> SCOPE
  CTX --> EXPL
```

## 说明

1. **Planner–Worker**：`planner.go` 跨唤醒共享 `TodoStore`，串行派发利用链步骤；worker 经 `claimNext` 领取。
2. **双图**：资产图全局共享，探索图每任务独立，靠血缘边与 `exploration_anchors` 锚定。
3. **冷压缩**：`compaction.go` + `coldgraph.go` 在每 planner 轮打冷/热戳，阈值 N=20 / M=8 触发 LLM digest。
4. **门禁**：`guard/guard.go` allow/deny/ask；ask 失败或超时 block（平台管控文案）。
5. **日历调度**：`server/scheduler.go` + `db/schedules.go` 负责任务级定时，非 Agent 内循环。
6. 深度能力在外部 **norma SDK**；本仓可见层是 Go 封装与双图存储。
