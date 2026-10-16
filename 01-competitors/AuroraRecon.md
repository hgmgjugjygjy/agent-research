# AuroraRecon（blkheadfish/AuroraRecon）竞品研究

> 仓库：https://github.com/blkheadfish/AuroraRecon · Python · Stars ≈3（2026-10-16）  
> 定位：基于大模型的自动化渗透测试系统；Kill-Chain + 世界模型（攻击图）。

## A. 定位与技术栈

| 项 | 内容 |
|---|---|
| 定位 | 侦察→利用→后渗→报告闭环；竞赛/授权演练向 |
| Agent 框架 | **LangGraph 0.2 + LangChain 0.3** |
| API | FastAPI + WebSocket（Redis Stream v2） |
| 存储 | PostgreSQL 16、Redis 7、MinIO |
| 前端 | Vue 3 + Pinia + Element Plus + ECharts 攻击图 |
| 运行 | Docker Compose；Kali toolbox 容器；可选 Metasploit RPC |

## B. Agent 拓扑

编排器 `backend/agents/orchestrator.py` 支持三种图模式（`ATTACK_CHAIN_MODE`）：

| 模式 | 形态 |
|---|---|
| linear | 固定 DAG：recon → … → exploit_decision → human_approval → foothold → … → report |
| feedback | 在 secondary/post_foothold/privesc 出口允许回流重规划 |
| supervisor | 星形：全阶段回 `supervisor` 再路由（约 31 条确定性规则 + LLM 兜底） |

阶段节点含 recon / vuln_scan / surface_enum / 域内 AD 节点 / cloud / exploit / persistence 等。

世界模型 `AttackGraph`：host/service/web_endpoint/finding/credential/session/loot/objective + 类型边。

## C. 调度管理

- `Orchestrator` 懒加载编译图；checkpointer：**AsyncPostgresSaver**，失败回退 `MemorySaver`
- `@retry_node`：工具失败最多重试 3 次
- `interrupt_before=["human_approval","post_foothold_approval"]` — LangGraph 原生中断
- 任务 API：create / cancel / abort / approve / checkpoint/respond / chat / resume
- Skill 引擎：确定性步骤优先，失败再 LLM ReAct；草案需人工 promote

证据：

```python
def build_graph(checkpointer=None):
    mode = _current_chain_mode()
    if mode == "supervisor": return _build_graph_supervisor(checkpointer)
    if mode == "feedback": return _build_graph_feedback(checkpointer)
    return _build_graph_linear(checkpointer)
```

## D. 上下文压缩

- **短时**：LangGraph checkpointer 持久化整图状态（断点续跑），非 transcript 滑动窗口摘要
- **跨任务记忆**：`engagement_memory.py` 从历史 task 抽 PriorIntel（服务/指纹/发现/凭据**存在性**提示，无明文），注入 `attack_graph`（`source=prior`）与 `runtime_facts`
- Skill / 世界模型查询把大上下文外置到图与 YAML，LLM 只看前沿与决策卡

摘录（记忆注入约定）：

```python
# PriorIntel: 凭据只存存在性提示 {service, username, has_secret:true}，不含明文
state.runtime_facts["prior_intel"] = prior.to_dict()
```

## E. 门限审核（产品亮点）

多层门，且尽量**确定性、LLM 不可覆盖**：

1. **scope_guard.py**：命令执行前解析目标 IP/域名，校验 `authorized_scope`
2. **safety_gate.py**：YAML 规则；黑名单（云 metadata、政府段、过大 CIDR、关键词）→ BLOCK；灰名单 WARNING；authorization_token 仍强制黑名单
3. **irreversible_gate.py**：autonomous 模式下可逆自动放行，不可逆（persistence / lateral / 用户创建 / 写系统…）仍 checkpoint
4. 三级自治：手动 / 监督 / 全自动（`autonomy_level`）
5. 急停 `POST /tasks/{id}/abort` + 容器清理；速率限制（LLM/工具/MSF）

摘录：

```python
# safety_gate: 所有规则来自 safety_rules.yaml，LLM 无法覆盖
# irreversible_gate: 不可逆分类宁可保守（多卡人）
```

## F. 可借鉴 / 不适合

**可借鉴**

1. LangGraph 三种拓扑可配置切换 — 产品上可做「编排档位」  
2. 作用域 + 安全 YAML + 不可逆分级 + 自治等级 — **门禁分层范本**  
3. 跨任务 prior 注入（脱敏）— 持续运营记忆  
4. interrupt + Postgres checkpointer — 企业 HITL 标配  

**不适合**

1. Stars/社区活跃度低，维护风险高  
2. 图与进攻阶段耦合深，难抽成通用调度内核  
3. License 未在元数据标明（README 写 MIT），合规需再核  

## 一句话结论

**AuroraRecon 是「LangGraph + 世界模型 + 分层门禁」最贴合自建渗透调度平台的蓝本，尤其门禁与自治等级设计应优先吸收。**
