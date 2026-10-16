# Dataflow (mermaid 备用 · 待 Archify render)

```mermaid
flowchart LR
  M[Messages] --> T[Tokens]
  T --> C[Compress]
  C --> S[Summary]
  S --> L[LLM]
```
