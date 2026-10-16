# DeepAudit（lintsinghua/DeepAudit）竞品研究

> 仓库：https://github.com/lintsinghua/DeepAudit · Python + TS 前端 · AGPL-3.0 · Stars ≈7.2k  
> 默认调研分支：`v3.0.0`。定位：人人可用的 AI 代码安全审计（SAST 向）+ 沙箱验证。

## A. 定位与技术栈

| 项 | 内容 |
|---|---|
| 定位 | 动态多 Agent 代码审计：侦察→分析→验证；支持 Ollama 私有化 |
| 后端 | Python 3.11+、FastAPI、LangChain/LiteLLM |
| 前端 | React 18、TypeScript、Ant Design；SSE 实时 |
| 存储 | PostgreSQL（Supabase）；Agent checkpoint 表 |
| 工具 | smart_scan / pattern_match / dataflow / Semgrep / Bandit / Gitleaks 等 |

官方文档：`docs/AGENT_AUDIT_ARCHITECTURE.md`、`docs/ARCHITECTURE.md`。

## B. Agent 拓扑

层级树（Orchestrator 动态派发）：

```
OrchestratorAgent
 ├─ ReconAgent
 ├─ AnalysisAgent
 └─ VerificationAgent
```

- 统一 `BaseAgent`：ReAct 循环（Thought / Action / Observation）
- `TaskHandoff` 结构化交接（已完成工作、发现、建议、优先区域）
- `AgentGraphController` + registry + message_bus：停 Agent、用户指令、树视图、汇总 findings

## C. 调度管理

路径：`agents/orchestrator.py`、`core/graph_controller.py`、`api/.../agent_tasks.py`

- 创建任务入队后台 → 初始化 Orchestrator → ReAct 最多约 20 轮  
- `dispatch_agent` 派子 Agent；子 Agent 各自迭代上限（recon15 / analysis30 / verification15）  
- 熔断器、指数退避重试、令牌桶限流（LLM/外部工具）  
- Checkpoint：按迭代间隔落库，支持恢复查询  
- 取消：`request_stop` / `cancel`；图控制器可 stop_all  

证据（架构文档）：Orchestrator 决策 `dispatch_agent` | `summarize` | `finish`。

## D. 上下文压缩

双通道：

1. **BaseAgent.compress_messages_if_needed** → `MemoryCompressor`（`backend/app/services/llm/memory_compressor.py`）  
   - 阈值：总 token > 100k × 0.9  
   - 保留全部 system + 最近 15 条；更早 chunk 抽关键信息（发现类型/工具/决策/错误）成 `<context_summary>`  
2. **配置** `max_context_messages` 等资源上限  
3. Checkpoint 外置长程状态，减轻单会话负担  

摘录：

```python
MAX_TOTAL_TOKENS = 100_000
MIN_RECENT_MESSAGES = 15
COMPRESSION_THRESHOLD = 0.9
```

## E. 门限审核

偏**健壮性与资源门**，非渗透 bash 审批：

- 输入路径校验、文件大小上限、exclude_patterns / target_files 范围限定  
- 熔断 / 限流 / 最大 findings 数  
- VerificationAgent 降误报；沙箱 PoC 验证级别可配  
- 前端可发用户指令到指定 Agent；停止 Agent  

缺少 Aurora 式作用域强制与不可逆分级（因主场景是读代码审计）。

## F. 可借鉴 / 不适合

**可借鉴**

1. Orchestrator + 专科子 Agent + TaskHandoff 协议清晰  
2. MemoryCompressor 实现简单可移植  
3. 熔断/限流/checkpoint 工程化齐全，适合「审计类」长任务  
4. SSE 事件模型（thinking_token / tool / finding / dispatch）产品体验好  

**不适合**

1. AGPL-3.0  
2. 主战场是白盒审计，不是资产/黑盒战役调度  
3. 门禁深度不足以支撑进攻性工具链  

## 一句话结论

**DeepAudit 是「ReAct 多 Agent 审计产品」的完整样板；调度与压缩可参考，进攻性平台门禁与双图能力需另选蓝本。**
