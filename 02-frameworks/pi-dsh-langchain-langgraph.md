# 框架专章：pi · dsh · LangChain · LangGraph

> 本章服务「自建智能体调度平台」选型：比较 **Agent Harness / 编排运行时**，不讨论漏洞利用细节。  
> 参考：earendil-works/pi、deepseek-ai/deepseek-harness、LangChain/LangGraph 官方文档、arXiv:2608.23953。

## 0. 结论先行

| 框架 | 一句话定位 | 对我们调度平台的建议用法 |
|---|---|---|
| **pi** | 极简可扩展 harness：会话树、lane、compaction、扩展点 | **会话内核 / 压缩 / 工具循环**首选参考或直接嵌入 |
| **dsh** | Everything-is-a-plugin（Cordis）；会话日志为真相源 | **插件化能力缝与审批策略**参考；学习成本高 |
| **LangChain** | 模型/工具/链/Agent 高层抽象与集成面 | 作为 **LLM 与工具适配层**，勿当战役调度器 |
| **LangGraph** | 低层有状态图编排：持久化、中断、HITL、流式 | **多阶段战役状态机 / 审批门**首选编排运行时 |

论文 *Architectural Convergence in Three LLM Agent Harnesses*（2608.23953）指出 deepagents / pi / dsh 虽哲学对立，却收敛到五要素：商品化 loop、可回放会话记录、模型 quirks 数据化、上下文渐进披露、显式扩展缝；并指出共同缺口：**对外可验证性（external verifiability）**——对安全审计平台是重要产品差距。

---

## 1. pi（earendil-works/pi）

### 1.1 定位

- 口号：*minimal, extensible agent harness*；默认**不做** sub-agent / plan mode，鼓励扩展包自建  
- 包结构：`pi-ai`（多 provider）、`pi-agent-core`（AgentHarness）、`pi-coding-agent`（CLI）、`pi-durable`、`pi-tui`、`chord`  
- 四种使用面：交互 / print-JSON / RPC / TypeScript SDK  

### 1.2 调度

- **Lane + Operation 状态机**：接受 prompt、跑 model/tool、队列、compact、resume 中断作业  
- 持久化：immutable entries + mutable registers + usage；`op.meta` / `op.state` 事务写入，支持崩溃恢复  
- SessionTree：分支导航、快照订阅、hooks/events  

### 1.3 上下文压缩

（详见 Full-Chain-Sentinel 仓内同步文档 `packages/coding-agent/docs/compaction.md`）

- 阈值：`contextTokens > contextWindow - reserveTokens`（默认 reserve 16384）  
- 保留近端 `keepRecentTokens`（默认 20000）；结构化摘要（Goal/Progress/Decisions…）+ 文件读写列表累积  
- Branch summarization；overflow 时 compact-and-retry；扩展可拦截 `session_before_compact`  

### 1.4 门禁 / 扩展

- 权限模型偏「当前用户权限执行」；沙箱需自建或容器化整 CLI  
- 扩展：tools / commands / skills / prompt templates / themes；npm 或 git 安装包  
- Full-Chain-Sentinel 在其上加了审批三档与 Kali 容器 —— 说明 **领域门禁应叠在 harness 外**  

### 1.5 应用建议

嵌入为「对话与工具循环内核」；战役级 planner-worker、资产图、不可逆分级在上层自建。

---

## 2. dsh（DeepSeek Harness）

### 2.1 定位与公开信息

- 仓库：https://github.com/deepseek-ai/deepseek-harness（**公开可用**，非「信息有限」）  
- 口号：**Everything is a Plugin**；底层 Cordis（时空可组合插件范式）  
- Profile：`web` / `headless` / `sdk` / `sdk-minimal` / `acp`；bundle + `cordis.patch.yml` 分层覆盖  

### 2.2 核心包与 ctx 键

| 包 | 职责 | ctx |
|---|---|---|
| core/session | 追加写 SessionEvent 日志 | ctx.sessions |
| core/system-prompt | 提示与工具 schema 装配 | ctx.systemPrompt |
| core/tools | 作用域工具注册与守卫执行管道 | ctx.tools |
| core/agent | Agent 接口与事件 | ctx.agents |
| core/agent-loop | 默认驱动 | ctx.agentLoop |
| llm/llm | 流式与适配缝 | ctx.llm |

**模型可见 ⟺ 已入日志**：`deriveMessages()` 从会话日志投影历史；迁移与世代文件规则严格。

### 2.3 调度（Turn / Step）

文档化 turn 流：

```
turn/start → claim input → assemble prompt → agent/pre-step
  → step/start → agent/request → stream → tool/call* (pre/execute/post)
  → step/end → … → agent/turn-stopping → turn/end
```

瀑布事件必须 `next()`；可在 pre-step 拒收或改写输入。

