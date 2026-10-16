# Archify Render Status

- **时间**：2026-10-16 14:32 CST (Asia/Shanghai)
- **CLI**：`node /tmp/archify/archify/bin/archify.mjs`（浅克隆 `/tmp/archify/archify`，Archify 3.1.0）
- **质量档**：`showcase`
- **结果**：**18/18 成功**，失败项无

## 成功清单（HTML 路径）

| 项目 | 类型 | HTML |
|------|------|------|
| ARTEX | architecture | `diagrams/ARTEX/architecture.html` |
| ARTEX | sequence | `diagrams/ARTEX/sequence.html` |
| ARTEX | dataflow | `diagrams/ARTEX/dataflow.html` |
| strix | architecture | `diagrams/strix/architecture.html` |
| strix | sequence | `diagrams/strix/sequence.html` |
| strix | dataflow | `diagrams/strix/dataflow.html` |
| AuroraRecon | architecture | `diagrams/AuroraRecon/architecture.html` |
| AuroraRecon | sequence | `diagrams/AuroraRecon/sequence.html` |
| AuroraRecon | dataflow | `diagrams/AuroraRecon/dataflow.html` |
| vulnhunter | architecture | `diagrams/vulnhunter/architecture.html` |
| vulnhunter | sequence | `diagrams/vulnhunter/sequence.html` |
| vulnhunter | dataflow | `diagrams/vulnhunter/dataflow.html` |
| Full-Chain-Sentinel-System | architecture | `diagrams/Full-Chain-Sentinel-System/architecture.html` |
| Full-Chain-Sentinel-System | sequence | `diagrams/Full-Chain-Sentinel-System/sequence.html` |
| Full-Chain-Sentinel-System | dataflow | `diagrams/Full-Chain-Sentinel-System/dataflow.html` |
| DeepAudit | architecture | `diagrams/DeepAudit/architecture.html` |
| DeepAudit | sequence | `diagrams/DeepAudit/sequence.html` |
| DeepAudit | dataflow | `diagrams/DeepAudit/dataflow.html` |

## 过程备注

1. 初稿 JSON 的 `meta.repository` 使用了 `host/owner/name`，与 schema 要求的 `url`+`revision` 不符；已去掉 `repository`（调研图不需要 `--repo-root` 溯源）。
2. showcase 比默认 validate 更严（viewport 宽高比、label clearance、短折线等）。几何按官方 example（`web-app` / `cache-miss-request` / `product-analytics`）重写后一次 finalize 全过。
3. 同目录会生成 Archify sidecar（`.delivery.json`、`.browser-check.json` 等），属正常产物。
4. **未执行 git push**（按任务要求）。
