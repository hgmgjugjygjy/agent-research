# VulnHunter · 代码模块拓扑（代码视角）

> 依据：`01-competitors/vulnhunter.md` + `_trees/VulnHunter.json` + 上游树；仅模块/调用关系。不含利用步骤。

```mermaid
flowchart TB
  subgraph entry["入口"]
    CLI["python -m agent repo-url\nvulnhunter-agent/agent/__main__.py"]
    CC["Claude Code 交互会话"]
    BATCH["harness/local_harness/batch/run.py"]
  end

  subgraph skills["prompt-only skills"]
    HUNT["vulnhunt/SKILL.md 编排器\n故意不做安全分析"]
    FIX["vulnhunter-fix/SKILL.md"]
    VER["vulnhunt-fix-verify/SKILL.md"]
  end

  subgraph hunt_phases["vulnhunt/phases/"]
    P1["phase1_recon.md"]
    P2["phase2_hunt.md\n+ phase2_class_inj/nav/log.md\n+ phase2_shared.md"]
    P2B["phase2b_verify.md\nAdversarial Disprove = Falsify"]
    P3["phase3_reproduce_test.md\nphase3c_fixes.md\nphase3d_sweep.md"]
    P4["phase4_report.md"]
  end

  subgraph fixpkg["vulnhunter-fix/"]
    FG["vulnhunter_fix/graph/\nbuild.py query.py schema.py"]
    GS["scripts/run-gates.py\nscripts/check-scope.py"]
    HW["tests/test_hand_wave_guard.py"]
    FP["prompts/implement.md plan.md verify.md"]
  end

  subgraph verpkg["vulnhunt-fix-verify/phases/"]
    V0["phase0_preflight.md"]
    V1["phase1_extract.md"]
    V2["phase2_verify.md"]
    V4["phase4_emit.md"]
  end

  subgraph agentrt["vulnhunter-agent/"]
    RUN["agent/runner.py"]
    BS["agent/build_settings.py\n_build_sandbox + 工具信封"]
    MAN["agent/manifest.py\nscan_manifest.schema.json"]
    VR["agent/verify_runner.py\n只读工具信封"]
    VFY["agent/verify.py + verify_*.py"]
    DISP["verify_disposition.schema.json"]
    AUD["agent/audit.py"]
    CFG["agent/config.py TOML"]
  end

  subgraph artifacts["落盘产物"]
    RES["*_VULNHUNT_RESULTS_*"]
    VERD["per-finding verdict\nverify_disposition.json"]
  end

  CLI --> RUN
  BATCH --> RUN
  CFG --> BS --> RUN
  RUN --> CC
  CC --> HUNT & FIX & VER
  HUNT --> P1 --> P2 --> P2B --> P3 --> P4
  P4 --> RES
  FIX --> FG & GS & HW & FP
  FIX --> RES
  VER --> V0 --> V1 --> V2 --> V4
  RUN --> MAN
  VR --> VFY --> DISP
  VR --> VER
  V4 --> VERD
  AUD --> RUN
  HUNT -.->|"编排器不持 findings 全文"| P2
```

## 说明

1. 三 skill 分权：`/vulnhunt` 找洞、`/vulnhunter-fix` 修、`/vulnhunt-fix-verify` 独立复核。
2. **Falsify** = `phases/phase2b_verify.md`（对抗证伪），再经 Capability Filter 后才进报告。
3. Headless：`runner.py` + Claude Agent SDK；`build_settings._build_sandbox` 限制 FS/网络（默认可关）。
4. Verify：`verify_runner.py` **剥离 Bash/Network**，只留 Read/Write/Edit/Glob/Grep/Agent。
5. CI 契约：`scan_manifest.schema.json` + `verify_disposition.schema.json`。
6. Fix 侧脚本门 `run-gates.py` / `check-scope.py` + hand-wave guard；结构外置到 `vulnhunter_fix/graph/`。
