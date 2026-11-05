# superaidc 运维手册

面向在 Mac mini / 本机上开发 AIDC 的日常操作。组织主页见 [`profile/README.md`](profile/README.md)。

## 1. 克隆与作者身份

```bash
# 产品仓
git clone https://github.com/superaidc/aidc-lab.git ~/opencode/work/aidc-lab

# 旁路 PoC
git clone https://github.com/superaidc/aidc-rsi-poc.git ~/opencode/work/aidc-rsi-poc
git clone https://github.com/superaidc/session-memory-poc.git ~/opencode/work/session-memory-poc
```

**每个仓**设置（不要依赖错误的 global）：

```bash
git config user.name 'Zhu Lei'
git config user.email 'woicyou@gmail.com'
```

规则：

- Commit / PR 作者**只能**是 Zhu Lei `<woicyou@gmail.com>`。
- **禁止** `Co-authored-by:`、禁止在 commit/PR 正文写 AI / 模型厂商署名。
- 分支命名偏好：`fix/<topic>`（与既有 aidc-lab 习惯一致）。

## 2. 端口与进程边界

| 端口 | 服务 | 说明 |
|---|---|---|
| `:4000` | aidc-engine | 上层路由；实验室/产品路径 |
| `:4100` | LiteLLM | **仅**模型网关；PoC 调模型只打这里 |
| （旁路） | Mem0 / claude-mem / RSI `loop_once` | 独立进程与数据目录，不进 k3d 产品清单除非另有决策 |

高阶 k3d 提示（细节以 aidc-lab 文档为准）：

- 集群名常见为 `k3d-aidc`；LiteLLM 经 port-forward 暴露到宿主机 `:4100`。
- `session-memory-poc` 提供 `bin/pf-litellm-4100.sh` 做可逆的 host 侧转发。
- **不要**为了省事把 AIDC Lane / 归档逻辑配进 LiteLLM config。

## 3. 环境变量

- 真实值只放各仓本地 `.env`（已被 gitignore）。
- 提交 `.env.example` 时**只写键名**与安全默认 URL。
- RSI live 跑法可 `set -a; . ~/opencode/work/session-memory-poc/.env; set +a` 再调 `loop_once.py --live`（密钥仍不落 RSI 仓文件）。

## 4. 什么不要放进 aidc-lab（产品仓）

- PoC 脏数据：`logs/`、大型 `*.jsonl`、本地 chroma/qdrant/sqlite。
- 旁路脚本的「临时最佳结果」、未评审的 RSI 候选政策。
- 任何密钥、token、个人 API 账单凭证。
- 对 Hermes memory 目录的写入脚本。

旁路实验 → 独立 `superaidc/aidc-*-poc` 私有仓；需要结论时用文档/PR 描述回馈产品仓，而不是把 PoC 目录嵌进去。

## 5. PR 与合并习惯

- 功能分支：`fix/...`；说明写清动机、验证步骤、对 `:4000`/`:4100` 边界的影响。
- 合并前确认：作者邮箱正确、无密钥、CI / 本地回归通过（以仓内 VERIFY/STATUS 为准）。
- RSI 权威文件 `scripts/loop_once.py` 的 md5 变更必须在 STATUS/VERIFY 专章记录；当前冻结值见组织主页。

## 6. 迁移后检查清单

```bash
cd ~/opencode/work/aidc-lab && git remote -v && git fetch origin
cd ~/opencode/work/aidc-rsi-poc && git remote -v && git status -sb
cd ~/opencode/work/session-memory-poc && git remote -v && git status -sb
```

旧书签 / CI / 子模块若仍指向 `kabishou11/aidc-lab`，改为 `superaidc/aidc-lab`。

## 7. 事故与红线速查

1. 误把密钥推进去 → 立刻轮换密钥，并视为已泄露（history 改写另议）。
2. 误打 `:4000` 做记忆/RSI → 停脚本，改回 `:4100`，写进该仓 REDLINES。
3. 误改 Hermes memory → 停止操作，按 Hermes 自身恢复流程，不在本组织仓内「修目录」。
