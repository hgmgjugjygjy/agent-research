# dsh（DeepSeek Harness）独立使用文档

> 仓库：https://github.com/deepseek-ai/deepseek-harness · TypeScript · MIT  
> 文档：https://deepseek-harness.github.io/deepseek-harness/  
> Stars（GitHub API，2026-10-16 快照）：≈251k  
> 一句话：**Everything is a Plugin——基于 Cordis 的插件化 Agent Harness；会话日志为真相源，沙箱与审批策略可作一等插件层。**

## 0. 结论先行

| 项 | 内容 |
|---|---|
| **定位** | 插件化 harness「操作系统」：模型适配、工具注册、会话日志、Agent loop、沙箱/审批均可热插拔替换 |
| **对本平台建议** | 作**执行面能力缝与策略插件中心**的强参考或候选；与 LangGraph 分工（Graph=战役，dsh=会话执行）。接受 Cordis 学习成本与 **developer preview** 破坏性变更 |
| **与相近物** | 比 pi 更「OS 化」、扩展更彻底；比 LangGraph 缺战役图语义；渗透**预制技能少**，需自研插件 |

---

## 1. 是什么

### 1.1 定位

DeepSeek Harness（CLI 名 **`dsh`**）是 DeepSeek AI 开源的 Agent Harness，架构口号为 **Everything is a Plugin**，底层为 [Cordis](https://github.com/cordiverse/cordis)（时空可组合插件范式，见 arXiv:2608.25512）。

官方状态（README）：**developer preview**，快速迭代，**将有破坏兼容的变更**。运行前须阅读仓库 `SAFETY.md` / `SAFETY.zh.md`。

### 1.2 核心概念

| 概念 | 含义 |
|---|---|
| **Profile** | 命名组合：`web` / `headless` / `sdk` / `sdk-minimal` / `acp`；堆叠 bundles + `cordis.patch.yml` |
| **Bundle** | Cordis 配置行与代码的分发单元；上层 patch 可整行替换 |
| **dsh-base** | `web`/`headless`/`sdk`/`acp` 共享首层：模型、工具、持久化、**sandbox and approval policy**、设置、凭证、遥测 |
| **Session log** | 追加写 `SessionEvent`；**模型可见 ⟺ 已入日志**（`deriveMessages()` 从日志投影历史） |
| **Turn / Step** | Turn = 零或多 Step；Step = 一次模型请求 + 其工具调用 |
| **Capability seam** | fs / subprocess / tools 等可整体指向远程沙箱的能力缝 |
| **瀑布事件** | `agent/pre-step`、`tools/pre-execute` 等须 `next()`；可拒收或改写 |

### 1.3 核心包与 ctx 键（官方 architecture.md）

| 包 | 职责 | ctx |
|---|---|---|
| core/session | 追加写 SessionEvent 日志 | `ctx.sessions` |
| core/system-prompt | 提示与工具 schema 装配 | `ctx.systemPrompt` |
| core/tools | 作用域工具注册与守卫执行管道 | `ctx.tools` |
| core/agent | Agent 接口与事件 | `ctx.agents` |
| core/agent-loop | 默认驱动 | `ctx.agentLoop` |
| llm/llm | 流式与适配缝 | `ctx.llm` |

### 1.4 Turn 流（文档化）

```
turn/start → claim input → assemble prompt → agent/pre-step
  → step/start → agent/request → stream → tool/call* (pre/execute/post)
  → step/end → … → agent/turn-stopping → turn/end
```

### 1.5 与相近物差异

| | dsh | pi | LangGraph |
|---|---|---|---|
| 哲学 | 一切皆插件 / Cordis | 极简 harness + 扩展包 | 图状态机 |
| 会话真相源 | SessionEvent 日志 | SessionTree / durable | Checkpointer 图状态 |
| 门禁 | **approval + sandbox 插件层** | 默认无内建权限 | 节点内 / interrupt |
| 多 Agent | Experimental Agent Teams | 扩展自建 | 子图/监督 |
| 成熟度标签 | developer preview | 产品化 CLI 较强 | 生产编排广泛 |

---

## 2. 应用场景

### 2.1 通用

- 本地 Web UI 编码/研究 Agent（`dsh web`）  
- Headless 一次性跑批  
- SDK / ACP 自动化集成  
- 需要**把沙箱、审批、工具、模型适配全部插件化**的宿主产品

### 2.2 安全 / Agent 调度相关

- **策略与能力缝**：将 `ctx.fs` / `ctx.subprocess` / tools 指向隔离执行环境；审批策略作为插件而非散落 if  
- **可审计执行面**：会话日志为模型上下文唯一真相源，利于合规回放与迁移规则治理  
- **与 LangGraph 组合**：Graph 管阶段与 HITL interrupt；dsh 管单阶段 tool loop 与策略拦截（`agent/pre-step`、`tools/pre-execute`）  
- **插件生态**：社区用 `dsh-plugin` topic 发现扩展

**缺口（对本平台）**：公开文档完善，但**渗透/漏洞挖掘领域预制技能与安全流水线资产包少**——业务技能、gate 策略、报告模板仍需自研（对比 VulnHunter 资产包挂在 YoungFlow+pi 上）。

---

## 3. 如何使用

> 摘自官方 README / architecture / Web UI guide（2026-10 核对）。preview 阶段命令与配置可能变更。

### 3.1 安装与入口

**通过 npm（需已装 Node.js）：**

```sh
npx @deepseek-ai/dsh web
```

默认 Web UI：`http://127.0.0.1:3080`；本机启动会尝试打开浏览器。SSH 场景只打印 URL。`--no-open` 仅起服务。

**从源码：**

```sh
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
pnpm install
pnpm run build
pnpm dsh web
```

### 3.2 最小使用路径（Web）

1. 启动 `dsh web`  
2. **Settings → Models**：配置 API key（如 DeepSeek；亦可按文档配其他/兼容端点）  
3. **Choose workspace**：选定工作区目录（未选则无法发会话）  
4. 开会话，发送任务；按当前 permission policy，敏感操作会要求批准  

来源：`docs/user/guide/index.md`。

### 3.3 Profile 与配置面

```sh
# 查看当前机器将启动的插件树（可被 patch 替换的每一行）
dsh --profile web --dump-config

# 或以简写选择 profile
dsh web
dsh headless
dsh sdk
```

| Profile | 用途（官方描述摘要） |
|---|---|
| `web` | 浏览器应用 + base |
| `headless` | 无服务器的一次性 runner |
| `sdk` | SDK JSON-RPC 服务 |
| `sdk-minimal` | 显式最小 SDK 树（**不**叠 dsh-base） |
| `acp` | 自动化 ACP 服务 |

分层：`profile bundles` → 仓库/用户 `cordis.patch.yml` → `--patch`。Python SDK 亦启动同版本 `dsh --profile sdk`，通过 profile + 有序 patch 组合，而非裸 Cordis 树。

### 3.4 扩展点（选型相关）

| 目标 | 机制（architecture「Where new behavior goes」） |
|---|---|
| 加模型 | `ctx.llm` 注册适配器 |
| 加工具 | `ctx.tools`；schema 进入 prompt 装配 |
| 拦截请求/工具/回合 | `agent/*`、`tools/*` 瀑布；`agent/turn-stopping` |
| 换 FS / 沙箱 | `ctx.fs` / `ctx.sandbox`；seam 可整体指远程 |
| 持久会话后端 | 实现 `SessionPersistence` |
| 实验多 Agent | `ctx.agentTeams`（花名册 + 任务状态 + inbox） |

插件发现：GitHub topic [`dsh-plugin`](https://github.com/topics/dsh-plugin)。

---

## 4. 效益

### 4.1 相对自研插件运行时

| 收益 | 代价 |
|---|---|
| 统一插件生命周期与可逆 effect（Cordis） | Cordis + 事件图学习曲线**高** |
| 会话日志驱动可回放、可迁移 | developer preview：破坏性变更风险 |
| 沙箱/审批可一等插件化 | 运维面（HMR、profile、bundle）复杂 |
| 能力缝一次替换整类工具执行世界 | 领域技能仍要自研 |

### 4.2 相对另三框架

- **相对 pi**：策略与沙箱缝更一等；CLI/SDK「开箱编码体验」与 compaction 产品文档不如 pi 直观。  
- **相对 LangChain**：执行面与策略治理更强；生态集成广度不如 LangChain。  
- **相对 LangGraph**：不替代战役编排；适合作为 Graph 节点内的执行运行时。

### 4.3 何时该用 / 不该用

| 该用 | 不该用 |
|---|---|
| 需要热插拔安全策略与远程沙箱 seam | 团队无 TypeScript/Cordis 带宽 |
| 强调「模型可见=已记录」的合规回放 | 只要稳定 API 的长期锁定依赖（preview） |
| 与自研控制面深度定制执行面 | 只想要 YAML 流水线胶水（可优先评 YoungFlow+pi） |

---

## 5. 与本调研其他部分的关系

| 文档 | 关系 |
|---|---|
| [pi-dsh-langchain-langgraph.md](./pi-dsh-langchain-langgraph.md) | 对比专章；推荐「Graph 管阶段，Harness 管子会话」 |
| [pi.md](./pi.md) | 同层会话 harness 的轻量对照 |
| [youngflow.md](./youngflow.md) | YoungFlow 当前绑定 **pi**，非 dsh；替换执行器属架构分叉 |
| [../00-index.md](../00-index.md) | 公开信息缺口表：dsh 架构公开，渗透预制技能少 |

---

## 6. 参考链接

- 仓库：https://github.com/deepseek-ai/deepseek-harness  
- 文档站：https://deepseek-harness.github.io/deepseek-harness/  
- 架构：`docs/architecture.md`（中文 `architecture.zh.md`）  
- 安全说明：`SAFETY.md` / `SAFETY.zh.md`  
- Cordis 范式论文：https://arxiv.org/abs/2608.25512  
- Harness 收敛论文：https://arxiv.org/abs/2608.23953  

**公开信息缺口**：渗透领域官方技能包数量与稳定 SDK 语义版本策略——以 preview 公告为准；本页不编造未文档化的 Python/TS API 签名细节。
