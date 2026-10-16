# ARTEX · 代码时序（代码视角）

> 依据：`01-competitors/ARTEX.md` + `_trees/ARTEX.json`。调度/门禁视角。

```mermaid
sequenceDiagram
  participant API as server/orchestration
  participant Plan as agent/planner.go
  participant Todo as TodoStore
  participant Comp as compaction+coldgraph
  participant W as agent/worker.go
  participant G as guard/guard.go
  participant I as intercept ask
  participant DB as ExplorationStore+Assets
  participant Art as findings/archives

  API->>Plan: 任务启动 plannerLoop
  Plan->>DB: 读资产图+探索图前沿
  Plan->>Comp: OnPlannerRound maintain
  Comp-->>Plan: 冷戳/可选后台 digest
  Plan->>Todo: 更新跨轮 todolist
  Plan->>DB: 写入 0..N intent 进 frontier
  loop worker 并发
    W->>DB: claimNext 领取意图
    W->>G: PreToolUse hooks
    alt ask 规则
      G->>I: HandleAsk 人工审批
      I-->>G: 通过/拒绝/超时
      G-->>W: allow 或 block
    else deny/allow
      G-->>W: 即时裁决
    end
    W->>W: norma agentcore 工具循环
    W->>G: PostToolUse 归因 ok/blocked/error
    W->>DB: 写 fact/finding/hint 血缘
  end
  Plan->>Plan: debounce 图变更再唤醒
  Note over Plan,Art: prove_goal / 终态 → finding_workflow + 归档
  API->>Art: finding_workflow / task_archives
```

## 说明

1. 闭环驱动：**图变更 debounce → planner → frontier → worker**，非 LangGraph 节点边。
2. 门禁落在 **工具钩子**（Pre/PostToolUse），审批超时 fail-closed。
3. 压缩与调度同拍：每 planner 轮先 `maintain`，再可能后台 LLM 压冷子图。
4. 产物：探索图节点 + findings 工作流 + 可选 task archive 包。
5. worker 终态 `done/exhausted/blocked`；planner 可 `kill_work/steer_work`。
