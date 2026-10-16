# Diagrams

每个竞品目录含 Archify typed JSON IR（`architecture.json` / `sequence.json` / `dataflow.json`）及 mermaid 备用（`*-mermaid.md`）。

**状态**：本机未安装 Archify CLI，JSON 已按 `tt-a1i/archify` schema v1 编写，**待 Archify render** 生成 HTML/SVG。

渲染示例（环境就绪后）：

```bash
archify render architecture diagrams/ARTEX/architecture.json
archify render sequence diagrams/ARTEX/sequence.json
archify render dataflow diagrams/ARTEX/dataflow.json
```
