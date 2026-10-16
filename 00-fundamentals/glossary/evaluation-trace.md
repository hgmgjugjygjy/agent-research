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

## 技术实现原理

**Trace 数据结构**：一次运行 = 一棵 span 树 `Span{trace_id, span_id, parent_id, type(llm|tool|retrieval|agent), input, output, tokens, latency, error}`，常用 OpenTelemetry 语义约定上报。

**评测流程**：
1. 建数据集：`{input, expected / rubric}`，来自真实 trace 抽样 + 人工标注。
2. 跑 Agent 得到输出与 trace。
3. 评分器：确定性（完全匹配、Schema、工具调用序列是否正确）+ LLM-as-judge（按 rubric 打分）+ 人工抽检。
4. 指标：任务成功率、步数、成本、延迟、工具错误率；回归对比基线后才能发布。

```python
for case in dataset:
    out, trace = agent.run(case.input, tracer=otel)
    scores.append({"exact": out == case.expected, "judge": judge(case.rubric, out), "steps": len(trace.spans)})
```

**为什么**：Agent 是多步随机过程，只看最终答案无法定位问题；trace 让每一步可回放，评测集让改 prompt/换模型有客观依据。

## 本仓关联

YoungFlow/VH 的 gate/decision 文件路由、双向论证，偏质量评测闭环；平台观测应覆盖编排与会话两层。
