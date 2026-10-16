# LangChain 独立使用文档

> 仓库：https://github.com/langchain-ai/langchain · Python（另有 JS）  
> 文档：https://docs.langchain.com/oss/python/langchain/overview  
> Stars（langchain 仓 GitHub API，2026-10-16 快照）：≈148k  
> 一句话：**Agent 工程平台的组件与集成层——`create_agent` harness + 海量模型/工具/检索集成；适合适配与拼装，不宜单独充当多阶段战役调度器。**

## 0. 结论先行

| 项 | 内容 |
|---|---|
| **定位** | 模型、工具、提示、中间件、预构建 Agent 与集成生态；v1 以 `create_agent` 为标准 Agent API |
| **对本平台建议** | 默认作为 **LLM / 工具 / RAG / 评估胶水（适配层）**；多阶段渗透/审计战役状态机用 **LangGraph**，不要用纯 Chain/Agent 硬撑 |
| **与相近物** | 比 LangGraph 更高层、更快上手；比 pi/dsh 弱在会话压缩与插件 OS；HITL 最终仍依赖 LangGraph checkpointer |

---

## 1. 是什么

### 1.1 定位

LangChain 提供构建 LLM 应用的标准组件与集成：聊天模型、工具、检索、记忆抽象、以及围绕「模型 + harness」的 Agent 创建方式。官方 overview 表述：Agent = Model + Harness；LangChain 提供可配置的 `create_agent` harness（提示、工具、middleware）。

**不是**：长程有状态编排的最底层运行时——那是 **LangGraph**（耐久执行、interrupt、checkpointer、Store）。

### 1.2 核心概念

| 概念 | 含义 |
|---|---|
| **`create_agent`** | LangChain 1.x 标准建 Agent 方式（替代旧 `langgraph.prebuilt.create_react_agent` 作为推荐入口） |
| **Tools** | `@tool` 等可调用能力；名称/描述/参数进入模型可见契约 |
| **Middleware** | 切入 Agent loop 的扩展点（如 HITL、摘要、护栏）；可组合 |
| **Integrations** | 独立 provider 包（`langchain-openai`、`langchain-anthropic` 等） |
| **LangSmith** | 追踪、评估与可观测（强烈建议联调） |
| **Deep Agents** | 更高层包（计划、子 Agent、文件系统等），构建在 LangGraph 之上；与「细粒度 LangChain agent」二选场景 |

### 1.3 与相近物差异

| | LangChain | LangGraph | pi / dsh |
|---|---|---|---|
| 层级 | 组件 + Agent harness | 低层图编排运行时 | 会话 harness |
| 状态/续跑 | 弱/外置；严肃 HITL 靠 Graph | Checkpointer + Store | 会话树 / 会话日志 |
| 生态 | **最广**集成面 | 编排与部署 | Node/TS 侧 |
| 安全调度贴合 | 适配层 | **战役图** | 执行壳 |

---

## 2. 应用场景

### 2.1 通用

- 快速原型：工具调用 Agent、RAG QA、ETL 式 Chain  
- 统一多模型 provider 初始化与切换  
- 与 LangSmith 联调追踪/评估  
- 用 middleware 叠加护栏、摘要、HITL 策略（底层仍 Graph）

### 2.2 安全 / Agent 调度相关

- **适配层**：把扫描器 CLI、工单 API、知识库检索封成 Tool，供 Graph 节点或 Agent 调用  
- **只读辅助 Agent**：报告润色、发现分类、检索类似历史案例（非编排内核）  
- **HITL middleware**：工具调用前审批（`HumanInTheLoopMiddleware` 等）——**必须**配置 checkpointer + `thread_id`  
- **评估**：对「是否越权调用工具」「摘要是否丢 findings」做轨迹评估  

**不建议**：用 RunnableSequence / 单一 ReAct Agent 表达「侦察→漏洞验证→报告」多阶段、带回流与不可逆分级的战役流水线。

---

## 3. 如何使用

> 摘自官方 Install / Quickstart / Agents 文档（2026-10 核对）。模型名字符串随厂商更新，示例中的具体模型 ID **不要**当作长期钉死版本。

### 3.1 安装

要求：Python **3.10+**。

```bash
pip install -U langchain
# 按需安装 provider
pip install -U langchain-openai
# 或
uv add langchain langchain-openai
```

