# Full-Chain-Sentinel-System · 代码时序（代码视角）

> 依据：`01-competitors/Full-Chain-Sentinel-System.md` + pi 分包树。

```mermaid
sequenceDiagram
  participant UI as packages/web
  participant Gate as 授权门+gate token
  participant CA as coding-agent SessionTree
  participant Dur as durable op state
  participant Loop as agent-loop.ts
  participant Mode as 审批三档
  participant Kali as Kali docker exec
  participant Comp as compaction
  participant Log as JSONL session

  UI->>Gate: 首次勾选授权承诺
  Gate->>CA: 创建/恢复会话
  CA->>Dur: 绑定 operation 状态机
  UI->>CA: prompt
  CA->>Dur: 检查 Docker 可用否则拒绝
  loop turn
    CA->>Comp: contextTokens 超窗?
    alt 需压缩
      Comp->>Log: 写摘要 entry 保留近端
      Comp-->>CA: compact 完成
    end
    CA->>Loop: model 调用
    Loop->>Mode: 工具调用拦截
    alt 审批模式
      Mode->>UI: 展示可编辑命令卡
      UI-->>Mode: 放行/拒绝
    else AI 审批
      Mode->>Mode: 守门员 ALLOW/BLOCK
      Note over Mode: 失败/超时 fail-open 放行
    else 自动执行
      Mode-->>Loop: 直接执行
    end
    Mode->>Kali: docker exec bash/droid_shell
    Kali-->>Loop: stdout/stderr
    Loop->>Log: append transcript entries
  end
  opt /tree 切分支
    CA->>Comp: branch summarization
    Comp->>Log: 离开分支注入摘要
  end
  Note over Dur,Log: 崩溃后按 durable + JSONL 恢复
```

## 说明

1. 主抽象是 **会话树 + 工具审批**，不是 planner-worker 战役图。
2. 压缩与 overflow recovery 在工具循环内联触发。
3. 门禁档位是产品层叠在 harness 外；AI 档 fail-open 需自建平台时改掉。
4. 产物：JSONL 会话、工具卡事件、可选漏洞中枢记录。
5. 工程模式「指挥官」多任务仍属产品层队列，非 LangGraph。
