# Dataflow (mermaid 备用 · 待 Archify render)

```mermaid
flowchart LR
  P[Projection] --> C[Cut]
  C --> S[Summary]
  S --> E[CompactionEntry]
  C --> K[Kept]
```
