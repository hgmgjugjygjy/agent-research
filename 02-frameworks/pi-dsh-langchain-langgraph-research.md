# Pi、DeepSeek Harness、LangChain 与 LangGraph：Agent 运行机制与技术选型对比

> 研究范围：Pi（`earendil-works/pi`）、DeepSeek Harness（`deepseek-ai/deepseek-harness`）、LangChain v1 Python Agent、LangGraph Python。重点是架构机制与工程取舍，不将框架能力推断为统一性能基准结果。

## 1. 比较对象与架构层级

四者并非完全处于同一抽象层级。Pi 与 DeepSeek Harness 更接近**具体 Agent 执行内核及其宿主环境**；LangChain v1 的 `create_agent` 是**高层 Agent 构建接口**；LangGraph 是**有状态的图执行与编排运行时**。LangChain v1 Agent 本身构建在 LangGraph 运行时之上，因此比较时应区分「开发 API」与「底层执行语义」。

| 维度 | Pi | DeepSeek Harness（dsh） | LangChain v1 Agent | LangGraph |
|---|---|---|---|---|
| 核心定位 | 可扩展编码 Agent / Agent Loop | Cordis 插件化 Agent Harness | 快速构建模型—工具 Agent | 有状态图编排与执行 |
| 主抽象 | `Agent`、`agentLoop`、Coding Agent、Session | Cordis `ctx` 服务、`Agent`、Turn/Step、SessionEvent | `create_agent`、Tool、Middleware | `StateGraph`、Node、Edge、Reducer、Checkpoint |
| 主运行粒度 | 单 Agent 内的模型与工具循环 | 单 Agent 的 Turn/Step 生命周期 | 预置 Agent 图中的模型与工具循环 | 图节点与 superstep |
| 主要扩展位置 | 工具、Extension、会话/上下文钩子 | 插件服务、事件、工具执行流水线 | Middleware、Tool、模型适配 | 自定义节点、边、子图、状态 |
| 典型实现语言 | TypeScript | TypeScript | Python（本报告） | Python（本报告） |
| 复杂多阶段流程 | 通常由上层组织 | 通常由上层组织 | 可在高层 Agent 外组合 | 原生显式建模 |

**判断标准**：不要只比较 API 数量，而应追问「谁决定下一步」「运行状态保存在哪里」「失败时从何处恢复」「工具调用在何处受控」。

## 2. Agent Loop：真实执行语义与调度差异

### 2.1 最小共同流程

典型工具型 Agent 都要完成：接收输入 → 准备消息与工具 Schema → 调用模型 → 解析工具调用 → 执行工具 → 追加结果 → 再次请求模型 → 结束。差异不在这条抽象流程，而在**循环控制权、拦截点、消息队列和失败恢复**。

### 2.2 Pi：Agent Core 负责连续模型—工具循环

Pi 的 `packages/agent/src/agent-loop.ts` 导出 `agentLoop(...)`，接受 prompts、AgentContext、配置、AbortSignal 与 StreamFn，返回 AgentEvent 的事件流。Agent 内部保留 `AgentMessage`，在模型调用边界转换为模型消息。工具声明转换与参数校验由执行链负责；工具结果进入后续模型上下文。`Agent` 封装调用入口与状态，Coding Agent 再补上交互式会话、扩展和压缩等产品能力。

**调度特征**：适合一个 Agent 在持续会话内反复「模型决策—工具执行」。Steering/follow-up 等输入管理是 Agent Loop 的重要工程特征，但它不自动等于一个可视化、多节点、持久化的业务工作流引擎。

**设计收益**：核心调用路径相对直接，适合深入控制工具循环与流式事件。**代价**：复杂阶段依赖、跨任务审批和分布式工作流仍需要上层设计。

