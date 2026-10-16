# LangGraph 独立使用文档

> 仓库：https://github.com/langchain-ai/langgraph · Python（另有 JS：langgraphjs）· MIT  
> 文档：https://docs.langchain.com/oss/python/langgraph/overview  
> Stars（GitHub API，2026-10-16 快照）：≈43k  
> 一句话：**低层有状态图编排运行时——耐久执行、Checkpointer、interrupt HITL、Store；自建漏洞扫描/挖掘调度平台的战役编排首选。**

## 0. 结论先行

| 项 | 内容 |
|---|---|
| **定位** | 用节点/边（或 Functional API）表达长程有状态 Agent/工作流；可混用确定性节点与 LLM 节点 |
| **对本平台建议** | **战役编排层首选**（阶段、回流、审批 interrupt、Postgres 续跑）；会话内 tool loop / compaction 仍可委派 pi 或 dsh |
| **与相近物** | 比 LangChain Agent 更可控、可审计；比 pi/dsh 强在多阶段状态机；YoungFlow 在内部用 LangGraph 跑 YAML flow |

---

## 1. 是什么

### 1.1 定位

LangGraph 是面向**长运行、有状态** Agent/工作流的低层编排框架。官方强调：

- **Durable execution**：失败后从检查点恢复  
- **Human-in-the-loop**：任意点暂停、审查、恢复  
- **Memory**：Checkpointer（线程内短记忆）+ Store（跨线程长记忆）  
- **可观测**：LangSmith tracing  
- **可部署**：面向有状态工作流的部署选项  

可与 LangChain 组件配合，但**不必**依赖 LangChain 才能建图。

### 1.2 核心概念

| 概念 | 含义 |
|---|---|
| **StateGraph** | 声明状态类型、节点、边、条件边的主 API |
| **State** | 贯穿执行的共享状态（如 messages、scope、findings 标志） |
| **Checkpointer** | 按 `thread_id` 持久化超级步快照；HITL 与容错的前提 |
| **Store** | 跨线程长期记忆（偏好、事实、共享知识） |
| **`interrupt()`** | 节点内动态暂停，向调用方抛出可恢复中断；用 `Command(resume=…)` 恢复 |
| **Subgraph / 多 Agent** | 子图、supervisor、swarm 等模式可组装 |
| **Functional API** | `@entrypoint` / `@task`：用普通控制流代替显式节点边 |

### 1.3 与相近物差异

| | LangGraph | LangChain | pi / dsh | YoungFlow |
|---|---|---|---|---|
| 主职 | 图编排运行时 | 组件/Agent 集成 | 会话 harness | YAML flow runner（内嵌 LG+pi） |
| HITL | **原生 interrupt** | middleware→Graph | 扩展/插件 | 偏产物文件路由信号 |
| 压缩 | 自建节点或外置 | 中间件 | 工业级/日志投影 | 写入 pi settings |
| 适合 | 战役状态机 | 适配层 | 单会话深度循环 | 可运营流水线资产包 |

---

## 2. 应用场景

### 2.1 通用

- 多步骤业务工作流（审批、分支、重试）  
- 对话式 Agent 需跨轮持久与时间旅行调试  
- 多 Agent：监督者路由、子图隔离  
- 生产环境需要 Postgres 等持久 checkpointer

### 2.2 安全 / Agent 调度相关（本调研核心）

AuroraRecon 是最贴近的开源样板：

| 模式 | 用法 |
|---|---|
| **Linear Kill-Chain DAG** | 阶段节点顺序推进；关键节点前 `interrupt` |
| **Feedback** | 失败回流边再规划 |
| **Supervisor** | 星形路由动态选阶段 |
| **状态承载** | findings / authorized_scope / autonomy_level / approved 等 |

典型门禁落点（产品层，非框架内置安全语义）：

1. Scope 强制节点  
2. 确定性 YAML safety  
3. 不可逆操作分级 → interrupt  
4. 自治档位只影响「可自动的可逆动作」，**不**取消不可逆审批  

与 **pi/dsh**：Graph 派生子会话；Harness 负责压缩与工具循环。与 **YoungFlow**：若选 YAML 资产包路线，引擎内部已用 LangGraph；深度 HITL 仍可能要在控制面或 Graph 节点补强。

---

## 3. 如何使用

> 摘自官方 Quickstart / Interrupts / Persistence（2026-10 核对）。

### 3.1 安装

```bash
pip install -U langgraph
# 常用：同时装 langchain 工具与模型辅助
pip install -U langchain
```

### 3.2 最小 Graph API 示例（结构示意）

以下为官方计算器 Agent 模式的**结构缩写**（完整逐步代码见 Quickstart）。生产请换团队核定的模型与工具。

