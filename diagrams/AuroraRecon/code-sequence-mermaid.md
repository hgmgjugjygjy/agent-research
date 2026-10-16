# AuroraRecon · 代码时序（代码视角）

> 依据：`01-competitors/AuroraRecon.md` + `_trees/AuroraRecon.json`。

```mermaid
sequenceDiagram
  participant API as tasks.py + task_runner
  participant Mem as engagement_memory
  participant Orch as orchestrator.build_graph
  participant Node as stage Agent node
  participant Scope as scope_guard
  participant Safe as safety_gate
  participant Irr as irreversible_gate
  participant CP as Postgres checkpointer
  participant UI as ApprovalComposer
  participant Art as findings+MinIO

  API->>Mem: 加载 PriorIntel 脱敏注入
  Mem->>Orch: runtime_facts + attack_graph prior
  API->>Orch: 编译图 mode=linear|feedback|supervisor
  Orch->>CP: 绑定 AsyncPostgresSaver
  loop 阶段推进
    Orch->>Node: 进入 recon/vuln/exploit/...
    Node->>Scope: 解析目标 vs authorized_scope
    alt 越界
      Scope-->>Node: BLOCK
    else 在范围内
      Node->>Safe: YAML 黑/灰名单
      alt BLOCK
        Safe-->>Node: 拒绝
      else 通过/WARNING
        Node->>Irr: 可逆?
        alt 不可逆且需审批
          Irr->>CP: checkpoint interrupt
          CP->>UI: human_approval
          UI-->>API: approve / reject
          API->>Orch: Command resume
        else 可逆或全自动允许
          Node->>Node: 工具执行 + retry_node
          Node->>Art: 写 finding/evidence
        end
      end
    end
    opt feedback/supervisor
      Node->>Orch: 回流 supervisor 或重规划
    end
  end
  Orch->>Art: report 阶段汇总
```

## 说明

1. 入口先注入 **跨任务 prior**，再编译 Kill-Chain 图。
2. 每次工具前过 **scope → safety YAML → irreversible**；LLM 不能覆盖 YAML 黑名单。
3. 不可逆动作走 **interrupt + UI 审批 + resume**；autonomous 仅自动放行可逆类。
4. checkpointer 使取消/恢复/聊天续跑成为一等能力。
5. 产物：AttackGraph 状态、findings、MinIO 证据对象。
