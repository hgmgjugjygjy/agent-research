# VulnHunter（Clouditera）· 代码时序（代码视角）

> Decide 螺旋：decide → 执行 stage → cycle_join → decide… → report。质量门 = gate + 双向论证 + 可选沙箱 POC。不含利用步骤。

```mermaid
sequenceDiagram
  participant UI as Web/API
  participant Svc as packages/service
  participant W as scan-worker Docker
  participant YF as YoungFlow orchestrator
  participant Dec as stage decide
  participant On as stage onboard
  participant Ex as map stages hunt/verify/…
  participant Gate as gate.yaml
  participant DecY as decision.yaml
  participant SB as sandbox-plane
  participant FS as output_dir 工作区

  UI->>Svc: 创建审计任务
  Svc->>W: 拉起 Worker + 挂载源码/flow
  W->>YF: youngflow flow.audit.yaml
  YF->>Dec: 进入 decide
  Dec->>FS: coverage / workspace_diff
  Dec->>DecY: 写 next + 派发 todo/*
  alt next=onboard
    YF->>On: map todo/onboard-*
    On->>SB: list/get/apply_sandbox（若动态）
    On->>Gate: next=continue|end
    alt end
      YF->>YF: route exit
    else continue
      YF->>YF: cycle_join → decide
    end
  else next=cognize|hunt|verify|…
    YF->>Ex: map 对应 todo 文件
    Ex->>FS: wiki/ADV/HYP/findings
    Note over Ex: verify=affirmer+challenger
    YF->>YF: cycle_join → decide
  else next=poc-verify|ev-assess|exp-build
    YF->>Ex: 动态阶段（读 sandbox_cfg）
    Ex->>FS: poc/exp 产物状态字段
    YF->>YF: cycle_join → decide
  else next=report
    YF->>FS: report/completion.yaml
  else next=exit
    YF-->>Svc: 异常收口
  end
  YF-->>Svc: 同步产物 / 状态
  Svc-->>UI: findings + 审核工作流
```

## 说明

1. 路由信号是 **文件**：`decision.yaml` / `gate.yaml`，不是 Claude Code skill 切换。
2. `cycle_join` 收口后回到 decide（`max_loops`）；`recursion_limit` 是 LangGraph 步进预算。
3. 动态阶段依赖 `enable_poc/exp/chain` + sandbox available；未配置 sandbox_cfg 时可本地执行（产品需自行收紧）。
4. 平台侧 findings 审核与 flow 内 verify **分层**：前者人审状态，后者论证裁决。
