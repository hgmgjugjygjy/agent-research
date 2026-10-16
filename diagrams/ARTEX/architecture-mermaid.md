# Architecture (mermaid 备用 · 待 Archify render)

```mermaid
flowchart TB
  UI[Next.js] --> API[Go server]
  API --> ENG[Engine]
  ENG --> PL[Planner]
  ENG --> WK[Workers]
  WK --> GD[Guard]
  PL --> PG[(PostgreSQL 双图)]
  WK --> PG
  PL --> LLM[LLM]
  WK --> LLM
```
