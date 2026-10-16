# Full-Chain-Sentinel-System · 代码模块拓扑（代码视角）

> 依据：`01-competitors/Full-Chain-Sentinel-System.md` + `_trees/Full-Chain-Sentinel-System.json`。pi fork + 渗透壳。

```mermaid
flowchart TB
  subgraph entry["入口 / 产品壳"]
    WEB["packages/web 中文控制台\n流式对话·工具卡·会话树"]
    CLI["pi CLI / RPC / print-JSON"]
    GATE["首次授权承诺 + gate token"]
  end

  subgraph pi_core["pi monorepo @earendil-works"]
    AGENT["packages/agent\nagent-loop.ts / agent.ts"]
    AI["packages/ai\nproviders + transcript"]
    CA["packages/coding-agent\nsession tree · tools · compact"]
    DUR["packages/durable\noperation 状态机崩溃恢复"]
    CHORD["packages/chord\nstate codec / delta"]
    CM["packages/codemode\nruntime/worker.ts"]
  end

  subgraph compact["压缩栈 coding-agent/docs/compaction.md"]
    AUTO["Auto-compaction\nreserveTokens=16k keepRecent=20k"]
    BR["Branch summarization /tree"]
    OV["Overflow compact-and-retry"]
    HOOK["session_before_compact 扩展钩子"]
  end

  subgraph gates["三档执行模式"]
    APPR["审批模式 默认\nbash/droid_shell 可编辑放行"]
    AUTOEX["自动执行 高风险"]
    AIA["AI 审批 守门员小模型\n失败超时 fail-open"]
  end

  subgraph domain["渗透领域壳"]
    KALI["docker/ Kali 作业容器\ndocker exec bash"]
    DROID["安卓 DroidBridge"]
    NUC["Nuclei POC 订阅 / 漏洞中枢"]
  end

  subgraph ext["扩展缝"]
    EX["coding-agent/examples/extensions\nbash-spawn-hook / confirm-destructive\ncustom-compaction"]
    SK["docs/skills.md"]
  end

  WEB --> GATE --> CA
  CLI --> CA
  CA --> AGENT --> AI
  CA --> DUR
  CA --> CHORD
  CA --> CM
  CA --> AUTO & BR & OV & HOOK
  CA --> APPR & AUTOEX & AIA
  APPR --> KALI
  AUTOEX --> KALI
  AIA --> KALI
  WEB --> DROID & NUC
  EX --> CA
  SK --> CA
```

## 说明

1. 内核是 **pi harness**：prompt→model→tools→compact→resume；默认无内建 sub-agent/plan mode。
2. 产品差异在 **Web 壳 + Kali 容器 + 三档审批**；包名仍 `@earendil-works/*`。
3. 压缩工业文档在 `packages/coding-agent/docs/compaction.md`；会话 JSONL = 审计真相源。
4. 审批链覆盖 bash/droid_shell；**不覆盖** read/edit/write、MCP、抓包等直连工具（README 已声明）。
5. AI 审批 **fail-open**；容器 root、无 seccomp——非整体沙箱。
6. Docker 检查失败则拒绝 prompt/compact/持续目标。
