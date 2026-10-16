# Dataflow (mermaid 备用 · 待 Archify render)

```mermaid
flowchart LR
  A[activity] --> E[探索图]
  A --> S[资产图]
  E --> C[Compactor]
  C --> D[Digest]
  D --> P[Planner]
  S --> P
```
