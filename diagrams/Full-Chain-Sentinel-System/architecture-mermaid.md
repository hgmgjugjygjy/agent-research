# Architecture (mermaid 备用 · 待 Archify render)

```mermaid
flowchart TB
  Web --> CA[coding-agent]
  CA --> Harness
  Harness --> Compact
  Harness --> Gate
  Gate --> Kali
  Harness --> JSONL[(Session Tree)]
```
