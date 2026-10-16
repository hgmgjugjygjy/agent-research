# YoungFlow（Clouditera/Youngflow）框架专章

> 仓库：https://github.com/Clouditera/Youngflow · TypeScript · v0.8.4 · Stars≈0（公开）  
> 用户口语 **「yongflow」** = 本仓。  
> 一句话：**通用 AI flow runner = LangGraph 编排多阶段流水线 + pi CLI 执行每个 stage；flow 是独立资产包，引擎不绑定业务。**

## 1. 与 VulnHunter 的关系

| 角色 | 仓库 | 职责 |
|---|---|---|
| **引擎** | Clouditera/Youngflow | 解析 `flow.yaml` / `flow.audit.yaml`，建 LangGraph DAG，调 pi，管 checkpoint / report / 路由 |
| **资产包** | Clouditera/VulnHunter `flows/vulnforge/` | 安全审计业务：decide 螺旋、skills、schemas、templates、extensions |
| **集成** | VulnHunter `.gitmodules` → `submodules/youngflow` | 平台 Worker 内跑 `youngflow …`；另挂 `pi-web-access` 扩展 |

结论：**YoungFlow 即 VulnHunter 审计流水线的 flow 引擎运行时**；VulnForge 不是引擎本身。

## 2. 核心模块（`src/`）

| 模块 | 职责 |
|---|---|
| `cli.ts` | 从 flow inputs 动态生成 CLI；`--resume` / `--continue` / `--until` / `--recursion-limit` |
| `spec.ts` + `flow.schema.yaml` | YAML→只读类型 + Ajv/语义校验 |
| `orchestrator.ts` | **LangGraph** DAG：single / parallel / map / **join**；routes + max_loops |
| `checkpoint.ts` / `state.ts` / `condition.ts` | 断点；从产物抽路由状态；条件表达式 |
| `executor.ts` / `runner.ts` | stage→RunConfig→**pi** 进程；NDJSON 事件流与重试 |
| `model-config.ts` | `.env` + 可选 `models.json`；启动 `pi --list-models` 硬预检 |
| `workspace.ts` / `prompt.ts` / `report.ts` | 输出目录；`${}` 插值；`flow-report.html` |
| `session-markdown.ts` | 会话可读导出 |

依赖：`@langchain/langgraph`、ajv、commander、fast-glob、js-yaml。前提：系统已装 **pi** CLI（Node≥20）。

## 3. 调度要点

- Stage 类型：`single` | `parallel` | `map` | `join`（join 不调 agent，只收口）  
- **Routes**：`when: stage.key op value`；多条件可并发扇出；无 when 为 fallback；回路强制 `max_loops`  
- **State 提取**：file+field（硬契约，缺文件即失败）/ keys_of / glob（0 匹配合法）  
- **Map**：glob 或 `{yaml,path}` 字符串列表；`filter` 按 frontmatter 过滤  
- **工具白名单**：stage `tools` **replace** defaults；`exclude_tools` denylist；扩展须 extensions+tools 双登记  
- **Recursion**：`recursion_limit`（LangGraph 步进预算）≠ `max_loops`（业务回路阀）  
- **Continue vs Resume**：continue 归档 `.youngflow/` 引擎态、保留业务产物、新 session；resume 跳过已完成 stage  

## 4. 上下文 / 压缩

- 引擎态进 `output_dir/.youngflow/`（checkpoints/logs/sessions），业务产物干净分目录  
- `defaults.compaction` → 写入 `.pi-agent/settings.json`（pi 原生压缩）  
- `session.reuse` + `session.compact_at`：同 run 内复用会话并按窗口比例主动 compact  
- 提示词四层：Agent / Skill / Task / Prompt（运行时变量只在 Prompt）  

## 5. 门禁相关（引擎能力，业务门在资产包）

- Schema 校验 fail-fast；模型预检 fail-fast  
- 工具 allowlist/denylist 按 stage  
- `error_strategy`: stop | continue  
- Extension 可做 output-contract（产物存在性/质量，失败可 follow-up）  
- **不内置**安全沙箱：沙箱由资产包 extension + 宿主平台（如 VulnHunter SandboxPlane）提供  

## 6. 与 LangGraph / pi / dsh 对比（选型）

| | YoungFlow | 纯 LangGraph | pi | dsh |
|---|---|---|---|---|
| 定位 | Flow 资产包 runner | 图编排库 | 会话 harness | 插件化 harness OS |
| 编排 | 内置 LangGraph | 自建节点 | 无战役图 | 无战役图 |
| 执行 | 调 pi | 自接 model/tool | 本职 | 本职 |
| 资产形态 | 目录化 flow.yaml | 代码定义图 | skills/extensions | plugins |
| 适合 | 安全/审计类可版本化流水线 | 强定制战役状态机 | 单会话深度 tool loop | 强策略/沙箱插件 |

**产品启示**：若采用「YAML 可运营流水线」，YoungFlow 类引擎可省自研 DAG+pi 胶水；若要深度 HITL interrupt / 不可逆分级，仍需在控制面或 Aurora 式 Graph 节点补强——YoungFlow 路由偏产物文件信号，而非一等公民 interrupt API。

## 一句话结论

**YoungFlow 是云起无垠的 flow 运行时（LangGraph+pi）；Clouditera VulnHunter 的 VulnForge 是跑在其上的安全审计资产包——二者是引擎与业务的关系，不是同义重复。**
