# Sequence (mermaid 备用 · 待 Archify render)

```mermaid
sequenceDiagram
  Op->>API: create task
  API->>G: run
  G->>CP: checkpoint
  G-->>Op: interrupt approval
  Op->>API: approve
  API->>G: resume
  G->>Ex: foothold
```
