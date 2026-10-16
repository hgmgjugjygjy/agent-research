# strix · 代码时序（代码视角）

> 依据：`01-competitors/strix.md` + `_trees/strix.json`。

```mermaid
sequenceDiagram
  participant CLI as CLI/factory
  participant Coord as AgentCoordinator
  participant Root as Root Scan Agent
  participant Comp as llm/compaction
  participant Child as Sub Agent
  participant Sand as Docker sandbox
  participant Gate as budget+coverage gates
  participant Rep as report/state

  CLI->>Coord: 注册 Root running
  CLI->>Root: 注入 scope.jinja + skills
  loop 扫描回合
    Root->>Comp: maybe_compact(session)
    Comp-->>Root: checkpoint 摘要或跳过
    Root->>Sand: 工具调用 HTTP/Browser/Shell
    Sand-->>Root: 观察结果
    opt 派生专家
      Root->>Coord: create_agent
      Coord->>Child: 异步 running + mailbox
      Root->>Coord: wait_for_agents 可选
      Child->>Sand: 专家工具循环
      Child->>Coord: agent_finish + 漏洞 id
      Coord->>Root: mailbox wake.set
    end
    Root->>Gate: 检查 budget_paused?
    alt 预算耗尽
      Gate-->>Coord: pause 或 stop 策略
    end
  end
  Root->>Gate: finish_coverage_gate
  Gate-->>Root: 覆盖完整才允许 finish_scan
  Root->>Rep: 落盘发现与 coverage 产物
  Root->>Coord: finish_scan → completed
```

## 说明

1. 调度语义：**mailbox + wake + wait_for_agents**（默认 300s，禁止同 turn 重复 wait）。
2. 终态：completed/stopped/crashed/failed/budget_*；非交互终态不可再投递。
3. 压缩切点保证 tool_call/result 成对；overflow 识别 LiteLLM/OpenRouter 400。
4. 门禁主轴是 **授权 scope + 沙箱 + 预算 + 覆盖完成门**，不是逐条 bash 审批。
5. 可 snapshot 恢复预算暂停会话。
