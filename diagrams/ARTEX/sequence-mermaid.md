# Sequence (mermaid 备用 · 待 Archify render)

```mermaid
sequenceDiagram
  participant EV as 图变更
  participant P as Planner
  participant F as Frontier
  participant W as Worker
  participant G as Guard
  EV->>P: 唤醒
  P->>F: 派意图
  W->>F: claim
  W->>G: PreToolUse
  G-->>W: allow/ask/deny
  W->>EV: 写回触发
```
