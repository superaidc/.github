# superaidc 仓库地图

联系人：Zhu Lei · woicyou@gmail.com · 组织：[`superaidc`](https://github.com/superaidc)

## 组织内仓库

| 仓库 | URL | 可见性 | 状态 | 用途 |
|---|---|---|---|---|
| aidc-lab | https://github.com/superaidc/aidc-lab | private | 活跃 · 已从 `kabishou11/aidc-lab` 转入 | 六层路由引擎验证实验室（产品/lab） |
| aidc-rsi-poc | https://github.com/superaidc/aidc-rsi-poc | private | 活跃 · PoC | RSI 外环旁路；G0–G6；权威 loop_once md5 `85c298146411229238b5e90a1e60ac17`；默认 live-mem0 未授权 |
| session-memory-poc | https://github.com/superaidc/session-memory-poc | private | 活跃 · PoC | 会话记忆/用户画像旁路（claude-mem + Mem0 → `:4100`） |
| .github | https://github.com/superaidc/.github | public | 活跃 · 文档 | 组织主页 `profile/README.md`、本手册与地图 |

## 相关但不迁入（个人 / fork）

| 仓库 | 说明 |
|---|---|
| [`kabishou11/hermes-overlay`](https://github.com/kabishou11/hermes-overlay) | 个人 Hermes 相关；**不**转入 superaidc |
| litellm / opencode 等 upstream fork | 上游或个人 fork，保持原处；AIDC 只经 `:4100` 使用 LiteLLM，不把路由逻辑沉进去 |

## 本地路径对照

| 仓 | Mac mini 路径 |
|---|---|
| aidc-lab | `~/opencode/work/aidc-lab` |
| aidc-rsi-poc | `~/opencode/work/aidc-rsi-poc` |
| session-memory-poc | `~/opencode/work/session-memory-poc` |

## 推送时默认排除（各 PoC .gitignore 共识）

- `.env`、`.venv/`、`venv/`、`__pycache__/`、`*.pyc`、`.DS_Store`
- `logs/`、`runs/`、大型 `*.jsonl`、`workspace/archive/`
- 本地 DB / vector 存储（如 `data/`、`workspace/mem0/`）
- `node_modules/`（若出现）

## 变更记录（迁移）

| 日期 (Asia/Shanghai) | 事项 |
|---|---|
| 2026-11-05 | 创建组织 `superaidc`；转入 `aidc-lab`；新建并推入 `aidc-rsi-poc`、`session-memory-poc`；发布本 `.github` 文档 |
