# pi（earendil-works/pi）独立使用文档

> 仓库：https://github.com/earendil-works/pi · TypeScript · MIT  
> 站点：https://pi.dev · npm：`@earendil-works/pi-coding-agent`  
> Stars（GitHub API，2026-10-16 快照）：≈115k  
> 一句话：**极简可扩展 Agent Harness——会话树、工具循环、工业级 compaction、扩展包；默认不做 sub-agent / plan mode。**

## 0. 结论先行

| 项 | 内容 |
|---|---|
| **定位** | 会话执行内核（tool loop + 持久会话 + 压缩 + 扩展缝），不是战役级状态机 |
| **对本平台建议** | 作为「单阶段深度推理 / 工具循环」首选参考或直接嵌入；战役编排交给 LangGraph（或 YoungFlow），领域门禁叠在 harness 外 |
| **与相近物** | 比 LangChain Agent 更偏「可审计会话运行时」；比 dsh 更轻、学习曲线更低；比纯 LangGraph 缺原生 interrupt HITL |

---

## 1. 是什么

### 1.1 定位

Pi 自称 *minimal, extensible agent harness*：提供强大默认能力，但**刻意不内建** sub-agents、plan mode 等产品化功能，鼓励用 [extensions](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/extensions.md) / [skills](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/skills.md) / 包（npm 或 git）按自己的工作流定制。

官方四种使用面：

1. **交互 TUI**（默认终端会话）  
2. **print / JSON 模式**（一次性自动化、事件流）  
3. **RPC 模式**（stdin/stdout 协议控制）  
4. **TypeScript SDK**（进程内嵌入）

### 1.2 核心概念

| 概念 | 含义 |
|---|---|
| **Agent harness** | 围绕「模型 + 工具循环」的运行时：排队、压缩、恢复、事件 |
| **Session / SessionTree** | 持久会话与分支导航；immutable entries + mutable registers |
| **Lane + Operation** | 接受 prompt、跑 model/tool、compact、resume 中断作业的状态机 |
| **Compaction** | 上下文窗口触顶时结构化摘要并保留近端 token（见下） |
| **Extensions / Skills / Packages** | 扩展工具、命令、技能、提示模板、主题；可打包分发 |
| **包结构** | `pi-ai`（多 provider）、`pi-agent-core`、`pi-coding-agent`（CLI）、`pi-durable`、`pi-tui`、`chord` |

### 1.3 与相近物差异

| | pi | dsh | LangChain Agent | LangGraph |
|---|---|---|---|---|
| 抽象 | 会话 harness | 插件化 harness OS | 组件/Agent 高层 API | 有状态图编排 |
| 默认多 Agent | 弱（扩展自建） | Agent Teams（实验） | 多种预构建 | 子图/监督一等 |
| 压缩 | **内建工业级** | 日志投影 + 渐进披露 | 需自建/中间件 | 节点内或外置 |
| 权限/沙箱 | **默认无内建权限系统**；需容器化 | sandbox + approval 插件层 | 中间件/外置 | 节点内自建 |
| 适合 | 嵌入式会话壳 | 强策略热插拔 | 快速拼装 | 战役状态机 |

论文 *Architectural Convergence in Three LLM Agent Harnesses*（arXiv:2608.23953）将 pi 与 deepagents / dsh 并列：虽哲学不同，却收敛到商品化 loop、可回放会话、模型 quirks 数据化、上下文渐进披露、显式扩展缝；并指出共同缺口——**对外可验证性（external verifiability）**。

---

## 2. 应用场景

### 2.1 通用

- 本地/团队编码 Agent（交互 TUI）  
- CI / 脚本化「print 一次任务」或 JSONL 事件采集  
- 宿主产品通过 SDK / RPC 嵌入会话能力（如 OpenClaw 类集成）  
- 需要**可分支、可恢复、可压缩**长会话的工具循环

### 2.2 安全 / Agent 调度相关（本调研语境）

- **会话执行层**：多阶段渗透/审计流水线中，单 stage 的深度 tool loop（读仓库、跑只读检测、写报告草稿）  
- **审计轨迹**：SessionTree / JSONL 会话作为执行面审计源  
- **与编排组合**：YoungFlow 每 stage 调 pi；Full-Chain-Sentinel 证明「pi + 渗透壳（Web + 容器 + 审批）」可行  
- **压缩策略参考**：长程扫描任务的 transcript 压缩默认实现  

**不适合单独承担**：跨阶段 Kill-Chain 状态机、一等公民 HITL interrupt、资产双图记忆、不可逆操作分级策略引擎——这些应放在 LangGraph / 控制面 / 业务资产包。

---

## 3. 如何使用

> 以下安装与 API 摘自官方 README / CLI / SDK 文档（2026-10 核对）。版本与 flag 以当前安装的 `pi --help` 为准。

### 3.1 安装与入口

**要求**：Node.js **22.19+**（官方 README）。

```bash
# 推荐：官方安装器（钉住传递依赖）
curl -fsSL https://pi.dev/install.sh | sh

# 或 npm（不钉传递依赖）
npm install -g --ignore-scripts @earendil-works/pi-coding-agent

# Nix
nix run github:earendil-works/pi/stable
```

```bash
cd /path/to/project
pi
# 交互内 /login 连接订阅或 API key
```

### 3.2 最小 CLI 用法

