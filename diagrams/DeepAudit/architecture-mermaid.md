# Architecture (mermaid 备用 · 待 Archify render)

```mermaid
flowchart TB
  FE --> API --> Orch
  Orch --> Recon
  Orch --> Analysis
  Orch --> Verify
  Analysis --> Tools
  Orch --> Mem[MemoryCompressor]
  Orch --> PG[(Postgres)]
```
