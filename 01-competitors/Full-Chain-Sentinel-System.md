# Full-Chain-Sentinel-System（zgsfsec/Full-Chain-Sentinel-System）竞品研究

> 仓库：https://github.com/zgsfsec/Full-Chain-Sentinel-System · TypeScript · MIT · Stars ≈79  
> **明确声明**：基于 [earendil-works/pi](https://github.com/earendil-works/pi) agent harness 的衍生 fork，加入中文 Web 控制台与渗透作业容器。

## A. 定位与技术栈

| 项 | 内容 |
|---|---|
| 定位 | 授权渗透测试 Agent：Web / 小程序 / APP「全链路」；方法论驱动系统提示 |
| 内核 | pi monorepo 分包：`@earendil-works/*`（agent / ai / coding-agent / durable / tui / chord…） |
| 新增 | `packages/web` 中文控制台；Kali 作业容器；安卓 DroidBridge；漏洞中枢；Nuclei POC 订阅 |
| 运行 | Node ≥22.19；每项目一 Kali 容器 `docker exec` 跑 bash |
| 会话 | pi JSONL 会话树 —— 「会话即审计」 |

## B. Agent 拓扑

继承 pi **极简单 Agent harness**（默认无内建 sub-agent / plan mode）：

- 主循环：prompt → model → tools → compact → resume
- 工程模式：「指挥官」多任务（README 称测试阶段）
- 子代理：通过 pi 扩展/工具可产生，bash 走同一审批链
- Web：流式对话、工具卡、会话树、文件与子代理弹窗

与 ARTEX/Strix 的常驻 planner-worker 不同：**会话树 + 工具审批**是主抽象。

## C. 调度管理

- Web 控制台创建会话 / 项目；模型端点可配（OpenAI 兼容）
- 持续目标 / 工程模式负责任务队列（产品层，非 LangGraph）
- 崩溃恢复：pi durable session + operation state machine（上游 harness 规格）
- 容器硬依赖：Docker 检查失败则拒绝 `prompt` / `compact` / 持续目标

## D. 上下文压缩

完全复用 pi 压缩栈（文档：`packages/coding-agent/docs/compaction.md`）：

| 机制 | 触发 | 作用 |
|---|---|---|
| Auto-compaction | `contextTokens > contextWindow - reserveTokens`（默认 reserve 16k） | 旧消息结构化摘要 + 保留近端 `keepRecentTokens`（默认 20k） |
| Branch summarization | `/tree` 切分支 | 离开分支时注入摘要 |
| Overflow recovery | provider overflow / length | compact-and-retry 一次 |
| Extension hooks | `session_before_compact` 等 | 可自定义摘要模型 |

切点规则：不切在 tool result 中间；支持 split user-message span 双摘要合并。会话格式 JSONL 可审计回放。

## E. 门限审核

三档执行模式（产品核心）：

1. **审批模式（默认）**：每条 bash / `droid_shell` 可编辑后放行或拒绝  
2. **自动执行**：模型可跑任意命令（高风险）  
3. **AI 审批**：守门员小模型判 ALLOW/BLOCK；**失败/超时 fail-open 放行**（README 直言不是沙箱）

边界坦诚（选型必读）：

- 审批链**不覆盖** read/edit/write、文件面板、MCP、frida/抓包/APK 分析等直连容器工具  
- 容器以 root 跑、无资源上限 / seccomp；控制台进程在宿主机  
- 首次进入须勾选授权承诺；随机 gate token 挡扫描器  

## F. 可借鉴 / 不适合

**可借鉴**

1. **直接采用/嵌入 pi harness** 作为编码与会话内核的现实路径（本仓已证明）  
2. 三档执行模式 + 守门员模型 — 产品化 HITL 档位  
3. 会话树审计 + compaction 工业文档完备  
4. Web/小程序/APP 载体抽象（同一后端不同前端工具开关）  

**不适合**

1. 门禁 fail-open 与「容器非整体沙箱」对严肃红队平台不够  
2. 缺双图/世界模型级任务编排；偏「会话 Agent」而非「战役调度」  
3. Fork 上游需持续 merge；包名仍 `@earendil-works/*` 品牌与供应链需治理  

## 一句话结论

**全链哨兵证明「pi 极简 harness + 渗透领域壳」可行；适合作为会话/压缩/审批档位参考，战役级调度仍需自建或借鉴 ARTEX/Aurora。**
