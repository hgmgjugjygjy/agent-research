# Agent 工程架构走读

> 主文 · 以工程模块与数据流为主；外部示意图仅附录  
> 相关：[01-what-is-an-agent.md](01-what-is-an-agent.md) · [glossary](glossary/README.md)

---

## 0. 结论先行

| 要点 | 说明 |
|---|---|
| **架构不是一张能力海报** | 要能回答：状态存在哪、工具谁批准、失败怎么回、人何时介入 |
| **推荐分层** | 接入 → 编排(Orchestration) → 会话执行(Agent Loop) → 工具/MCP → 记忆/RAG → 门禁/HITL → 观测 |
| **本仓默认取向** | 编排偏 LangGraph；会话偏 pi；门禁 fail-closed；检索与记忆分家 |

---

## 1. 推荐逻辑分层（本仓口径）

```mermaid
flowchart TB
  U[用户 / 上游系统] --> API[接入层<br/>目标·权限·租户]
  API --> ORCH[编排层 Orchestration<br/>阶段图 / 状态机 / 并行 / interrupt]
  ORCH --> LOOP[会话执行层<br/>Agent Loop / ReAct / pi harness]
  LOOP --> LLM[LLM 推理核]
  LOOP --> TOOLS[Tools / Function calling]
  TOOLS --> MCP[MCP / 连接器 / 沙箱]
  LOOP --> MEM[Memory<br/>短·长·工作 / checkpointer]
  LOOP --> RAG[RAG<br/>向量库·chunk·rerank]
  ORCH --> GR[Guardrails / HITL]
  LOOP --> GR
  TOOLS --> GR
  ORCH --> OBS[Observability<br/>trace·成本·回放]
  LOOP --> OBS
  LOOP --> REF[Reflection / Eval]
  REF --> ORCH
```

| 层 | 负责 | 不负责 |
|---|---|---|
| 接入 | 鉴权、目标归一、配额 | 具体推理 |
| 编排 | 阶段、分支、战役进度、跨 Agent 派发 | 单步 tool 细节 |
| 会话执行 | 热循环、消息历史、压缩 | 全局战役拓扑 |
| 工具/MCP | 副作用与外部 I/O | 是否允许调用（应交禁裁） |
| Memory | 任务内/跨任务状态 | 整库文档问答（那是 RAG） |
| RAG | 外部知识取证 | 替代工作记忆 |
| 门禁/HITL | 范围、不可逆、审批 | 替代业务逻辑 |
| 观测 | 可审计、可复盘 | 自动「变聪明」 |

### 1.1 渲染版：系统架构示意图

![系统架构示意图](assets/system-architecture.svg)

*图 A：系统架构。自上而下：入口网关（鉴权、限流、输入护栏）→ 编排层（状态机/检查点）→ Agent 内核（Planner→LLM 推理→Reflection，不通过则回到 Planner）。内核向右连**工具层**（function calling，经 **MCP** 接外部系统），向下连 **RAG**（向量库+重排）与 **Memory**（短期上下文/长期存储）；编排层挂 **HITL**；输出前过**输出护栏**；虚线表示 **Trace/Eval** 横切所有层。PNG：[system-architecture.png](assets/system-architecture.png)*

### 1.2 组件关系图

![组件关系图](assets/component-relations.svg)

*图 B：组件关系（类图）。Orchestrator 调度多个 Agent；Agent **拥有** Memory（组合），**可调用**多个 Tool（聚合），并依赖 LLM、Planner、Retriever、Guardrail；MCPClient 把远端工具**动态注册**为 Tool；Retriever 也服务于长期记忆检索。读法：实心菱形＝生命周期绑定，空心菱形＝可插拔。PNG：[component-relations.png](assets/component-relations.png)*

---

## 2. 一次任务的端到端数据流（中性示例）

**目标（中性）**：根据内部知识库与公开文档，整理某开源竞品的「调度与门禁」要点，写入 Notion，并在 Slack 通知负责人。

| 步 | 模块 | 发生什么 |
|---|---|---|
| 1 | 接入 | 校验用户身份与「可写 Notion / 可发 Slack」权限 |
| 2 | Planner | 拆成：检索资料 → 摘要对比 → 写页面 → 通知 |
| 3 | RAG | 从竞品笔记库检索相关 chunk；不足再 Web 检索 |
| 4 | Agent Loop | 多轮 tool：search → read → draft |
| 5 | Memory | 工作记忆记下已读来源与草稿路径 |
| 6 | Reflection | 检查是否覆盖「调度/门禁」两节；缺则回流 |
| 7 | Guardrails | 「发 Slack」属对外可见 → **HITL 审批** |
| 8 | 执行 | 写 Notion；审批通过后发 Slack |
| 9 | Observability | 留下 trace：用了哪些 tool、token、审批人 |

此例刻意**不含**扫描利用类动作，只演示闭环与门禁切面。

![数据流图](assets/dataflow.svg)

