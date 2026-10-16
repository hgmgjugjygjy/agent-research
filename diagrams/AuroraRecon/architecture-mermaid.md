# Architecture (mermaid 备用 · 待 Archify render)

```mermaid
flowchart TB
  FE[Vue] --> API[FastAPI]
  API --> OR[Orchestrator LangGraph]
  OR --> AG[Agents]
  AG --> GATE[Scope/Safety/Irreversible]
  GATE --> TB[Toolbox]
  OR --> PG[(Postgres CP)]
  AG --> WM[(AttackGraph)]
```
