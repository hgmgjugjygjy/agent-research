# Diagrams

每个竞品目录含 Archify typed JSON IR 与已渲染 HTML（showcase）：

| 文件 | 说明 |
|------|------|
| `architecture.json` / `.html` | 系统架构图 |
| `sequence.json` / `.html` | 调度时序图 |
| `dataflow.json` / `.html` | 上下文/数据流图 |
| `*-mermaid.md` | mermaid 备用（未作为正式交付） |

**渲染状态（2026-10-16）**：**18/18 已 render**（`finalize --quality showcase`）。

CLI：`node /tmp/archify/archify/bin/archify.mjs`

| 项目 | architecture | sequence | dataflow |
|------|:------------:|:--------:|:--------:|
| ARTEX | ✅ | ✅ | ✅ |
| strix | ✅ | ✅ | ✅ |
| AuroraRecon | ✅ | ✅ | ✅ |
| vulnhunter | ✅ | ✅ | ✅ |
| Full-Chain-Sentinel-System | ✅ | ✅ | ✅ |
| DeepAudit | ✅ | ✅ | ✅ |

详见 [RENDER-STATUS.md](./RENDER-STATUS.md)。

重新渲染：

```bash
ARCHIFY="node /tmp/archify/archify/bin/archify.mjs"
for proj in ARTEX strix AuroraRecon vulnhunter Full-Chain-Sentinel-System DeepAudit; do
  for typ in architecture sequence dataflow; do
    $ARCHIFY finalize "$typ" "diagrams/$proj/$typ.json" "diagrams/$proj/$typ.html" --quality showcase --json
  done
done
```
