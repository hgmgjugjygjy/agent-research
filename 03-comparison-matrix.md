# 竞品对比矩阵（调度 / 上下文 / 门禁 / 技术栈）

> 调研日：2026-10-16。评分：● 强 / ◐ 中 / ○ 弱或缺失。仅架构视角。

## 1. 总览

| 项目 | Stars≈ | 语言 | License | 主场景 | Agent 范式 | 编排底座 |
|---|---:|---|---|---|---|---|
| ARTEX | 229 | Go | AGPL-3.0 | 自主渗透平台 | Planner–Worker + 双图 | 自研 engine + norma |
| Strix | 68.8k | Python | Apache-2.0 | AI 渗透 CLI/Cloud | 多 Agent 树 | AgentCoordinator |
| AuroraRecon | 3 | Python | MIT* | Kill-Chain 渗透 | 阶段图 + 监督器 | **LangGraph** |
| VulnHunter | 78 | TypeScript | Apache-2.0+* | AI 挖掘工作台 | decide 螺旋 | **YoungFlow**（LangGraph+pi） |
| Full-Chain-Sentinel | 79 | TypeScript | MIT | 授权渗透控制台 | 单会话+扩展 | **pi fork** |
| DeepAudit | 7.2k | Python | AGPL-3.0 | AI 代码审计 | Orchestrator 树 | 自研 ReAct + SSE |

\* Aurora README 声明 MIT，GitHub license 元数据为空，合入前需再核。

\*\* VulnHunter=**Clouditera/VulnHunter**（纠偏；非 capitalone）。License open-core，package.json 亦见 AGPL，需法务再核。引擎=YoungFlow。

## 2. 调度管理

| 能力 | ARTEX | Strix | Aurora | VulnHunter | Sentinel | DeepAudit |
|---|---|---|---|---|---|---|
| 任务创建与生命周期 | ● | ● | ● | ◐（仓级 job） | ●（会话/项目） | ● |
| 并行 Worker/子 Agent | ● | ● | ◐（阶段串行主） | ●（阶段内并行） | ◐ | ● |
| 状态机/图编排 | ●（意图 frontier） | ●（status 集） | ●（三类图） | ●（YoungFlow routes） | ○（会话） | ◐ |
| 失败重试 | ● | ● | ●（retry_node） | ●（YF resume/continue） | ●（pi resume） | ●（熔断重试） |
| 预算/配额 | ◐ | ● | ●（速率） | ○ | ○ | ●（限流） |
| 断点续跑 | ◐ | ●（snapshot） | ●（Postgres CP） | ◐ | ●（JSONL） | ●（checkpoint 表） |

## 3. 上下文压缩 / 记忆

| 能力 | ARTEX | Strix | Aurora | VulnHunter | Sentinel | DeepAudit |
|---|---|---|---|---|---|---|
| Transcript 摘要压缩 | ● | ●（安全模板） | ○ | ●（pi compaction） | ●（pi） | ●（MemoryCompressor） |
| 图/外置记忆 | ●（双图+digest） | ◐（report state） | ●（攻击图+prior） | ●（wiki/todo/leads） | ○ | ◐（handoff） |
| 跨任务记忆 | ◐ | ◐ | ●（engagement_memory） | ○ | ○ | ○ |
| Tool 配对安全切分 | ●（norma） | ● | — | — | ● | ◐ |
| Worker 过程共享 | ●（traces 检索） | ●（inbox 消息） | ◐ | ○ | ○ | ●（message_bus） |

## 4. 门限审核

| 能力 | ARTEX | Strix | Aurora | VulnHunter | Sentinel | DeepAudit |
|---|---|---|---|---|---|---|
| 人在环路审批 | ●（intercept ask） | ◐ | ●（interrupt） | ◐（finding 审核流） | ●（三档） | ◐（用户消息） |
| Scope 强制 | ◐（任务 scope） | ●（prompt+授权） | ●（runtime 解析命令） | ◐（audit_scope 指令） | ●（授权承诺） | ●（文件范围） |
| 不可逆分级 | ○ | ○ | ● | ○ | ○ | ○ |
| 工具权限/沙箱 | ●（规则+代理） | ●（Docker） | ●（toolbox 容器） | ●（SandboxPlane+tools 白名单） | ●（Kali 容器，边界弱） | ◐ |
| 质量证伪门 | ◐（归并人工） | ◐（coverage gate） | ◐ | ●（decide+gate+双向论证） | ○ | ●（VerificationAgent） |
| 审批 fail-closed | ● | — | ● | — | ○（AI 审批 fail-open） | — |

## 5. 技术栈速查

| 项目 | LLM 接入 | 前端 | 存储 | 特色子系统 |
|---|---|---|---|---|
| ARTEX | Anthropic/OpenAI | Next.js 内嵌 | Postgres | MITM 流量、ScopeSentry 同步 |
| Strix | LiteLLM 多厂 | TUI+Viewer | 本地 run 目录 | MCP、Cloud、CI |
| Aurora | OpenAI 兼容 | Vue3 | PG+Redis+MinIO | MSF、Skill YAML、世界模型 |
| VulnHunter | 多 provider / models.json | React 工作台 | PG+MinIO | VulnForge decide 螺旋 |
| Sentinel | OpenAI 兼容 | Web（pi） | JSONL+本地配置 | 安卓桥、POC 订阅 |
| DeepAudit | LiteLLM/Ollama | React | PG/Supabase | Semgrep/Bandit 工具链 |

## 6. 选型映射（对我们目标）

| 我们需要的能力 | 首选借鉴 |
|---|---|
| 战役阶段编排 + HITL | AuroraRecon（LangGraph）+ 官方 LangGraph |
| 会话压缩与审计日志 | pi / Full-Chain-Sentinel / Strix compaction |
| 资产与探索长期记忆 | ARTEX 双图 + 冷 digest |
| 分层安全门 | Aurora（scope+YAML+irreversible+autonomy） |
| 发现质量 | Clouditera VulnHunter decide/gate/双向论证 + sandbox_cfg |
| 插件化工具/策略 | dsh seams |
| 审计类产品壳 | DeepAudit 事件与 Orchestrator 树 |
