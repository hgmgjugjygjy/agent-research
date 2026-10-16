# Sequence (mermaid 备用 · 待 Archify render)

```mermaid
sequenceDiagram
  User->>Web: prompt
  Web->>Agent: turn
  Agent->>Gate: bash
  Gate->>User: approve?
  User->>Gate: allow
  Gate->>Kali: exec
```
