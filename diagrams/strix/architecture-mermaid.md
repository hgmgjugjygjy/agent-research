# Architecture (mermaid 备用 · 待 Archify render)

```mermaid
flowchart TB
  CLI --> Runner --> Coord[AgentCoordinator]
  Coord --> Root
  Root --> Child
  Root --> Compact
  Child --> Sandbox
  Coord --> Artifacts[(strix_runs)]
```
