# 框架对比 · 代码时序（代码视角）

> 依据：`02-frameworks/pi-dsh-langchain-langgraph.md`。推荐栈一次战役回合时序。

```mermaid
sequenceDiagram
  participant Ctrl as 控制面 任务/授权
  participant LG as LangGraph 战役图
  participant CP as Checkpointer
  participant Gate as 门禁节点 scope/safety/HITL
  participant Har as pi或dsh 会话 harness
  participant Comp as compaction/projection
  participant LLM as LangChain或LiteLLM
  participant Tool as 沙箱工具/MCP
  participant Qual as 可选 VulnHunter Verify

  Ctrl->>LG: 创建战役 thread + autonomy/scope
  LG->>CP: 编译图并持久化初始 state
  loop 阶段节点
    LG->>Gate: 进入阶段前确定性检查
    alt interrupt_before 审批
      Gate->>CP: 挂起
      Ctrl->>LG: 人工 approve → Command resume
    end
    LG->>Har: 派生/复用会话执行子目标
    Har->>Comp: 超窗则压缩或日志投影
    Har->>LLM: model 流式
    LLM->>Tool: tool_call
    Note over Tool: dsh: tools pre/execute/post\npi: extension hooks / 产品审批档
    Tool-->>Har: observation
    Har-->>LG: 结构化结果写回 graph state
    opt 失败回流 feedback/supervisor
      LG->>LG: 重路由或重规划
    end
  end
  opt 白盒质量子系统
    LG->>Qual: 对 survivors 独立 Verify
    Qual-->>Ctrl: disposition verdict
  end
  LG->>Ctrl: 战役完成 + 审计轨迹
```

## 说明

1. **Graph 管阶段与 HITL**；**Harness 管单会话 tool loop/压缩/审计日志**——职责分离。
2. dsh Turn 流：`turn/start → pre-step → request → tool/call* → turn/end`，瀑布须 `next()`。
3. pi：lane + operation 状态机；overflow 时 compact-and-retry。
4. 门禁应尽量落在 Graph 确定性节点或 harness 策略插件，避免只靠 prompt。
5. 质量 Verify 应 **独立会话**，避免与挖洞 Agent 共享可写工具面。