### 3.2 最小 Agent 示例

```python
from langchain.agents import create_agent
from langchain.tools import tool

@tool
def get_weather(city: str) -> str:
    """Get weather for a city."""
    return f"Weather in {city}: sunny"

agent = create_agent(
    model="openai:gpt-4.1",  # 实际部署时换成团队已验证的 model 字符串
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Weather in Paris?"}]
})
```

来源：LangChain Agents / Quickstart / v1 release notes 中的 `create_agent` 模式（模型字符串按官方当前示例调整）。

### 3.3 关键 API / 配置面

| 面 | 说明 |
|---|---|
| **Model** | `model="provider:model_id"` 或 `init_chat_model(...)` |
| **Tools** | `@tool`、可调用对象、tool dict |
| **system_prompt** | 行为约束入口 |
| **Middleware** | 组合 HITL、摘要、限流等；HITL 需 checkpointer |
| **Checkpointer** | 传入持久化实现后，Agent 可跨 interrupt 恢复 |
| **Classic** | 旧 Chains/Retrievers 等可经 `langchain-classic` 迁移 |

### 3.4 HITL（与 Graph 的边界）

官方 Human-in-the-loop 文档说明：middleware 在工具调用匹配策略时发出 interrupt，**依赖 LangGraph 持久化层**；生产应用 Postgres/Mongo 等 saver，开发可用 `InMemorySaver`。调用时传入带 `thread_id` 的 `config`。

对安全调度：把「不可逆工具」列入 `interrupt_on` 策略，审批默认 **fail-closed**（产品策略，非框架默认保证）。

### 3.5 扩展点

- 自定义 tools / middleware  
- Provider 包与兼容网关  
- LangSmith 数据集与评估器  
- 需要长期任务能力时评估 **Deep Agents**（仍建在 LangGraph 上）

---

## 4. 效益

### 4.1 相对自研适配层

| 收益 | 代价 |
|---|---|
| 极广的模型与工具集成，降低对接成本 | 抽象层多，版本迁移（classic→v1）需治理 |
| `create_agent` + middleware 快速出原型 | 隐藏状态机细节，不利强审计战役 |
| 与 LangGraph / LangSmith 同生态 | 误用为编排内核会导致难维护的「大 Agent」 |

### 4.2 相对另三框架

- **相对 LangGraph**：上手快、集成强；编排与 HITL 深度不如直接用 Graph API。  
- **相对 pi/dsh**：Python 生态与企业集成友好；会话压缩/插件 OS 需另选执行 harness。  

### 4.3 何时该用 / 不该用

| 该用 | 不该用 |
|---|---|
| 模型/工具/RAG 适配与胶水 | 多阶段 Kill-Chain 唯一运行时 |
| 快速验证某个检测工具是否适合 Agent 化 | 需要工业级会话 compaction 开箱（看 pi） |
| 与现有 Python 数据/安全工具链集成 | 需要 Cordis 级热插拔策略 OS（看 dsh） |

---

## 5. 与本调研其他部分的关系

| 文档 | 关系 |
|---|---|
| [langgraph.md](./langgraph.md) | 编排层首选；本篇为适配层 |
| [pi-dsh-langchain-langgraph.md](./pi-dsh-langchain-langgraph.md) | 对比专章推荐栈：Graph + Harness + LangChain/LiteLLM |
| [../01-competitors/AuroraRecon.md](../01-competitors/AuroraRecon.md) | 战役在 LangGraph；工具侧仍可用 LC 风格封装 |
| [../01-competitors/DeepAudit.md](../01-competitors/DeepAudit.md) | ReAct 多 Agent 审计向参考 |

---

## 6. 参考链接

- Overview：https://docs.langchain.com/oss/python/langchain/overview  
- Install：https://docs.langchain.com/oss/python/langchain/install  
- Agents：https://docs.langchain.com/oss/python/langchain/agents  
- HITL：https://docs.langchain.com/oss/python/langchain/human-in-the-loop  
- v1 变更：https://docs.langchain.com/oss/python/releases/langchain-v1  
- 仓库：https://github.com/langchain-ai/langchain  

**公开信息缺口**：具体默认模型字符串与 middleware 类名随小版本变化；集成生产前以当前 docs 与 `pip show langchain` 为准。本调研**不**提供攻击利用工具示例。
