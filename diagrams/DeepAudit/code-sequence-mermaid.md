# DeepAudit · 代码时序（代码视角）

> 依据：`01-competitors/DeepAudit.md` + `_trees/DeepAudit.json`。

```mermaid
sequenceDiagram
  participant API as agent_tasks.py
  participant Orch as OrchestratorAgent
  participant GC as graph_controller
  participant Sub as Recon/Analysis/Verification
  participant Comp as memory_compressor
  participant CB as circuit_breaker+rate_limiter
  participant Tool as agent tools+sandbox
  participant CP as persistence checkpoint
  participant SSE as event_manager/SSE

  API->>Orch: 入队后台任务 校验路径/大小/exclude
  Orch->>GC: 注册 Agent 树
  Orch->>SSE: thinking/dispatch 事件
  loop Orchestrator 最多约20轮
    Orch->>Orch: 决策 dispatch_agent|summarize|finish
    alt dispatch
      Orch->>Sub: TaskHandoff 交接
      loop 子 Agent 迭代上限
        Sub->>Comp: compress_messages_if_needed
        Sub->>CB: 限流/熔断检查
        Sub->>Tool: Action 工具调用
        Tool-->>Sub: Observation
        Sub->>SSE: tool/finding 事件
        Sub->>CP: 按间隔 checkpoint
      end
      Sub-->>Orch: handoff 回传发现与建议
    else summarize/finish
      Orch->>GC: 汇总 findings
      Orch->>SSE: finish
    end
  end
  opt 用户停止
    API->>GC: request_stop / stop_all
  end
  Orch->>API: 任务终态 + 报告产物
```

## 说明

1. 调度是 **Orchestrator ReAct 派生子 Agent**，非子图 LangGraph（文档称自研 graph_controller）。
2. 每层有迭代上限；checkpoint 支持恢复查询。
3. 压缩在 BaseAgent 回合内触发，与熔断/限流并列。
4. 门禁偏资源与输入校验 + Verification 降误报，非进攻 bash 审批。
5. SSE 事件模型：thinking_token / tool / finding / dispatch 驱动前端。