*图 C：端到端数据流。1 请求 → 2 理解（意图/约束，读记忆）→ 3 规划（任务 DAG）→ 4 判断下一步需要「外部动作」还是「知识」→ 5a 工具调用 / 5b RAG 检索 → 6 观察写入记忆 → 7 反思：未完成或有错回到 3，完成进入 8 输出（过护栏）。上表 9 步即此图的一次具体实例：表第 3 步对应 5b，第 4/8 步对应 5a，第 6 步对应 7，第 7 步的 HITL 发生在 5a 执行前（见[门禁时序](glossary/guardrails-hitl.md)）。PNG：[dataflow.png](assets/dataflow.png)*

---

## 3. 编排 vs 会话执行（选型常混）

| | Orchestration（编排） | Agent Loop / Harness（会话执行） |
|---|---|---|
| 粒度 | **阶段 / 战役 / 多 Agent** | **单会话内** think-act-observe |
| 状态 | 图节点、checkpoint、业务实体 | 消息列表、tool 结果、压缩摘要 |
| 典型技术 | LangGraph、工作流引擎 | pi、dsh、LangChain AgentExecutor |
| HITL 位置 | 节点 interrupt、阶段门 | 单次 tool 前确认 |
| 本仓建议 | 长流水线用编排托底 | 每阶段内用 harness 跑热循环 |

详见 [orchestration](glossary/orchestration.md) · [agent-loop](glossary/agent-loop.md) · [pi](../02-frameworks/pi.md) · [langgraph](../02-frameworks/langgraph.md)。

---

## 4. 门禁切面应插在哪

```mermaid
flowchart LR
  Q[待执行动作] --> S{范围/租户 OK?}
  S -->|否| X[拒绝]
  S -->|是| R{确定性规则<br/>YAML/策略}
  R -->|拒| X
  R -->|过| I{不可逆 / 对外可见?}
  I -->|是| H[HITL 审批]
  I -->|否| Y[允许]
  H -->|批准| Y
  H -->|拒绝| X
```

原则：**默认 fail-closed**（不确定则拦）。产品化参考 Aurora 四层；概念见 [guardrails-hitl](glossary/guardrails-hitl.md)。

---

## 5. 上下文爆炸时怎么拆

长程任务失败的头号工程原因往往是**窗满 + 失忆 + 重复劳动**，不是「模型不够大」。

| 手段 | 作用 | 专篇 / 落点 |
|---|---|---|
| Context compaction | 摘要/折叠历史，保留关键事实与待办 | [context-compaction](glossary/context-compaction.md)、pi |
| 结构化 Memory | 图、工件、checklist 代替纯聊天记录 | [memory](glossary/memory.md)、ARTEX 双图 |
| RAG 外置知识 | 大文档不进权重、不进永久窗口 | [rag](glossary/rag.md) |
| 阶段切分 | 每阶段新会话 + 只传 digests | Orchestration |

系统讲解见 [06-context-engineering.md](06-context-engineering.md) 与 [context-explosion](glossary/context-explosion.md)。

---

## 6. 技术实现原理：一个最小 Agent 运行时

把上面各层串起来的骨架（伪代码，省略错误处理）：

```python
class AgentRuntime:
    def __init__(self, llm, tools, memory, retriever, policy, tracer):
        self.llm, self.tools, self.mem, self.rag, self.policy, self.tr = llm, tools, memory, retriever, policy, tracer

    def run(self, goal, max_steps=20):
        spec = understand(goal)                          # 任务理解 → 结构化规格
        msgs = [system_prompt(spec), *self.mem.recall(goal)]
        for step in range(max_steps):
            with self.tr.span("llm"):
                r = self.llm(msgs, tools=self.tools.schemas())   # 推理：决定下一步
            if not r.tool_calls:
                return self.policy.check_output(r.content)       # 输出护栏
            for call in r.tool_calls:
                if not self.policy.allow(call):                  # 门禁 / HITL
                    msgs.append(denied(call)); continue
                with self.tr.span("tool", name=call.name):
                    obs = self.tools.execute(call)               # 执行（沙箱/超时）
                msgs.append(tool_msg(call, obs))
            msgs = compact_if_needed(msgs)                       # 上下文治理
            self.mem.write(extract_facts(msgs[-4:]))             # 记忆写入
        return "超出步数上限"
```

**为什么这样分层**：LLM 只做「判断」，确定性代码掌控「执行、权限、预算、终止」；每层可单独替换（换模型、换向量库、换编排引擎）与单独测试。编排层（图/状态机）在此运行时之外再包一层，负责阶段与检查点。

## 7. 附录：外部示意图对照

![参考：完整架构](assets/agent-complete-architecture.jpg)

> 外部面板强调 Planner / Reasoning / Loop / Reflection + Memory / Tool / MCP。对照本仓分层：请把 **Orchestration、RAG、Guardrails、Observability** 视为一等公民，而不是「Loop 里顺便做」。

![参考：七层能力](assets/agent-chain-of-thought-overview.jpg)

> 七层能力图适合对产品讲「智能体有什么本事」；落地架构仍以第 1 节分层为准。

---

## 8. 读完下一步

1. 按需查 [glossary/](glossary/README.md)（建议先：agent-loop · rag · orchestration · guardrails-hitl · context-compaction）  
2. 再进 [02-frameworks/](../02-frameworks/) 与 [01-competitors/](../01-competitors/)  
3. 总选型见根 [00-index.md](../00-index.md)
