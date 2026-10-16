# VulnHunter（capitalone/VulnHunter）竞品研究

> 仓库：https://github.com/capitalone/VulnHunter · Python（技能为主）· Apache-2.0 · Stars ≈1.1k  
> Capital One 开源；依赖 Claude Code / Opus 级推理。

## A. 定位与技术栈

| 项 | 内容 |
|---|---|
| 定位 | **源码侧**攻击者优先分析：入口→ sink 前向推理 + 证伪引擎；非黑盒动态渗透平台 |
| 形态 | Claude Code **prompt-only skills** + 可选 headless Agent + batch harness |
| 技能三件套 | `/vulnhunt`（Hunt）→ `/vulnhunter-fix`（Fix）→ `/vulnhunt-fix-verify`（Verify） |
| 运行时 | `vulnhunter-agent/`：Claude Agent SDK 无头跑 skill；TOML + 环境变量配置 |
| 存储产物 | `*_VULNHUNT_RESULTS_*` 目录；可选推送到报告仓 / 开 GitHub Issue |

## B. Agent 拓扑

**Dispatcher + 阶段子 Agent**（非常驻多 Agent 服务）：

| 阶段 | 职责 |
|---|---|
| Phase1 Recon | 输入清单、分区、可达性 |
| Phase2 Hunt | 并行 class agents（injection / navigation / logic-crypto）+ sink 审计 |
| Phase2b Verify | **对抗式证伪**，淘汰约半数弱候选 |
| Phase3 Reproduce | PoC / 测试 / 修复策略（不改目标仓） |
| Phase3d Sweep | 根因模式全库扫 |
| Phase4 Report | 编排器汇总 |

编排器 `SKILL.md` **故意不做安全分析**，避免发现进编排上下文导致方法论漂移。

## C. 调度管理

- 交互：Claude Code 会话内按阶段 `Agent` 工具派生子会话，校验产物文件存在再进入下一阶段
- 无头：`python -m agent <repo-url>` → clone → 跑 `/vulnhunt` → manifest → 可选 publish/issues
- Batch：`harness/local_harness/batch` 多仓扫描；`--resume` 跳过已完成
- Verify 模式：只读工具信封，输出 per-finding 判决 schema
- 失败重试：Agent SDK 对 429 等瞬态错误重试；auth 统一 `build_claude_settings`

## D. 上下文压缩

- **方法论级上下文隔离**：分阶段子 Agent + 共享只读参考（`phase2_shared.md` 缓存），编排器不持有 findings 全文
- Fix 技能侧有 `graph/`（代码属性图 build/query）把结构外置，减轻 prompt
- 无类似 Strix/pi 的通用 transcript compaction 模块；依赖 Claude Code 自身上下文与阶段切分
- Agent 配置含 autocompact threshold（对接 Claude Code 自动压缩）

## E. 门限审核

- 证伪引擎 = **质量门**：未通过 adversarial disprove / capability filter 不进入报告
- Verify Agent：**只读**（Read/Write/Edit/Glob/Grep/Agent，无 Bash、无网络）— 独立验证闭环
- Fix 技能：`run-gates.py` / `check-scope.py` 等脚本门；hand-wave guard 防空话修复
- Anthropic Cyber Verification / 双用途免责声明；扫描默认不改目标代码
- Sandbox 段可限制 CLI 文件系统/网络；`Bash` 默认剥离除非 `--enable-bash`

## F. 可借鉴 / 不适合

**可借鉴**

1. Hunt→Fix→Verify **三人分权**，防自卖自夸  
2. 编排器不做分析、阶段产物落盘 — 上下文与质量双赢  
3. Falsification 作为一等公民门禁  
4. Headless Agent + manifest schema 作为 CI 合约  

**不适合**

1. 强绑定 Claude Code/Opus，难以多模型中立调度  
2. 面向白盒源码，不覆盖黑盒/资产发现调度  
3. 「调度平台」能力弱，本质是技能流水线  

## 一句话结论

**VulnHunter 示范「阶段化子 Agent + 证伪门 + 独立复核」的质量体系；适合纳入扫描质量子系统，而非运行时调度内核。**
