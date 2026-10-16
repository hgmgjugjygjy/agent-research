# VulnHunter（Clouditera）· 代码模块拓扑（代码视角）

> 依据：Clouditera/VulnHunter 树 + `flows/vulnforge/flow.audit.yaml` + YoungFlow 子模块。  
> **纠偏**：非 capitalone Claude Code 三技能；主轴为 **flows → stages → decide → schemas/templates → sandbox**。不含利用步骤。

```mermaid
flowchart TB
  subgraph entry["入口 / 平台壳"]
    WEB["packages/web React/Vite"]
    SVC["packages/service Hono+WS+MCP"]
    SCH["workers/scheduler.ts\nscan-worker.ts"]
    ENT["packages/enterprise 商业增值"]
  end

  subgraph data["数据面"]
    PG[(PostgreSQL)]
    MIN[(MinIO)]
  end

  subgraph worker["Docker Worker"]
    YF["YoungFlow CLI\nsubmodules/youngflow"]
    PI["pi CLI + extensions"]
  end

  subgraph flows["flows/ 资产包"]
    VF["vulnforge/flow.audit.yaml"]
    VR["vulnhunter-report/flow.yaml"]
  end

  subgraph vulnforge["flows/vulnforge/"]
    DEC["stage id: decide\ntasks/decide.md"]
    STG["stages: onboard cognize research\nhunt verify poc-verify\nev-assess exp-build report"]
    SCHM["schemas/\ndecision.schema.yaml\ngate.schema.yaml\npoc/exp/cog/advice…"]
    TPL["templates/\npoc exp cog advice…"]
    SK["skills/* SKILL.md"]
    EXT["extensions/\noutput-contract\nsandbox-plane\ncoverage / workspace-diff"]
    MOD["models.json.example"]
  end

  subgraph sandbox["沙箱平面"]
    DEP["deploy/sandbox/\nconfig.yaml profiles.yaml"]
    SP["sandbox-plane extension\nlist/get/apply_sandbox"]
    CFG["flow_inputs.sandbox_cfg\n→ .sandbox_config"]
  end

  subgraph workspace["输出工作区"]
    TODO["todo/ leads/ done/"]
    DY["decision.yaml / gate.yaml"]
    KNOW["knowledge/wiki coverage build"]
    FIND["findings/BUG-*/"]
  end

  WEB --> SVC --> SCH
  SVC --> PG & MIN
  SCH --> YF
  YF --> VF & VR
  VF --> DEC --> STG
  VF --> SCHM & TPL & SK & EXT & MOD
  YF --> PI
  EXT --> SP
  SP --> DEP
  STG --> CFG
  DEC --> DY & TODO
  STG --> KNOW & FIND
  ENT -.-> SVC
```

## 说明

1. **YoungFlow** 跑 `flow.audit.yaml`；平台只负责任务/存储/Worker 生命周期。
2. **decide** 写 `decision.yaml.next` 做路由；**onboard** 写 `gate.yaml` 做环境门禁。
3. **sandbox_cfg** 经 sandbox-plane 申请；扩展不泄露沙箱地址/凭证。
4. schemas/templates 约束产物契约；skills 外置方法论。
5. capitalone 的 `vulnhunter-agent` / `/vulnhunt` skill **不在本仓**。
