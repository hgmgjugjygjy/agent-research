# 框架对比 · 代码模块拓扑（代码视角）

> 依据：`02-frameworks/pi-dsh-langchain-langgraph.md`。Harness vs 编排运行时选型图。

```mermaid
flowchart TB
  subgraph control["控制面 自研"]
    CTRL["任务 / 资产 / 授权 / 配额"]
  end

  subgraph langgraph["LangGraph 战役编排"]
    LG["StateGraph\ncheckpointer + Store"]
    INT["interrupt / interrupt_before\nCommand resume"]
    MOD["模式: linear DAG\nfeedback 回流\nsupervisor 星形"]
    AUR["参考实现: AuroraRecon\norchestrator.py"]
  end

  subgraph harness["会话执行 Harness 二选一"]
    subgraph pi["pi earendil-works"]
      PI_LOOP["AgentHarness lane+op SM"]
      PI_COMP["工业级 compaction"]
      PI_TREE["SessionTree JSONL 审计"]
      PI_EXT["extensions tools/skills/hooks"]
      FCS["落地样板: Full-Chain-Sentinel"]
    end
    subgraph dsh["dsh deepseek-harness"]
      CORD["Cordis Everything-is-Plugin"]
      SESS["core/session SessionEvent 日志"]
      TOOLS["core/tools 守卫执行管道"]
      LOOP["core/agent-loop Turn/Step"]
      POL["dsh-base sandbox+approval policy"]
      SEAM["capability seams fs/subprocess"]
    end
  end

  subgraph adapt["适配层"]
    LC["LangChain\n模型/工具/RAG 胶水"]
    LITE["或 LiteLLM / MCP"]
  end

  subgraph quality["质量子系统可选"]
    VH["VulnHunter Hunt→Falsify→Verify"]
  end

  subgraph memory["记忆子系统可选"]
    ARTEX["ARTEX 双图"]
    WM["Aurora AttackGraph"]
  end

  CTRL --> LG
  LG --> INT
  LG --> MOD
  AUR -.-> LG
  LG --> pi
  LG --> dsh
  pi --> LC
  dsh --> LC
  LC --> LITE
  PI_LOOP --> PI_COMP --> PI_TREE --> PI_EXT --> FCS
  CORD --> SESS --> LOOP
  CORD --> TOOLS --> POL --> SEAM
  CTRL --> quality
  CTRL --> memory
```

## 说明

1. **推荐组合**：LangGraph 管阶段与审批 interrupt；pi 或 dsh 管子会话 tool loop 与压缩；LangChain/LiteLLM 做适配。
2. pi：极简可扩展，默认弱多 Agent；压缩与会话树成熟（Sentinel 已验证嵌入）。
3. dsh：插件 OS + 会话日志为真相源；sandbox/approval 一等插件；学习曲线高。
4. LangChain：勿当战役调度器；HITL middleware 仍依赖 Graph checkpointer。
5. 论文 2608.23953：harness 收敛五要素，共同缺口 **external verifiability**——正好对接 VulnHunter 质量子系统。
6. 本图为选型拓扑，非单一仓库路径。
