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


## 代码视角 Mermaid（2026-10-16）

相对粗骨架 `*-mermaid.md` / Archify HTML 的**代码模块级**增补：目录/包名节点 + 门禁/沙箱/落盘契约。Archify 产物未删除。

| 项目 | 代码模块拓扑 | 代码时序 |
|------|:------------:|:--------:|
| ARTEX | [code-modules-mermaid.md](./ARTEX/code-modules-mermaid.md) | [code-sequence-mermaid.md](./ARTEX/code-sequence-mermaid.md) |
| strix | [code-modules-mermaid.md](./strix/code-modules-mermaid.md) | [code-sequence-mermaid.md](./strix/code-sequence-mermaid.md) |
| AuroraRecon | [code-modules-mermaid.md](./AuroraRecon/code-modules-mermaid.md) | [code-sequence-mermaid.md](./AuroraRecon/code-sequence-mermaid.md) |
| vulnhunter | [code-modules-mermaid.md](./vulnhunter/code-modules-mermaid.md) | [code-sequence-mermaid.md](./vulnhunter/code-sequence-mermaid.md) |
| Full-Chain-Sentinel-System | [code-modules-mermaid.md](./Full-Chain-Sentinel-System/code-modules-mermaid.md) | [code-sequence-mermaid.md](./Full-Chain-Sentinel-System/code-sequence-mermaid.md) |
| DeepAudit | [code-modules-mermaid.md](./DeepAudit/code-modules-mermaid.md) | [code-sequence-mermaid.md](./DeepAudit/code-sequence-mermaid.md) |
| frameworks (pi/dsh/LangChain/LangGraph) | [code-modules-mermaid.md](./frameworks/code-modules-mermaid.md) | [code-sequence-mermaid.md](./frameworks/code-sequence-mermaid.md) |

阅读建议：先看竞品 `01-competitors/*.md`，再用本表对照真实路径；**VulnHunter 已纠偏为 Clouditera**，时序为 YoungFlow decide 螺旋（非 capitalone Hunt→Falsify→Verify）。框架图含 YoungFlow。
