# Agent 挑战赛 · 项目管理仓库

本仓库是参加两项比赛的项目管理仓库，使用 [OpenSpec](https://github.com/Fission-AI/OpenSpec) 做规格驱动的变更管理。

## 比赛

> 两道任务已从《DP Arena 比赛参赛指南》给出的 Playground challenge ID 实际下载；要点见下表与 `openspec/changes/`。题包未给出正式截止日期。

| 比赛 | 链接 | 备注 |
|---|---|---|
| 比赛 A（DPA部署赛） | https://dptechnology.feishu.cn/wiki/NevpwcEEGi2deek51TMcNTvDnAh | **DPA4C Nano 单 PPU 生产部署优化赛**（challenge `optimize-the-complete-dpa4c-nano-lammps-md-step-ac-6a1e56aa`）：单张 PPU810E 96GB，CuNi/Si/MgO 共九个 MD 场景；全部正确后按九题加速比几何平均排名。当前下载包缺题面要求的 `instruction.md`，资源说明存在 v4/v5 冲突，且冻结环境为 `release_ready: false`；未澄清前不得启动收费沙箱。比赛页面 https://play.bohrium.com/competitions/dpa |
| 比赛 B（LLM 部署赛） | https://dptechnology.feishu.cn/wiki/NevpwcEEGi2deek51TMcNTvDnAh | **推理服务评测赛**（challenge `llm-challenge-arena-v1`）：固定 GLM-5.3-Flash 与 8×A100-SXM4-80GB；能力门通过后，先比 `N@SLO`，同档再比 `TPOT`。交付 `/app/submission/submission.json` 指向授权镜像及启动命令，并实现 `/chat/completions`、`/generate`、`/flush_cache`；评分元数据与详细题面存在冲突，待主办方确认。比赛页面 https://play.bohrium.com/llm-arena/competitions/llm-deploy-arena-v1 |

## 仓库边界

本仓库是 **store 仓库**，只负责对齐需求与记录事实：比赛题目解读、方案规格、分工与进度。比赛代码**不放在这里**。

- 每个比赛项目另开独立代码仓库（多个项目则一题一仓），代码仓库只放实现
- 代码仓库需要读规格时，clone 本仓库并注册为 store：`openspec store register <path> --id agent-challenge --yes`
- 各项目代码仓库统一登记在下表，避免实现散落后找不到归属：

| 项目 | 代码仓库 | 备注 |
|---|---|---|
| 比赛 A | 待创建 | |
| 比赛 B | 待创建 | |

## 工具链约定

| 用途 | 工具 |
|---|---|
| 规格 / 变更管理 | `openspec`（本仓库） |
| Bohrium 资源（文件 / 数据集 / 任务 / 节点等） | `bohr` CLI |
| 数据集下载 | `wenyon`（`wenyon-cli`） |
| 算力资源 / Arena Team | `trisol`（当前账号的 `arena` 申请已 approved；执行任务前仍实时检查 team 与 quota） |

## OpenSpec 安装（新成员必读）

要求 **Node.js ≥ 20.19.0**（`node --version` 确认）。

```bash
# 1. 安装 CLI
npm install -g @fission-ai/openspec@latest

# 2. 验证
openspec --version   # 应输出 1.13.0 或更高

# 3. 克隆本仓库后，注册为本机 store
git clone git@github.com:Grenzlinie/agent-challenge.git
cd agent-challenge
openspec store register . --id agent-challenge --yes

# 4. 检查（输出 Issues: none 即成功）
openspec store doctor agent-challenge
```

常见问题：

- `openspec: command not found` → npm 全局 bin 不在 PATH：运行 `npm prefix -g`，把输出路径下的 `bin` 子目录加入 shell 的 PATH
- **不要**在本仓库运行 `openspec init`——`openspec/` 目录已初始化并配置好团队约定

## OpenSpec 使用

- 规格与变更：`openspec/`（`specs/` 为已定稿规格，`changes/` 为进行中的变更）
- AI 工具命令已配置：Kimi Code / Claude Code / Cursor / 共享 `.agents`
- 发起新变更：`/opsx:propose "想法"`（Claude）或对应工具的等价命令
- 详细约定见 `openspec/config.yaml`

## 团队协作

完整协作流程（接入步骤、propose → apply → archive 闭环、各 AI 工具命令对照、FAQ）见飞书文档：
[OpenSpec 协作指南 · Agent 挑战赛](https://dptechnology.feishu.cn/docx/PC2jds3Fjoig7NxH3vfcv9sdnhf)

本仓库即团队的单一事实来源（single source of truth），并已注册为 OpenSpec store（ID：`agent-challenge`）：成员 `git clone` 并 `openspec store register` 后，`openspec/` 目录对所有人和 AI 编码助手可见。跨仓库共享规格可进一步使用 OpenSpec Stores（beta），见 OpenSpec 文档。