```python
from typing import Literal
import operator
from typing_extensions import Annotated, TypedDict

from langchain.tools import tool
from langchain.chat_models import init_chat_model
from langchain.messages import AnyMessage, SystemMessage, HumanMessage, ToolMessage
from langgraph.graph import StateGraph, START, END

model = init_chat_model("claude-sonnet-4-6", temperature=0)  # 示例字符串，按文档替换

@tool
def add(a: int, b: int) -> int:
    """Adds a and b."""
    return a + b

tools = [add]
tools_by_name = {t.name: t for t in tools}
model_with_tools = model.bind_tools(tools)

class MessagesState(TypedDict):
    messages: Annotated[list[AnyMessage], operator.add]
    llm_calls: int

def llm_call(state: MessagesState):
    return {
        "messages": [
            model_with_tools.invoke(
                [SystemMessage(content="You are a helpful arithmetic assistant.")]
                + state["messages"]
            )
        ],
        "llm_calls": state.get("llm_calls", 0) + 1,
    }

def tool_node(state: MessagesState):
    result = []
    for tc in state["messages"][-1].tool_calls:
        observation = tools_by_name[tc["name"]].invoke(tc["args"])
        result.append(ToolMessage(content=str(observation), tool_call_id=tc["id"]))
    return {"messages": result}

def should_continue(state: MessagesState) -> Literal["tool_node", END]:
    if state["messages"][-1].tool_calls:
        return "tool_node"
    return END

builder = StateGraph(MessagesState)
builder.add_node("llm_call", llm_call)
builder.add_node("tool_node", tool_node)
builder.add_edge(START, "llm_call")
builder.add_conditional_edges("llm_call", should_continue, ["tool_node", END])
builder.add_edge("tool_node", "llm_call")
agent = builder.compile()

agent.invoke({"messages": [HumanMessage(content="Add 3 and 4.")]})
```

来源：https://docs.langchain.com/oss/python/langgraph/quickstart 。

### 3.3 HITL：interrupt + checkpointer

```python
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.types import interrupt, Command

# 在节点内：
# decision = interrupt({"action": "irreversible_op", "detail": "..."})
# 调用方恢复：
# graph.invoke(Command(resume=True), config={"configurable": {"thread_id": "eng-001"}})

checkpointer = InMemorySaver()  # 开发用；生产用 PostgresSaver / AsyncPostgresSaver 等
graph = builder.compile(checkpointer=checkpointer)
config = {"configurable": {"thread_id": "engagement-001"}}
```

要点（官方 Interrupts / Persistence）：

- interrupt **必须**启用 checkpointer  
- `thread_id` 是恢复指针  
- 恢复时节点从**开头重跑**，interrupt 之前的副作用须幂等  
- 内存 saver 进程重启即丢；生产用 DB 型 saver  

### 3.4 关键配置面

| 面 | 说明 |
|---|---|
| **Persistence** | Checkpointer（短）+ Store（长） |
| **Streaming** | `stream` / `stream_events`；interrupt 载荷可出现在流事件中 |
| **部署** | LangSmith Deployment / Agent Server（托管时可隐藏部分持久化装配） |
| **多 Agent** | 子图、supervisor 等模式 |
| **与 LC** | 节点内随意调用 LangChain tools/models |

### 3.5 扩展点（安全平台）

- 自定义状态字段承载 scope / autonomy / gate 结果  
- 条件边实现回流与 supervisor 路由  
- 审批节点统一 `interrupt`，控制面负责人机 UI  
- 子图隔离「侦察」与「验证」权限集  
- 需要 YAML 运营化时评估 YoungFlow（LG+pi 胶水）

---

## 4. 效益

### 4.1 相对自研状态机

| 收益 | 代价 |
|---|---|
| 持久化、HITL、时间旅行、流式已产品化 | 需理解超级步、幂等、thread 模型 |
| Aurora 等开源样板可抄模式 | 图变复杂后可观测依赖 LangSmith 等 |
| Python 生态与安全工具链易接 | 单会话工业压缩仍要外置 harness |
| 与控制面 API 清晰：invoke/resume | 不提供安全策略语义（须自建门禁节点） |

### 4.2 相对另三框架

- **相对 LangChain**：可审计阶段边界与审批点；集成广度靠搭配 LC。  
- **相对 pi/dsh**：战役级编排明显更强；执行壳能力需组合。  
- **相对 YoungFlow**：更灵活的代码定义图与原生 interrupt；YoungFlow 胜在 flow 资产包运营。

### 4.3 何时该用 / 不该用

| 该用 | 不该用 |
|---|---|
| 多阶段、需审批、需续跑的调度内核 | 单次工具调用脚本（过度） |
| 要明确状态 schema 与边条件 | 团队只想维护 YAML、不愿写图代码（评 YoungFlow） |
| 与 Postgres 控制面共享 engagement id 作 thread_id | 期望框架内置漏洞利用能力（本调研范围外且不应内嵌） |

---

## 5. 与本调研其他部分的关系

| 文档 | 关系 |
|---|---|
| [langchain.md](./langchain.md) | 适配层；Graph 为编排层 |
| [pi.md](./pi.md) / [dsh.md](./dsh.md) | 执行 harness 组合对象 |
| [youngflow.md](./youngflow.md) | 内部使用 `@langchain/langgraph` + pi |
| [../01-competitors/AuroraRecon.md](../01-competitors/AuroraRecon.md) | LangGraph Kill-Chain + 分层门禁蓝本 |
| [pi-dsh-langchain-langgraph.md](./pi-dsh-langchain-langgraph.md) | 对比与推荐组合图 |
| [../00-index.md](../00-index.md) | 结论第 1 条：编排层选 LangGraph |

---

## 6. 参考链接

- Overview：https://docs.langchain.com/oss/python/langgraph/overview  
- Quickstart：https://docs.langchain.com/oss/python/langgraph/quickstart  
- Interrupts：https://docs.langchain.com/oss/python/langgraph/interrupts  
- Persistence：https://docs.langchain.com/oss/python/langgraph/persistence  
- Checkpointers：https://docs.langchain.com/oss/python/langgraph/checkpointers  
- 仓库：https://github.com/langchain-ai/langgraph  

**公开信息缺口**：各语言 SDK 功能对齐以各自 docs 为准；生产 saver 的具体包名（Postgres 等）随发行物拆分，集成前核对当前 Persistence 文档。本调研不包含 exploit/PoC。
