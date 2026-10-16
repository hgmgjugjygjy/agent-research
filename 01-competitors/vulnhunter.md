# VulnHunter（Clouditera/VulnHunter · 云起无垠）竞品研究

> **纠偏说明（2026-10-16）**：此前调研误标 [capitalone/VulnHunter](https://github.com/capitalone/VulnHunter)（Claude Code 三技能 Hunt→Fix→Verify）。**现以云起无垠 [Clouditera/VulnHunter](https://github.com/Clouditera/VulnHunter) 为准。** capitalone 仅为易混淆同名开源项目，脚注保留，不作本调研主锚点。  
> 仓库：https://github.com/Clouditera/VulnHunter · TypeScript monorepo · Stars ≈78 · 版本 ≈2.3.x · License：Apache-2.0 + 附加条款（open-core；`package.json` 亦标 AGPL-3.0-only，合入前需法务再核）  
> 编排运行时：**YoungFlow**（子模块 `submodules/youngflow` → [Clouditera/Youngflow](https://github.com/Clouditera/Youngflow)）  
> 核心审计资产包：`flows/vulnforge/`（`flow.audit.yaml` decide 螺旋）

## A. 定位与技术栈

| 项 | 内容 |
|---|---|
| 定位 | **自托管 AI 漏洞挖掘工作台**：目标画像 → 认知螺旋审计 → 双向论证 → 可选沙箱 POC/EXP → 审核与报告；白盒/源码侧为主，带平台壳 |
| 形态 | pnpm/turbo monorepo：`packages/{service,web,shared,worker-bridge,enterprise}` + `flows/{vulnforge,vulnhunter-report}` + Docker Workers |
| 编排 | **YoungFlow** 执行 `flows/vulnforge/flow.audit.yaml`；CLI `youngflow flow.audit.yaml --work-dir … --output-dir …` |
| 会话执行 | YoungFlow 底层调 **pi CLI**（`@earendil-works/pi-*`）；扩展经 pi extension（coverage / output-contract / sandbox-plane / workspace-diff） |
| 服务面 | Hono + WebSocket + MCP；Postgres + MinIO；React/Vite Web（端口默认 23000） |
| 模型 | `models.json.example` + 多凭证；OpenAI 兼容等多 provider（非绑定单一 Claude Code） |
| 沙箱 | `deploy/sandbox/` SandboxPlane + flow input `sandbox_cfg`；profiles：`base-linux` / `linux-docker`(sysbox) / `linux-qemu-system` |

## B. Agent 拓扑（VulnForge decide 螺旋）

**不是** capitalone 的 Hunt→Fix→Verify 三 skill；而是 **decide 驱动的认知螺旋**（wiki + coverage 锚定）：

| Stage id | 职责 |
|---|---|
| **decide** | 唯一调度器：读工作区 / coverage / workspace_diff，写 `decision.yaml.next`，维护 todo/leads/done；**不亲自**建认知/狩猎/论证/复现 |
| onboard | 画像 + wiki + 威胁模型；写 `gate.yaml`（continue\|end）；动态任务经 sandbox-plane 申请沙箱 |
| cognize | 切面认知（COG）→ 产出 ADV 审计链路 |
| research | 公开情报检索（可选 pi-web-access）→ ADV/线索 |
| hunt | 沿 ADV 精读 → 产出 HYP 假设 |
| verify | **双向论证**（affirmer + challenger）→ confirmed finding 或 refuted |
| poc-verify / ev-assess / exp-build | 可选动态复现 / 单洞 EXP / 组合链（需 enable_* + sandbox） |
| cycle_join → decide | 回环（max_loops 业务阀） |
| report / complete / exit | 汇总或异常退出 |

提示词四层（YoungFlow 契约）：**Agent**（`agents/agent.md`） / **Skill**（方法论） / **Task**（`tasks/*.md`） / **Prompt**（flow YAML 运行时变量）。

## C. 调度管理

- **引擎**：YoungFlow = LangGraph DAG + pi 执行器；stage 类型 single / parallel / map / join  
- **路由契约**：`decision.schema.yaml` 的 `next` ∈ {onboard,cognize,hunt,verify,poc-verify,ev-assess,exp-build,report,exit}；`gate.schema.yaml` 的 `next` ∈ {continue,end}  
- **平台侧**：`packages/service` workers（`scheduler.ts` / `scan-worker.ts` / reconciler）起 Docker Worker，挂载 flow 资产与产物；任务生命周期（创建/暂停/取消/恢复/重跑）  
- **并行**：flow `defaults.max_parallel`；map stage `concurrency`；decide `max_parallel: 4`  
- **续跑**：YoungFlow `--resume` / `--continue`；平台任务控制面再包一层  
- **报告 flow**：另有 `flows/vulnhunter-report/flow.yaml` 独立报告流水线  

## D. 上下文压缩

- **阶段隔离 + 工作区外置记忆**：wiki / COG / ADV / HYP / findings / knowledge 落盘；decide 不持全量源码分析正文  
- **coverage extension**：hunt 阅读覆盖率外置，decide/report 用 viewer 查询  
- **workspace-diff / snapshot**：decide 只看增量，降低上下文污染  
- **YoungFlow/pi**：`defaults.compaction` + `session.reuse` / `session.compact_at`（引擎层）；会话进 `.youngflow/sessions/`  
- 无 capitalone 式「编排器故意不读 findings」同名机制，但 **decide 角色边界**达到类似效果  

## E. 门限审核

| 门 | 落点 |
|---|---|
| **onboard gate** | `gate.yaml`：continue→cycle_join / end→exit；reason 枚举含 `no_compatible_sandbox` 等；平台 `parseGateYaml` fail-closed |
| **decide 路由门** | 只派发、不执行；`sched_instr` 可截断阶段；动态任务要求 `knowledge/build/build.md` sandbox=available |
| **双向论证质量门** | verify：affirmer + challenger；hyp_status confirmed\|refuted；未确认不成 finding |
| **output-contract** | 产物写权限 + schema 校验（extension） |
| **工具白名单** | stage `tools` **整表替换** defaults；扩展工具须同时进 tools + extensions |
| **沙箱** | `flow_inputs.sandbox_cfg`；sandbox-plane 只暴露 profile 能力标志，不回传地址/凭证；POC/EXP 可走隔离 profile |
| **产品审核** | Findings 四态审核工作流（平台层，非 flow stage） |

## F. 可借鉴 / 不适合

**可借鉴**

1. **YAML flow + decide 螺旋**：调度与执行分权，路由信号文件化（decision/gate）  
2. **Skill 外置方法论**，flow 只保留高层认知动作  
3. **gate fail-closed + sandbox plane 代理**（凭证不下沉到 agent）  
4. **todo/leads/done + wiki** 作为跨 session 记忆协议  
5. 平台壳（任务/Chat MCP/报告）与审计引擎解耦  

**不适合 / 风险**

1. 强依赖 YoungFlow + pi 栈；自建平台若已选纯 LangGraph，需适配或替换引擎层  
2. Open-core / 许可证表述需法务核（Apache 附加条款 vs package.json AGPL）  
3. decide 螺旋靠 LLM 调度，需 `max_loops` / recursion_limit / sched_instr 防漂  
4. 动态 POC/EXP 与沙箱运维成本高；未配置 sandbox 时本地执行风险需产品策略兜住  

## 一句话结论

**Clouditera VulnHunter 是「YoungFlow decide 螺旋 + 平台壳 + 沙箱平面」的自托管漏洞挖掘工作台；质量门在 decide/gate/双向论证/sandbox_cfg，而非 capitalone 式 Claude Code 三技能。**

## 脚注：同名项目 capitalone/VulnHunter

Apache-2.0、Python、Claude Code skills（`/vulnhunt` `/vulnhunter-fix` `/vulnhunt-fix-verify`）+ `vulnhunter-agent`。与本仓目录结构、编排引擎、产品形态均不同；用户截图中的 `flows/vulnforge`、`id: decide`、`sandbox_cfg` **仅属于 Clouditera 版**。
