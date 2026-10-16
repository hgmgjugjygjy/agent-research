# Evaluation / Trace（评测与追踪）

> 相关：[reflection.md](reflection.md) · [orchestration.md](orchestration.md) · [guardrails-hitl.md](guardrails-hitl.md)

---

## 定义

- **Trace（追踪）**：一次任务中模型调用、tool 调用、状态转移、审批事件的**结构化时间线**，用于调试与审计。  
- **Evaluation（评测）**：用数据集、规则、LLM-as-judge 或人工，对 Agent **成功率、安全性、成本、延迟**打分，支撑回归。

---

## 解决什么

- 「偶发很好、上线很飘」无可观测就无法改  
- 门禁与压缩变更需要回归  
- 事故复盘需要证据链

---

## 怎么用

1. 每轮 Loop / 每图节点打点（id、父子、token、延迟、成败）  
2. 保留可回放输入（脱敏）；大工件存对象存储  
3. 建黄金任务集：改 prompt/模型先跑评测  
4. 安全集：越权、外发、范围突破用例必测

---

## 常见坑

- 只有日志文件无线索 id  
- 评测只测「会不会说话」不测工具副作用  
- Trace 含密钥明文

---

## 本仓关联

YoungFlow/VH 的 gate/decision 文件路由、双向论证，偏质量评测闭环；平台观测应覆盖编排与会话两层。