### 2.4 上下文与门禁

- 渐进披露与投影缝（session-projection）；working directory 与沙箱写根分离  
- `dsh-base` 含 **sandbox and approval policy** 作为一等插件层  
- Capability seams：fs / subprocess / tools 可整体指向远程沙箱  
- Experimental Agent Teams：花名册与任务状态 + inbox  

### 2.5 应用建议

若团队接受 TypeScript/Cordis 学习曲线，dsh 适合做**可热插拔安全策略与工具注册中心**；与 LangGraph 可分工（dsh=执行面，LangGraph=战役图）。公开文档完善，缺口在「渗透领域预制技能」而非架构不可见。

---

## 3. LangChain

### 3.1 定位

- **Agent 框架**：模型、工具、检索、记忆、预构建 Agent 抽象与海量集成  
- 适合快速拼装 LLM 应用；**不是**长程有状态编排的最底层  

### 3.2 与调度平台关系

- 用作 Provider 适配、工具包装、RAG、评估胶水  
- 预构建 Agent 隐藏状态机细节，不利于「审批门 / 阶段可审计」——安全调度应下沉到 LangGraph 或自研引擎  
- HITL：新版可通过 middleware（如 HumanInTheLoopMiddleware）在工具调用前 interrupt，但仍依赖 LangGraph checkpointer  

### 3.3 应用建议

**集成层默认选 LangChain（或 LiteLLM）**；不要用 RunnableSequence 硬撑多阶段渗透战役。

---

## 4. LangGraph

### 4.1 定位

- **编排运行时**：耐久执行、流式、HITL、持久化；可混用确定性节点与 LLM 节点  
- AuroraRecon、众多生产 Agent 的战役骨架  

### 4.2 关键能力（官方）

| 能力 | 机制 |
|---|---|
| Persistence | Checkpointer（线程内短记忆）+ Store（跨线程长记忆） |
| HITL | `interrupt()` / `interrupt_before`；`Command(resume=…)` 恢复；需持久 checkpointer |
| 生产存储 | PostgresSaver / AsyncPostgresSaver；内存仅开发 |
| 多 Agent | 子图、supervisor、swarm 等模式可组装 |
| 可观测 | LangSmith tracing |

### 4.3 在安全调度中的模式

- **线性 Kill-Chain DAG** + 关键节点前 interrupt（Aurora 默认）  
- **Feedback** 回流边做失败再规划  
- **Supervisor** 星形路由做动态阶段选择  
- 状态对象承载 findings / scope / autonomy_level / approved 标志  

### 4.4 应用建议

**战役编排首选 LangGraph**；与 pi/dsh 组合时：Graph 管阶段与审批，Harness 管子会话工具循环与压缩。

---

## 5. 对比总表（框架层）

| 维度 | pi | dsh | LangChain | LangGraph |
|---|---|---|---|---|
| 抽象层级 | Harness | Harness（插件 OS） | 组件框架 | 图编排运行时 |
| 多 Agent | 默认弱，扩展强 | Agent Teams（实验） | 多种预构建 | 一等（子图/监督） |
| 状态持久 | SessionTree + durable | SessionEvent 日志 | 弱/外置 | Checkpointer+Store |
| 上下文压缩 | 内建工业级 | 日志投影+渐进披露 | 需自建/中间件 | 需节点内或外置 |
| HITL | 扩展/产品层 | approval policy 插件 | middleware→Graph | 原生 interrupt |
| 学习/治理成本 | 中 | 高 | 低 | 中 |
| 渗透平台贴合 | 会话壳优秀 | 能力缝优秀 | 适配层 | **战役图优秀** |

## 6. 推荐组合（产品架构草案）

```
┌─────────────────────────────────────────────┐
│ 控制面：任务/资产/授权/配额（自研）            │
├─────────────────────────────────────────────┤
│ 战役编排：LangGraph（阶段·回流·审批 interrupt）│
├─────────────────────────────────────────────┤
│ 会话执行：pi 或 dsh（tool loop·compact·审计日志）│
├─────────────────────────────────────────────┤
│ 模型/工具适配：LangChain 或 LiteLLM + MCP     │
└─────────────────────────────────────────────┘
```

质量/审计流水线可借鉴 **Clouditera VulnHunter**（YoungFlow decide 螺旋 + gate/双向论证）；若直接采用 YAML flow 资产包，编排层可评估 **YoungFlow**（见 `youngflow.md`）作为 LangGraph+pi 胶水，与「纯 LangGraph 自建」二选一。记忆子系统借鉴 ARTEX 双图或 Aurora 世界模型。


## 7. YoungFlow 索引

云起无垠 flow runner（口语 yongflow）：专章见 [youngflow.md](./youngflow.md)。VulnHunter 子模块依赖它执行 `flows/vulnforge/flow.audit.yaml`。
