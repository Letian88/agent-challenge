# Proposal

## Why

已下载的 `llm-challenge-arena-v1` 题包给出了「推理服务评测赛」的接口、提交和压测硬门，而现有比赛 B 规划仍包含“评测题完全可见”“320B-A18B MoE”等题包无法证实的描述。题包内部的详细正文、摘要和结构化评分元数据还互相冲突，需要在规格中保留事实边界并等待组织方确认。

## What Changes

- 将任务校正为固定 GLM-5.3-Flash、固定 8×A100-SXM4-80GB、固定真实多轮 Agent 会话轨迹下的推理服务部署优化。
- 固化唯一交付合同：`/app/submission/submission.json` 指向获授权镜像与直接执行的启动命令。
- 固化 `/chat/completions`、引擎根 `/generate` SSE、`/flush_cache` 和 `/models` 的外部接口，以及能力门、并发梯子、TTFT/TPOT 硬门和安全约束。
- 明确记录三类未决冲突：详细正文与摘要的排序规则不同、结构化评分标为 `human_review_only`、资源清单与公开开发集/提交格式说明不一致。
- 删除题包无法证实的模型规模、MoE 结构、评测题完全可见、正式截止日期等断言。
- 不实现服务、不启动 GPU、不构建镜像或发起比赛提交。

## Capabilities

### New Capabilities

- `competition-b-llm-deploy`: 推理服务评测赛的固定运行条件、接口与提交合同、能力及 SLO 门禁、合规边界，以及评分、资源和赛期冲突的确认机制。

### Modified Capabilities

（无）

## Impact

- 仅影响本仓库的 OpenSpec 规划材料和 README 比赛摘要；服务实现应位于独立代码仓库。
- 正式赛题来源：`DP Arena比赛参赛指南`（https://dptechnology.feishu.cn/wiki/NevpwcEEGi2deek51TMcNTvDnAh）和 Playground challenge `llm-challenge-arena-v1` 的实际题包。
- 后续实现依赖 Playground CLI、Bohrium/Trisol、LBG 与 Wenyon；模型、数据、镜像和运行日志不进入本 store 仓库。
- 比赛页面：https://play.bohrium.com/llm-arena/competitions/llm-deploy-arena-v1
