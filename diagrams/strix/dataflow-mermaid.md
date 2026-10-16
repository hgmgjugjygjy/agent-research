# Dataflow (mermaid 备用 · 待 Archify render)

```mermaid
flowchart LR
  S[Session] --> B[Budget]
  B --> Sum[Summary LLM]
  Sum --> CK[Checkpoint]
  S --> R[Recent]
```
