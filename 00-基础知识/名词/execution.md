# Execution（执行）

> 图位：思维链总览第 6 层 · 完整架构中 Function 调用 / 执行结果 · [总览图](../assets/agent-chain-of-thought-overview.jpg) · [架构图](../assets/agent-complete-architecture.jpg)  
> 相关：[tool.md](tool.md) · [agent-loop.md](agent-loop.md) · [check-and-correction.md](check-and-correction.md)

---

## 结论先行

**执行**是把「已选工具 + 已填参数」变成**真实世界效果**的一步：发起调用、拿到返回、产生中间结果或副作用（写文件、发消息、改库等）。

Thinking / Action 仍在模型侧；**执行跨出模型，进入运行时与外部系统**——因此也是门禁与审计的主战场。

---

## 是什么

图1将「执行」放在工具调用与记忆之后、检查与修正之前：调用工具或采取行动，生成中间结果。

图2对应「Function 调用 / 执行结果」箭头：从核心循环连向工具，再把结果送回 Reflection。

执行层通常负责：

- 鉴权与凭证注入（不交给模型明文玩）  
- 超时、重试、幂等  
- 沙箱 / 网络策略  
- 结构化日志（谁、何时、调了什么、结果摘要）  

---

## 应用场景

| 场景 | 执行面关注点 |
|---|---|
| 发 Slack | 频道权限、消息审计、失败重试 |
| 查库 | 只读账号、行级权限、慢查询超时 |
| 跑代码 | 容器资源限额、出网策略 |

---

## 效益

- 把「模型意图」变成「可观测事实」  
- 为 Reflection 提供硬证据（而非再猜）  
- 使安全策略可落在调用路径上  

---

## 与本仓挂钩

Full-Chain-Sentinel 等表明：有 harness 执行壳还不够，**安全边界要产品层补强**。Aurora / VulnHunter 的 sandbox、scope_guard 都作用于执行面前后。
