# AuroraRecon · 代码模块拓扑（代码视角）

> 依据：`01-competitors/AuroraRecon.md` + `_trees/AuroraRecon.json`；仅模块/调用关系。不含利用步骤。

```mermaid
flowchart TB
  subgraph entry["入口"]
    API["backend/api/routers/tasks.py\ncreate/cancel/abort/approve/resume"]
    RUN["backend/api/services/task_runner.py"]
    FE["frontend Vue3\nApprovalComposer / PipelineFlow\nAttackGraphView"]
  end

  subgraph orch["编排 LangGraph"]
    ORCH["agents/orchestrator.py\nbuild_graph linear|feedback|supervisor"]
    SUP["agents/supervisor.py"]
    REACT["agents/react_base.py"]
    CHAIN["agents/chain_templates.py"]
  end

  subgraph stage["阶段 Agent"]
    RECON["recon_agent.py"]
    VULN["vuln_agent.py"]
    EXP["exploit_agent.py"]
    POST["post_agent.py"]
  end

  subgraph world["世界模型 / 记忆"]
    WM["world_model.py AttackGraph"]
    MEM["engagement_memory.py PriorIntel"]
    FACT["fact_hooks.py"]
  end

  subgraph gates["分层门禁 LLM不可覆盖"]
    SCOPE["scope_guard.py\nauthorized_scope"]
    SAFE["safety_gate.py\nsafety_rules.yaml"]
    IRR["irreversible_gate.py\n可逆/不可逆"]
    SKG["skills/guard.py"]
    AUTO["autonomy_level\n手动/监督/全自动"]
  end

  subgraph verify["验证 / 过滤"]
    FV["finding_verifier.py"]
    EV["evidence_verifier.py"]
    DF["detection_filter.py"]
  end

  subgraph tools["工具协调"]
    DSC["tools/deep_scan_coordinator.py"]
    DIR["tools/dir_scan_orchestrator.py"]
    PLAN["tools/tool_coverage_planner.py"]
  end

  subgraph persist["持久化"]
    CP["AsyncPostgresSaver / MemorySaver"]
    REDIS["Redis Stream v2 WebSocket"]
    MINIO["MinIO 证据"]
  end

  FE --> API --> RUN --> ORCH
  ORCH --> SUP
  ORCH --> RECON & VULN & EXP & POST
  ORCH --> CP
  ORCH --> WM
  RUN --> MEM --> WM
  RECON & VULN & EXP --> REACT
  REACT --> SCOPE --> SAFE --> IRR
  IRR --> AUTO
  SKG --> SAFE
  EXP --> FV & EV
  ORCH --> DF
  ORCH --> DSC & DIR & PLAN
  RUN --> REDIS
  EV --> MINIO
  CHAIN --> ORCH
```

## 说明

1. **三种图模式**由 `ATTACK_CHAIN_MODE` 切换：`_build_graph_linear|feedback|supervisor`。
2. HITL：`interrupt_before=["human_approval","post_foothold_approval"]` + Postgres checkpointer。
3. 门禁四件套：**scope_guard → safety_gate YAML → irreversible_gate → autonomy_level**，确定性规则优先于 LLM。
4. 世界模型 `AttackGraph` 承载 host/service/finding/credential…；跨任务 `PriorIntel` 只注存在性不注明文。
5. Skill 引擎：确定性步骤优先，失败再 ReAct；草案需人工 promote。
6. 急停 `POST .../abort` + 容器清理；节点 `@retry_node` 最多 3 次。