源码：[agent-loop.ts](https://github.com/earendil-works/pi/blob/main/packages/agent/src/agent-loop.ts)。

### 2.3 dsh：Turn/Step + Cordis 事件驱动的执行生命周期

DeepSeek Harness 的 `core/agent-loop` 提供默认执行驱动；`core/agent` 定义 Agent 接口及实时注册表。官方将 **Step** 定义为一次模型请求及其工具调用，将 **Turn** 定义为零个或多个 Step：Turn 在认领输入前开启，直到没有待处理工作才关闭。执行中通过 `agent/*` 事件提供初始化、请求、校验、状态及继续执行等拦截点。

一条典型路径是：创建 Agent → 串行执行 `agent/created` 初始化 → 从 inbox 认领输入 → 组装 Prompt/Tool Schema/上下文 → `agent/pre-step` 等检查 → 模型请求 → 处理响应与工具 → 写入 SessionEvent → 判断继续或关闭 Turn。这里是官方架构文档描述的**生命周期路径**，不是某个单一函数的逐行调用栈。

**设计收益**：插件可在服务、事件和工具流水线处替换或拦截能力，适合需要多处注入治理逻辑的宿主。**代价**：执行行为分散在插件与事件处理器中，定位问题时必须同时追踪插件加载顺序、事件消费者及状态投影。

参考：[architecture.md](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md)、[core subsystem](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/core.md)。

### 2.4 LangChain：`create_agent` 提供高层默认 Agent 循环

LangChain v1 通过 `create_agent` 把模型、工具、系统提示和 Middleware 组织为 Agent；其运行基于 LangGraph。开发者通常无需自行写「模型节点 → 工具节点 → 再回模型」的全部状态转移。Middleware 可围绕模型请求、工具调用或 Agent 生命周期插入逻辑，适合快速加入动态提示、上下文裁剪、审批等横切能力。

**设计收益**：较少的样板代码、Python 生态与标准化 Tool 接口。**代价**：当业务需要严格控制每一步执行路径、重试位置或跨阶段状态时，需要理解底层图运行语义，甚至直接改用 LangGraph 构建自定义图。

参考：[LangChain Agents](https://docs.langchain.com/oss/python/langchain/agents)、[Middleware](https://docs.langchain.com/oss/python/langchain/middleware)。

### 2.5 LangGraph：图调度而非固定的 Agent Loop

LangGraph 由 State、Node 和 Edge 组成；条件边决定下一节点，Reducer 决定并发更新如何合并。运行时采用 Pregel 风格的 superstep 执行模型。模型—工具循环只是可以用图表达的一种结构；也可以表达并行调查、人工审批、结果聚合、循环重试和阶段性结束。

**设计收益**：控制流显式、可测试，适合多阶段、分支、并行和可恢复的业务流程。**代价**：状态 Schema、节点幂等性、Reducer 与 checkpoint 边界必须明确设计；简单单 Agent 任务直接写图可能增加工程复杂度。

参考：[Graph API](https://docs.langchain.com/oss/python/langgraph/graph-api)、[Pregel](https://docs.langchain.com/oss/python/langgraph/pregel)。

### 2.6 调度机制横向结论

| 关键问题 | Pi | dsh | LangChain Agent | LangGraph |
|---|---|---|---|---|
| 下一步由谁决定 | Agent Loop 基于模型响应与待处理输入 | 默认 Loop + 事件/服务扩展 | 预置 Agent 图与 Middleware | 图边、状态与调度器 |
| 最适合观察的单位 | AgentEvent、工具调用 | Turn、Step、SessionEvent、`agent/*` | Agent state、Middleware 调用 | Node、state update、superstep |
| 运行中干预 | Agent 输入队列/事件与扩展 | Inbox 与事件拦截 | Middleware、HITL | interrupt、条件路由、状态更新 |
| 多分支并行任务 | 需要上层编排 | 需要上层编排 | 需进一步组合图 | 图执行的核心能力 |
| 主要调试困难 | 长会话与工具事件追踪 | 插件组合、事件顺序、状态投影 | 高层封装与底层图边界 | 状态合并、重放与幂等性 |

## 3. 上下文管理：构建、压缩与 Token 预算

首先区分三个概念：**模型上下文**是本次发送给 LLM 的消息；**会话历史**是系统保存的事件或消息记录；**长期记忆**是跨会话可检索的知识。保存历史不代表每次都全部送入模型，Checkpoint 也不自动等于压缩。

### 3.1 Pi：Coding Agent 的 Compaction

Pi Coding Agent 的 Compaction 文档给出自动触发条件：

`contextTokens > contextWindow - reserveTokens`

默认 `reserveTokens = 16384`，`keepRecentTokens = 20000`。其核心做法是对较旧内容生成结构化摘要，同时保留近期消息；文件读取/修改信息可以跨摘要累积。Compaction 发生在 Coding Agent 的会话管理层，而不是说底层 `agentLoop` 本身就包含完整压缩系统。还存在 Branch Summarization，用于在分支切换时保留被离开分支的关键信息；Context Overflow 可触发压缩与重试路径。

**取舍**：适合长时间编码/工具任务；风险是摘要遗漏关键约束、精确参数或工具输出，因此重要事实应以结构化任务状态或文件保存，而不是仅依赖摘要。

参考：[Compaction 设计](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/compaction.md)、[实现](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/src/core/compaction/compaction.ts)。

### 3.2 dsh：事件日志与模型上下文投影分离

dsh 将 Session 表示为追加式 `SessionEvent` 日志，模型历史通过 `deriveMessages()` 等投影机制生成，而不是另存一份独立且始终同步的聊天数组。系统提示、工具 Schema 和运行时上下文由相关服务组装。官方架构还提供 Compaction 相关扩展与持久化路径，但**事件投影本身不是压缩**：压缩策略、触发条件、摘要格式与恢复规则必须单独核对其 Compaction 子系统。

**取舍**：原始事件可用于审计和重新投影，利于工具调用生命周期追踪；代价是「事件真实记录」与「模型当前看到的消息」可能不同，调试时必须同时检查两者。

参考：[core subsystem](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/core.md)、[architecture](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md)。

### 3.3 LangChain：Middleware 在模型请求前管理上下文

`create_agent` 维护 messages 等 Agent 状态，Middleware 可动态改写请求上下文，并按需求配置消息裁剪或摘要策略。**LangChain 有管理上下文的扩展能力，不等于所有 Agent 默认自动按某一固定阈值压缩**。应区分「对即将发送的消息裁剪」与「生成持久摘要并改变会话状态」两种操作。

参考：[Short-term memory](https://docs.langchain.com/oss/python/langchain/short-term-memory)、[Middleware](https://docs.langchain.com/oss/python/langchain/middleware)。

### 3.4 LangGraph：State 是运行数据，Prompt 由节点构建

LangGraph 的 State 可包含完整消息、摘要、任务清单或外部引用，但节点如何构造模型输入由应用决定。Checkpointer 保存图状态快照，**不会自动理解哪些消息需要总结或删除**。可专设摘要节点、在模型节点前截断历史，或从 Store/RAG 检索相关事实。

参考：[Memory](https://docs.langchain.com/oss/python/langgraph/add-memory)、[Persistence](https://docs.langchain.com/oss/python/langgraph/persistence)。

### 3.5 对比结论

| 问题 | Pi | dsh | LangChain | LangGraph |
|---|---|---|---|---|
| 上下文来源 | Session 投影及 Agent 上下文 | SessionEvent 投影、Prompt 服务 | Agent messages + Middleware | 节点读取 State 后自行构建 |
| 压缩位置 | Coding Agent 内置 Compaction | 独立压缩能力/插件路径，细节按配置核验 | Middleware/应用策略 | 自定义节点/应用策略 |
| 主要优势 | 有明确的长会话压缩流程 | 原始事件与模型视图解耦 | 易在请求层加策略 | 能与业务状态明确分离 |
| 主要风险 | 摘要信息损失 | 投影与压缩路径复杂 | 策略依赖配置 | 默认不替应用解决 Token 预算 |

## 4. 状态持久化、中断与恢复

需要分开考察四种恢复：①恢复聊天历史；②恢复尚未完成的工具循环；③恢复业务工作流位置；④恢复外部工具产生的副作用。任何一个框架支持前三项，都不意味着自动回滚或去重第四项。

| 能力 | Pi | dsh | LangChain | LangGraph |
|---|---|---|---|---|
| 会话历史 | Coding Agent Session | SessionEvent 追加日志 | Agent state/messages | State 中的 messages |
| 持久化方式 | 会话管理实现；另有 Durable 相关组件 | 可配置 SessionPersistence/JSONL 等 | 可传入 Checkpointer | Checkpointer、thread_id |
| 执行中断 | Abort 与宿主控制 | Agent 生命周期、取消/拦截 | HITL 与底层图机制 | `interrupt()` + `Command(resume=...)` |
| 业务阶段恢复 | 需上层管理 | 需上层管理 | 可依赖底层图 | 主要能力之一 |
| 外部副作用幂等 | 应用负责 | 应用负责 | 应用负责 | 应用负责，重放尤其要注意 |

**LangGraph 的关键边界**：使用持久化与 interrupt 时，恢复可能重新执行中断点所在节点的部分代码；不可重复执行的网络请求、写入、审批动作应放在具备幂等键或明确任务边界的执行单元中。Checkpoint 是状态恢复工具，不是数据库事务回滚。

**Pi/dsh 的关键边界**：会话恢复和 Agent 运行状态恢复不是同一个承诺。对于「进程重启后精确恢复到某个工具调用中途」的要求，必须通过特定版本、配置与故障注入实验验证，不能仅凭存在 Session 或 Durable 组件就认定支持。

参考：[LangGraph Durable Execution](https://docs.langchain.com/oss/python/langgraph/durable-execution)、[Interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts)、[dsh core](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/core.md)。

## 5. 工具执行、扩展机制与安全边界

### 5.1 工具如何进入 Agent

- **Pi**：Agent Tool 定义进入模型可见的工具集合，Agent Loop 负责参数校验、执行与结果回传；Coding Agent Extensions 可注册工具、命令和生命周期行为。
- **dsh**：`core/tools` 提供作用域工具注册和受控执行流水线；Cordis 插件通过 `ctx` 服务与事件挂载能力，卸载插件时注册 effect 可撤销。
- **LangChain**：Python Tool/Schema 由 `create_agent` 使用，Middleware 可处理模型调用、工具执行和审批。
- **LangGraph**：工具可封装为节点，或在预置 Agent 子图内部执行；图的条件边控制何时调用工具及之后去向。

### 5.2 「可拦截」不等于「已隔离」

| 安全问题 | 框架能提供什么 | 仍需由应用/部署层解决什么 |
|---|---|---|
| 模型试图调用高风险工具 | Tool Schema、Hook/Middleware、条件路由 | 权限策略、允许操作范围、目标授权校验 |
| 执行前人工审批 | Extension/事件、HITL、interrupt 等机制 | 审批身份、不可绕过的执行网关、审计记录 |
| 工具运行隔离 | 可以接入独立执行后端 | 容器/沙箱、网络与文件权限、凭据隔离 |
| 长任务失败重试 | Agent/图的重试与恢复机制 | 幂等键、去重、超时、外部副作用核对 |
| 工具输出污染 | 可加结果过滤/校验层 | 不可信内容标记、提示注入防护、数据出站控制 |

不能将「Prompt 告诉 Agent 不要做」视为执行权限控制，也不能将「存在审批回调」视为已经实现系统级隔离。

### 5.3 扩展方式的本质差异

Pi 的扩展主要围绕单 Agent 产品运行时；dsh 的 Cordis 使模型适配、工具、会话与 Loop 本身都可以作为插件组合；LangChain Middleware 更适合在现成 Agent 上添加横切逻辑；LangGraph 通过节点/边/子图改变**控制流本身**。因此，当需求是「在每次工具调用前加规则」时四者都能实现；当需求是「三阶段任务，每阶段独立审批，失败后从阶段二恢复」时图编排通常更直接。

## 6. 工程成本与适用场景

以下为基于架构机制的**定性判断**，不是基准测试分数。

| 需求 | 更值得优先评估的方案 | 主要理由 | 必须验证的代价 |
|---|---|---|---|
| 快速完成 Python 工具 Agent 原型 | LangChain Agent | `create_agent`、Tool、Middleware 开箱组合 | 自定义执行语义的边界 |
| 深入改造单 Agent 循环 | Pi | Agent Loop 接口直接、事件可观测 | 会话/扩展与上层编排如何衔接 |
| 运行时插件化、服务替换 | dsh | Cordis 服务、事件和可撤销注册 | 插件加载顺序、跨服务排错 |
| 多阶段任务、分支并行、审批恢复 | LangGraph | StateGraph、Checkpoint、Interrupt | 状态设计、幂等、节点复杂度 |
| Python 编排 + TypeScript Agent 内核 | LangGraph + Pi/dsh | 分离业务控制流与单任务工具循环 | RPC 协议、状态同步、取消传播、部署运维 |
| 已使用 LangChain，希望增加工作流控制 | LangGraph | 同一生态下下沉到显式图层 | 迁移现有 Agent 状态与工具封装 |

### 6.1 面向授权安全测试平台的建议架构

```text
任务入口 / 授权范围
        │
        ▼
工作流层（阶段、状态、审批、重试、归档）
        │
        ▼
Agent 执行层（模型决策、工具选择、上下文）
        │
        ▼
工具执行网关（权限校验、隔离、审计、限流）
        │
        ▼
经授权的测试环境 / 资产
```

这是一种**职责划分建议**，不是某个框架自动具备的完整安全架构。若平台以 Python 为主、需求包含多阶段任务与人工审批，可优先做 `LangGraph + LangChain Agent` 的最小验证；若需要高度定制 Agent Loop 或 Cordis 插件体系，再以 Pi/dsh 为候选内核做替换实验。跨语言组合不应在缺乏必要性的情况下先行引入。

### 6.2 推荐的验证实验

1. **同一任务的 Loop 跟踪**：记录模型请求数、工具调用顺序、失败传播与最终终止原因，比较真实执行链。
2. **长上下文实验**：输入超过窗口预算的工具历史，记录触发压缩的位置、保留信息与 Token 变化；分别测试默认配置和启用策略的配置。
3. **崩溃恢复实验**：分别在模型请求前、工具调用前、工具完成后强制终止进程，检查历史、任务状态和副作用是否重复。
4. **人工审批实验**：在高风险工具调用前中断，测试拒绝、批准、超时和恢复后行为。
5. **工程成本记录**：统计所需代码量、依赖、运行进程数、排错路径和维护边界，而非仅凭框架介绍判断轻重。

## 7. 结论

- **Pi**：适合作为可扩展的单 Agent 模型—工具执行内核；其 Coding Agent 提供具体的长会话压缩与交互能力。
- **DeepSeek Harness**：核心竞争力在 Cordis 插件服务与事件驱动的执行生命周期，适合需要多层能力替换、拦截和治理的宿主。
- **LangChain v1 Agent**：高层 Agent API 和 Middleware 降低 Python Agent 的初始开发成本，底层与 LangGraph 并非割裂。
- **LangGraph**：强项是显式多阶段控制流、状态持久化、分支和人工中断；不应被当作自动解决上下文压缩或工具隔离的组件。

**选型原则**：先判断主要难题是「Agent 内部如何循环」还是「多个任务阶段如何协作与恢复」。前者重点比较 Pi/dsh/LangChain；后者重点评估 LangGraph。最终结论应由同一组场景、配置和故障注入实验验证。

## 参考资料

1. [Pi 仓库](https://github.com/earendil-works/pi)；[Agent Loop 源码](https://github.com/earendil-works/pi/blob/main/packages/agent/src/agent-loop.ts)；[Compaction](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/compaction.md)
2. [DeepSeek Harness 架构](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md)；[Core 子系统](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/core.md)；[事件机制](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/framework/events.md)
3. [LangChain Agents](https://docs.langchain.com/oss/python/langchain/agents)；[Middleware](https://docs.langchain.com/oss/python/langchain/middleware)
4. [LangGraph Graph API](https://docs.langchain.com/oss/python/langgraph/graph-api)；[Persistence](https://docs.langchain.com/oss/python/langgraph/persistence)；[Interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts)
