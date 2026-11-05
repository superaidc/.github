# superaidc

**AIDC（AI Data Center / 六层路由引擎）验证实验室** 与旁路实验组织。

本组织托管产品实验仓与解耦的旁路 PoC。目标是在**不污染产品主路径**的前提下，验证路由、成本/质量证据、会话记忆与 RSI（Recursive Self-Improvement）外环能力。

联系人：Zhu Lei · `woicyou@gmail.com` · GitHub [`kabishou11`](https://github.com/kabishou11)

---

## 我们在做什么

- **aidc-lab**：六层路由引擎验证实验室——双协议接入、Lane/归档、成本 + 质量证据闭环。
- **旁路 PoC**：会话记忆/用户画像、RSI 外环门控等，**独立私有仓**，默认不进产品树。
- **硬边界**：上层路由与模型网关解耦；Brain / RSI 外环永远在引擎与 LiteLLM 之外。

---

## 架构红线（HARD）

| 组件 | 端口 / 角色 | 允许 | 禁止 |
|---|---|---|---|
| **aidc-engine** | `:4000` 上层路由 / Lane / archive | 产品路由、证据链、实验室验证 | 把路由逻辑下沉进 LiteLLM |
| **LiteLLM** | `:4100` **仅**模型网关 | OpenAI-compatible 转发、密钥与上游模型 | 承接 AIDC 路由 / Lane / RSI / Brain |
| **Brain / RSI 外环** | 旁路进程 / 独立仓 | 读质量报告、经 `:4100` 调模型、写 PoC 工作区 | 进入 aidc-engine 主路径；改 LiteLLM 路由表 |
| **Hermes memory 目录** | 本机 Hermes 落盘 | 只读观察（若业务需要） | 从本组织任一仓的运维脚本**编辑/清空** |

一句话：**aidc-engine 与 LiteLLM 保持解耦；RSI / 会话画像走旁路；Hermes 记忆目录禁止从这些仓的运维去改。**

---

## 仓库地图

| 仓库 | 可见性 | 一句话 |
|---|---|---|
| [`aidc-lab`](https://github.com/superaidc/aidc-lab) | private | 产品 / 实验室：六层路由引擎验证 |
| [`aidc-rsi-poc`](https://github.com/superaidc/aidc-rsi-poc) | private | RSI bypass PoC：G0–G6 门控、Mem0 隔离；权威 `loop_once.py` md5 **`85c298146411229238b5e90a1e60ac17`**；默认 live-mem0 **未授权** |
| [`session-memory-poc`](https://github.com/superaidc/session-memory-poc) | private | 会话记忆 / 用户画像旁路：claude-mem + Mem0，只接 LiteLLM `:4100` |
| [`.github`](https://github.com/superaidc/.github) | public | 本组织主页与运维手册（你正在看的文档） |

更细的表格与「相关但不迁入」说明见 [`REPO-MAP.md`](../REPO-MAP.md)。日常操作见 [`ORG.md`](../ORG.md)。

---

## 如何新增实验仓

1. **一律建在 `superaidc` 下**，不要建在个人账号后再「长期放着」。
2. 命名：`aidc-<topic>-poc`（产品特性可不用 `-poc` 后缀，但实验默认带）。
3. **默认 private**。
4. 首批文件至少包含：`README.md`、`ARCH.md`、`STATUS.md`、`REDLINES.md`，以及 `.gitignore` + `.env.example`（若有密钥）。
5. **红线写进 REDLINES**：不进 aidc-lab 产品树、不碰 Hermes memory、密钥不上库、模型只走 `:4100`（若适用）。
6. 本地克隆路径约定：Mac mini 上 `~/opencode/work/<repo>`。
7. Commit author **只能**是 `Zhu Lei <woicyou@gmail.com>`（仓库 local config；禁止 Co-authored-by / AI 署名进公开提交信息）。

```bash
gh repo create superaidc/aidc-<topic>-poc --private --description '…'
cd ~/opencode/work/aidc-<topic>-poc
git init -b main
git config user.name 'Zhu Lei'
git config user.email 'woicyou@gmail.com'
# 写好 .gitignore 后再 git add
git remote add origin https://github.com/superaidc/aidc-<topic>-poc.git
git push -u origin main
```

---

## 本机布局（Mac mini）

```text
~/opencode/work/
  aidc-lab/              # 产品仓（已迁至 superaidc/aidc-lab）
  aidc-rsi-poc/          # RSI 旁路
  session-memory-poc/    # 会话记忆旁路
  aidc-lab-*/            # 历史 worktree / 实验分支工作区（共享 aidc-lab .git）
```

aidc-lab 的多个 worktree 共享同一 `.git`，**改一次 `origin` 即可覆盖全部 worktree**。

---

## 远程迁移说明

| 旧地址 | 新地址 |
|---|---|
| `https://github.com/kabishou11/aidc-lab.git` | `https://github.com/superaidc/aidc-lab.git` |

本地更新：

```bash
cd ~/opencode/work/aidc-lab
git remote set-url origin https://github.com/superaidc/aidc-lab.git
git fetch origin
git remote -v
```

个人账号下的 `hermes-overlay`、以及 litellm / opencode 等 **fork / 个人仓不迁入**本组织（见 REPO-MAP）。

---

## 安全

- **禁止**把真实密钥、token、`.env`、私钥提交进任何仓。
- 只提交 `.env.example`（键名 + 安全默认 URL，无值）。
- PoC 默认 **private**；组织主页 `.github` 为 public，**不得**写入内部密钥或未脱敏日志。
- 大体积运行时数据（`logs/`、`runs/`、`workspace/archive/*.jsonl`、本地 vector DB）一律 gitignore。

---

## RSI 现状摘要（2026-11-05）

| 项 | 状态 |
|---|---|
| G0–G6 门控 + fail-closed + G6 幂等 | 已通；权威脚本 md5 `85c298146411229238b5e90a1e60ac17` |
| Mem0 `user_id` 隔离 / `:4100` 探活 | 已通（stub / 受控路径） |
| 默认 live-mem0（方案 #3） | **未授权**，保持关闭 |
| 内部小流量 dogfood | **CONDITIONAL GO** |
| 面向数千用户 | **NO-GO**，直至补齐 live Mem0 + `rsi-key` 限流等缺口 |
| 剩余缺口 | 标定（calibration）、`propose()`、配额、`rsi-key` rate limit 实装 |

详情见 [`aidc-rsi-poc`](https://github.com/superaidc/aidc-rsi-poc) 内 `STATUS.md` / `VERIFY.md` / `VERIFY-REPORT.md` / `REDLINES.md`。

---

## 文档索引

- [ORG.md](../ORG.md) — 克隆、端口、分支/提交约定、什么不该进产品仓
- [REPO-MAP.md](../REPO-MAP.md) — 全仓表 + 相关个人仓说明
