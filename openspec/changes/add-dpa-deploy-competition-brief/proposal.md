# Proposal

## Why

现有比赛 A 摘要与实际下载题包不一致：正式任务是九个完整 MD 场景，而非固定 1024 原子的单次推理；当前题包还缺少题面规定的 `instruction.md`，存在资源版本 v4/v5 冲突，且冻结环境标记为 `release_ready: false`。团队需要以可复核的题包内容校正规格，并在阻塞解除前禁止启动收费沙箱。

## What Changes

- 以 Playground challenge `optimize-the-complete-dpa4c-nano-lammps-md-step-ac-6a1e56aa` 的实际 `task.md`、配置、schema 和资源清单重写任务目标、输入、正确性门槛、评分与交付物。
- 增加题包完整性硬门禁：必须定位同时包含 `instruction.md` 与 `asset/tools/arena.py` 的唯一根目录，否则停止并联系组织方。
- 记录下载元数据指向数据集 v5、而 `download-instructions.md` 示例仍写 v4 的冲突；在组织方澄清前不下载大资产或启动收费沙箱。
- 要求组织方解释或更新冻结环境中的 `release_ready: false`，不得自行把 `release_candidate_ready: true` 解读为正式发布。
- 不实现比赛代码，不提交模型、数据或大文件。

## Capabilities

### New Capabilities

- `competition-a-dpa-deploy`: DPA4C Nano 单 PPU 生产部署优化赛的可验证参赛契约，覆盖九场景输入、正确性门禁、综合加速比、交付物及启动前阻塞条件。

### Modified Capabilities

（无）

## Impact

- 仅影响本仓库中的 OpenSpec 规划材料和 README 比赛摘要，不包含实现代码。
- 正式赛题来源：`DP Arena比赛参赛指南`（https://dptechnology.feishu.cn/wiki/NevpwcEEGi2deek51TMcNTvDnAh）与 Playground 题包。
- 当前直接阻塞：下载目录全量检查未发现 `instruction.md`；资源清单主体为 version 5，但下载说明仍引用 version 4；冻结环境同时记录 `release_candidate_ready: true` 与 `release_ready: false`。
- 比赛页面：https://play.bohrium.com/competitions/dpa
