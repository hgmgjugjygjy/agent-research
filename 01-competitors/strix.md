# Strix（usestrix/strix）竞品研究

> 仓库：https://github.com/usestrix/strix · Python · Apache-2.0 · Stars ≈68.8k（2026-10-16）  
> 产品定位：开源 AI 渗透测试 Agent；另有 Cloud / Enterprise。

## A. 定位与技术栈

| 项 | 内容 |
|---|---|
| 定位 | 自主 AI 渗透测试：动态跑目标、验证发现、给出修复建议；开发者优先 CLI |
| 运行时 | Python；OpenAI Agents SDK 风格会话；LiteLLM 多模型 |
| 沙箱 | Docker 沙箱镜像；工具含 HTTP 代理（Caido）、Browser、Shell、Python 运行时 |
| UI | Bubble Tea TUI + 本地 Web Viewer（Agent 图谱、发现列表）；Cloud 平台另线 |
| 集成 | GitHub Actions / CI、MCP servers、`npx skills add usestrix/strix` |

## B. Agent 拓扑

**Graph of Agents（多 Agent 树）**：

- Root 扫描 Agent + 按需 `create_agent` 派生专家子 Agent
- `AgentCoordinator` 拥有 statuses / parent_of / mailbox / wake Event
- 工具面：`view_agent_graph` / `send_message_to_agent` / `wait_for_agents` / `create_agent` / `agent_finish` / `stop_agent`
- Skills 注入专业化（XSS、SQLi 等）；建议每子 Agent 1–3 个 skill

## C. 调度管理

路径：`strix/core/agents.py`、`strix/tools/agents_graph/tools.py`

- 注册 → running；消息入 mailbox 并 `wake.set()`；可选打断当前 stream
- 子 Agent 异步跑，父可并行工作或 `wait_for_agents`（默认 300s，禁止同 turn 重复 wait）
- 终态：completed / stopped / crashed / failed；交互模式可唤醒，非交互终态不可再投递
- 预算：`budget_paused` / `budget_stopped`；策略 `stop` 或 `pause`；可 snapshot 恢复
- Root 用 `finish_scan`，子 Agent 用 `agent_finish` 并向父 inbox 投递完成报告（含已提交漏洞 id）

证据摘录（`AgentCoordinator`）：

```python
Status = Literal["running","waiting","completed","stopped","crashed","failed","budget_paused"]
ACTIVE_STATUSES = frozenset({"running","waiting","budget_paused"})
```

## D. 上下文压缩

路径：`strix/llm/compaction.py`、`strix/llm/context_budget.py`

- Provider 无关会话压缩：超窗或 overflow 时，旧轮摘要为 `<conversation-checkpoint>`，近端原文保留
- 安全向结构化摘要模板（Objective / Findings / Credentials / Work State / Dead Ends…）——**禁止合并去重发现**
- 切分点保证 tool_call / tool_result 成对；`is_context_overflow` 识别 LiteLLM/OpenRouter 400
- Token 预算：LiteLLM model info + fallback；`keep_tokens` / `compact_buffer_tokens` / `summary_max_tokens` 可配

摘录：

```python
async def maybe_compact(session, *, model, instructions="", tools_text="", force=False) -> bool:
    # ... split = _select_split(...); summary = await _summarize(...); replace_session_items(...)
```

## E. 门限审核

- **授权声明**：README WARNING — 仅授权目标；`scope.jinja` 注入范围提示
- **预算门**：spend 达限暂停/停止；测试见 `tests/test_budget_pause_policy.py`
- **完成门**：`finish_coverage_gate`（覆盖记录不完整则阻完成）
- **工具权限**：Docker 沙箱；MCP `allowed_tools` 白名单；交互 vs 非交互投递策略不同
- 无 ARTEX/Aurora 级「每条 bash 人工审批」默认链；偏「授权范围 + 沙箱 + 预算」

## F. 可借鉴 / 不适合

**可借鉴**

1. 成熟的多 Agent 协调器（mailbox + wait + cascade stop + snapshot）  
2. 安全域专用 compaction 模板（防丢漏洞/凭据）  
3. 预算 pause/resume 与 CI 非交互语义分离  
4. Skill 化专家拆分与父历史 Inheritance 可控  

**不适合**

1. 作为「自建调度平台」底座过重且产品闭环强（Cloud 商业化）  
2. 门禁偏运行时沙箱，缺少企业级审批工作流与不可逆分级  
3. Stars 极高但议题多（开源社区噪音大），二次定制边界不清  

## 一句话结论

**Strix 是多 Agent 协作与上下文压缩的标杆实现；适合抄协调器与安全摘要协议，不适合整体替换为内部平台内核。**