```bash
# 交互
pi

# 一次性打印结果
pi --print "Summarize this repository"

# JSONL 事件流（自动化/可观测）
pi --mode json "Inspect this repository" > events.jsonl

# 限制工具白名单
pi --tools read,grep,find,ls --print "Review this project"

# 续跑最近会话
pi --continue
```

来源：`packages/coding-agent/docs/cli.md`。

### 3.3 最小 SDK 用法（TypeScript）

```typescript
import { createAgentSession } from "@earendil-works/pi-coding-agent";

const { session } = await createAgentSession();

try {
  await session.prompt("What files are in the current directory?");
  console.log(session.getLastAssistantText());
} finally {
  session.dispose();
}
```

内存会话（不落盘）：

```typescript
import { createAgentSession, SessionManager } from "@earendil-works/pi-coding-agent";

const { session } = await createAgentSession({
  sessionManager: SessionManager.inMemory(),
});
```

来源：`packages/coding-agent/docs/sdk.md`。

### 3.4 关键配置面与扩展点

| 面 | 说明 |
|---|---|
| **Models** | `--model` / `--provider` / `--list-models`；多 provider 经 `pi-ai` |
| **Tools** | `--tools` allowlist；设置里默认工具集；扩展可注册工具 |
| **Sessions** | `--continue` / `--resume` / `--session` / `--fork` / `--no-session` |
| **Modes** | 交互、`--print`、`--mode json`、`--mode rpc` |
| **Packages** | `pi install <source>`；extensions / skills / prompt templates / themes |
| **Compaction** | 阈值约 `contextTokens > contextWindow - reserveTokens`（默认 reserve 16384）；保留近端 `keepRecentTokens`（默认约 20000）；扩展可拦 `session_before_compact`（细节见 coding-agent `docs/compaction.md`，Full-Chain-Sentinel 仓内有同步副本） |
| **权限** | **无内建** FS/进程/网络权限系统；官方建议 Docker / Gondolin / OpenShell 等容器化（`docs/containerization.md`） |

### 3.5 扩展点（产品化时常用）

- 自定义 tools / commands / skills  
- 拦截压缩前后事件  
- RPC / JSON 模式对接控制面 Worker  
- 整进程或工具侧容器化，再在产品层叠审批三档（Sentinel 路径）

---

## 4. 效益

### 4.1 相对自研会话循环

| 收益 | 代价 |
|---|---|
| 省去会话持久化、分支、崩溃恢复、压缩重试等工程 | 绑定 TypeScript/Node 栈与 pi 会话格式 |
| 四种使用面覆盖交互与自动化 | 默认能力边界需用扩展补齐（多 Agent 等） |
| 扩展包生态可版本化分发 | **沙箱与权限必须自建**（安全产品硬成本） |
| Full-Chain-Sentinel / YoungFlow 已有落地样板 | 上游演进节奏需跟进；API 以官方 docs 为准 |

### 4.2 相对另三框架

- **相对 dsh**：更轻、文档入口更「产品化 CLI」；缺一等插件 OS 与内建 approval/sandbox 层。  
- **相对 LangChain**：更强的会话审计与 compaction；集成面与 RAG 生态不如 LangChain。  
- **相对 LangGraph**：不做战役图；应组合使用而非二选一。

### 4.3 何时该用 / 不该用

| 该用 | 不该用 |
|---|---|
| 需要可嵌入的 tool loop + 工业压缩 | 只要链式 Prompt（用 LangChain 即可） |
| 希望 JSONL/RPC 对接调度 Worker | 需要原生 `interrupt()` 审批图（用 LangGraph） |
| 团队接受「领域策略在壳外」 | 期望开箱即用的多租户沙箱与 RBAC |

---

## 5. 与本调研其他部分的关系

| 文档 | 关系 |
|---|---|
| [pi-dsh-langchain-langgraph.md](./pi-dsh-langchain-langgraph.md) | 四框架**对比专章** |
| [youngflow.md](./youngflow.md) | YoungFlow 每 stage 调 **pi** 进程 |
| [../01-competitors/Full-Chain-Sentinel-System.md](../01-competitors/Full-Chain-Sentinel-System.md) | pi fork 渗透控制台样板 |
| [../01-competitors/vulnhunter.md](../01-competitors/vulnhunter.md) | 经 YoungFlow 间接依赖 pi |
| [../00-index.md](../00-index.md) | 选型结论：会话执行层参考 pi |

---

## 6. 参考链接

- 仓库：https://github.com/earendil-works/pi  
- 文档站：https://pi.dev/docs/latest  
- CLI：`packages/coding-agent/docs/cli.md`  
- SDK：`packages/coding-agent/docs/sdk.md`  
- Compaction / Containerization：同目录 `compaction.md`、`containerization.md`  
- 收敛论文：https://arxiv.org/abs/2608.23953  

**公开信息缺口**：具体 minor 版本号与默认 compaction 数值以安装版本 docs 为准；本页不钉死某一 release tag。权限模型官方明确「无内建限制」——产品威胁模型须单独写清。

---

## 第二版补充：Agent Loop 与产品层边界

pi-ai 负责模型适配；pi-agent-core 负责模型—工具循环；coding-agent 承载 CLI、会话与压缩等产品能力。需区分各包职责，不应将所有会话能力归属于核心 Loop。

验证重点：逐一跟踪 prompt→模型响应→工具执行→结果回填→再次请求；覆盖工具异常、取消、并行调用、上下文压缩与会话恢复。

> 本节为第一版的增量修订；尚未完成固定 Commit 的逐函数源码审计。
