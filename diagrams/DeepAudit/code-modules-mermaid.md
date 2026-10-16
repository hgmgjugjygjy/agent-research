# DeepAudit · 代码模块拓扑（代码视角）

> 依据：`01-competitors/DeepAudit.md` + `_trees/DeepAudit.json`；SAST 向多 Agent。

```mermaid
flowchart TB
  subgraph entry["入口"]
    API["backend/app/api/v1/endpoints/\nagent_tasks.py + tasks.py"]
    FE["frontend React18 SSE"]
  end

  subgraph agents["services/agent/agents/"]
    ORCH["orchestrator.py\ndispatch_agent|summarize|finish"]
    RECON["recon.py 上限约15轮"]
    ANAL["analysis.py 上限约30轮"]
    VER["verification.py 上限约15轮"]
    BASE["base.py ReAct\nThought/Action/Observation"]
  end

  subgraph core["services/agent/core/"]
    GC["graph_controller.py\n停Agent/树视图/汇总"]
    REG["registry.py"]
    EXEC["executor.py"]
    ST["state.py + persistence.py\ncheckpoint"]
    CB["circuit_breaker.py"]
    RL["rate_limiter.py"]
    RT["retry.py + fallback.py"]
    VAL["validation.py"]
  end

  subgraph ctx["上下文"]
    MC["services/llm/memory_compressor.py\n100k×0.9 / 近15条"]
    CTX["core/context.py"]
  end

  subgraph tools["services/agent/tools/"]
    CODE["code_analysis_tool.py"]
    PAT["pattern_tool.py"]
    RAG["rag_tool.py"]
    EXT["external_tools.py Semgrep/Bandit/..."]
    SB["sandbox_tool.py\nsandbox_vuln.py\nsandbox_language.py"]
    FIN["finish_tool.py"]
    REP["reporting_tool.py"]
  end

  subgraph know["knowledge/"]
    FW["frameworks django/flask/..."]
    VULN["vulnerabilities injection/xss/..."]
    RAGK["rag_knowledge.py"]
  end

  subgraph stream["流式 / 遥测"]
    SH["streaming/stream_handler.py"]
    TR["telemetry/tracer.py"]
    EV["event_manager.py"]
  end

  subgraph db["存储"]
    PG["PostgreSQL agent_task\n+ agent_checkpoint 表"]
  end

  FE --> API --> ORCH
  ORCH --> RECON & ANAL & VER
  RECON & ANAL & VER --> BASE
  ORCH --> GC --> REG
  BASE --> EXEC --> tools
  BASE --> MC
  BASE --> CB & RL & RT
  ORCH --> ST --> PG
  tools --> know
  API --> SH & EV & TR
  VAL --> API
  SB --> VER
```

## 说明

1. 拓扑：**OrchestratorAgent → Recon / Analysis / Verification**，统一 `BaseAgent` ReAct。
2. `TaskHandoff` 结构化交接；`AgentGraphController` 管停树、用户指令、findings 汇总。
3. 压缩：`MemoryCompressor` 阈值总 token>100k×0.9，保留 system+近15，旧消息变 `<context_summary>`。
4. 工程门：熔断、令牌桶限流、指数退避、迭代 checkpoint、最大 findings。
5. Verification + **sandbox_*** 工具做误报抑制/PoC 级验证（审计场景）。
6. 缺 Aurora 式 scope/不可逆分级——主战场是读代码审计。
