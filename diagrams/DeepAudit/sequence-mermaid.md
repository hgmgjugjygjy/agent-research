# Sequence (mermaid 备用 · 待 Archify render)

```mermaid
sequenceDiagram
  User->>API: create
  API->>Orch: run
  Orch->>Ana: dispatch
  Ana-->>Orch: handoff
  Orch->>SSE: events
  SSE-->>User: stream
```
