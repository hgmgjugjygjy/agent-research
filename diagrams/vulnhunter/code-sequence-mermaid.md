# VulnHunter · 代码时序（代码视角）

> 依据：`01-competitors/vulnhunter.md` + 上游 phases / agent 树。Hunt→Falsify→Verify 为质量闭环。

```mermaid
sequenceDiagram
  participant CLI as runner.py / Claude Code
  participant Orch as vulnhunt/SKILL.md
  participant FS as 产物目录
  participant Hunt as Phase2 class agents
  participant Fal as phase2b_verify Falsify
  participant Cap as Capability Filter
  participant Fix as vulnhunter-fix
  participant Gates as run-gates/check-scope
  participant Ver as verify_runner 只读信封
  participant Disp as verify_disposition.json

  CLI->>Orch: 启动 /vulnhunt 或 headless scan
  Orch->>FS: 校验 Phase1 recon 产物存在
  Orch->>Hunt: Agent 派生并行 inj/nav/logic
  Hunt->>FS: 写候选 findings
  Orch->>Fal: 对抗证伪「为何打不通」
  Fal-->>FS: 淘汰弱候选约半数
  Orch->>Cap: 过滤真实攻击者能力
  Cap->>FS: survivors + 路径说明
  Note over Orch,FS: 编排器仍不持 findings 全文
  Orch->>FS: Phase3/3d/4 复现建议与报告

  opt 开发修复环 /vulnhunter-fix
    CLI->>Fix: 读 survivors
    Fix->>Gates: RED 测试门 + scope 门
    Fix->>Fix: GREEN 改码 / graph query
    Fix->>FS: patch / PR 材料
  end

  CLI->>Ver: 全新会话 + 只读工具信封
  Ver->>FS: 读 finding + patch
  Ver->>Disp: 写 per-finding verdict schema
  Note over Ver,Disp: 无 Bash / 无网络；防自卖自夸
```

## 说明

1. 阶段闸门靠 **产物文件存在** 推进，不是常驻服务状态机。
2. Phase2b **Falsify** 与 Capability Filter 是 Hunt 内硬质量门。
3. Fix 可选；脚本门防止空话修复（hand-wave guard）。
4. Verify **分 skill、分会话、只读信封** → `verify_disposition.json`。
5. Batch：`harness/local_harness/batch` 多仓 + `--resume`；扫描默认不改目标仓。
6. Sandbox 段与 `--enable-bash` 控制是否允许动态复现工具。
