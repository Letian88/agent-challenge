# Spec Delta

## Purpose

定义推理服务评测赛中可由实际题包证实的固定环境、服务协议、提交格式、能力与压力测试门槛及合规边界，并要求团队在评分、资源和赛期冲突确认前保持显式待确认状态。

## ADDED Requirements

### Requirement: 固定任务目标

参赛服务 SHALL 使用固定 GLM-5.3-Flash 模型、固定 8× `A100-SXM4-80GB` 和固定多轮 Agent 会话负载，在守住服务质量底线的前提下提升可承载逻辑会话数及 token 输出速度。规格 MUST NOT 添加题包未证实的模型规模、MoE 结构或“正式题目完全可见”等断言。

#### Scenario: 识别优化边界
- **WHEN** 团队定义候选方案
- **THEN** 模型、硬件和固定回放负载保持不变，优化对象限于部署、调度、缓存和服务实现

### Requirement: 镜像与提交合同

提交物 SHALL 只包含 `/app/submission/submission.json`。该文件 MUST 包含非空字符串 `image` 和 `command`，可包含字符串映射 `env` 与字符串 `model_name`；省略 `model_name` 时服务 MUST 接受 `model=default`。提交物 MUST NOT 包含 `base_url`、`api_key`、密文、公网 endpoint、`team` 或 `cluster`。镜像 MUST 经 LBG/Bohrium 构建并注册且可被 Trisol `w1` 拉取。

#### Scenario: 校验 submission.json
- **WHEN** 团队组装正式 outputs
- **THEN** 文件只包含允许字段、命令可直接按 argv 执行、凭据不在文件或镜像配置中

### Requirement: Playground 提交

正式提交 SHALL 使用 challenge ID `llm-challenge-arena-v1`，并提供非空 agent trace；trace 首行 MUST 为 `session_start`，其后至少包含一条 user 或 assistant 消息。trace SHALL 只满足 attempt 合同，不参与比赛评分。

#### Scenario: 创建有效 attempt
- **WHEN** 团队执行 Playground 提交
- **THEN** outputs 指向有效 submission.json，trace 结构合格且不含 token，并返回可追踪 attempt ID

### Requirement: 服务基础接口

服务 SHALL 实现 `GET {base_url}/models`、OpenAI 兼容的 `POST {base_url}/chat/completions`、引擎根 `POST /generate` 和引擎根 `POST /flush_cache`，并在可能超过 10 小时的评测窗口内保持可用。`/generate` 与 `/flush_cache` MUST NOT 仅实现在 `/v1` 下。

#### Scenario: 平台检查服务
- **WHEN** 平台以提交的 `model_name` 检查容器
- **THEN** `/models` 返回 200，且三个评测接口均可在规定路径访问

### Requirement: 能力评测门禁

`/chat/completions` 的最终答案 MUST 位于 `choices[0].message.content`，服务 MUST NOT 压低输出预算或截断完整推理。只有 `aime26.points` 与 `gpqa-diamond.points` 均严格大于 90 时，提交 SHALL 进入压力测试。

#### Scenario: 能力门失败
- **WHEN** 任一科 points 小于或等于 90
- **THEN** 压力测试不启动，结果必须如实记录为能力门未通过

### Requirement: 固定轨迹 Generate 协议

`POST /generate` SHALL 接受已渲染 chat template 的原始 `text`、逐请求 `sampling_params`、`stream=true` 和请求 ID。服务 MUST 原样送入 `text`，不得再次套模板；未知 `X-S1-*` 头 MUST NOT 导致拒绝。`ignore_eos=true` 时 MUST 精确输出 `max_new_tokens` 个 token。

响应 SHALL 为 SSE，每个事件以 `data: {...}` 发送并以 `data: [DONE]` 结束。计分事件 MUST 提供累计 `completion_tokens`、`prompt_tokens` 和真实 `cached_tokens`；建议同时提供真实 `request_received_ts` 与 `prefill_finished_time`，否则 TTFT 将退回包含网络抖动的客户端口径。

#### Scenario: 回放冻结请求
- **WHEN** 平台发送预渲染 prompt、`ignore_eos=true` 和逐请求输出预算
- **THEN** 服务不重复套模板，以 SSE 返回精确数量 token 及真实计量元数据

### Requirement: 前缀缓存清理

`POST /flush_cache` SHALL 真实清除前缀 KV，成功时返回 2xx 与 `{"success": true}`。每档正式测量前和切换并发档前 MUST 完成清理；假清缓存或只返回成功 MUST 视为违规。

#### Scenario: 切换并发档位
- **WHEN** 平台准备启动下一并发档
- **THEN** 上一档的前缀 KV 被真实清除，后续缓存命中从已清理状态重新计算

### Requirement: 并发梯子与单档硬门

压力测试 SHALL 使用 `2 / 6 / 10 / 14 / 18 / 22 / …` 的逻辑会话档位，从 N=10 开始探索；`n_at_slo` 为通过全部硬门的最大实测档位。一个档位仅在以下十一项全部成立时通过：`coverage=100%`、`harness_data=0`、`harness_render=0`、`engine_error<1%`、`infra_error<1%`、`fast_intra` TTFT 目标 3 秒、`overall_intra` 5 秒、`turn_start` 15 秒、`chain_start` 30 秒、所有门控 phase 有样本、`tpot_p95≤0.10` 秒/token。TTFT 失败按题包的超标率 95% 单侧下界高于 5% 判定。

#### Scenario: 任一硬门失败
- **WHEN** 某档十一项硬门至少一项不成立
- **THEN** 该档判为 FAIL，不能成为 `n_at_slo`

### Requirement: 评分冲突处理

团队 SHALL 保留并请求组织方消歧以下冲突：详细正文规定按 `n_at_slo` 降序、再按同档 `tpot_mean` 升序；`challenge.json.abstract` 规定 `N@SLO → TPM(decode) → chain_start p95`；结构化评分标为 `human_review_only`。在确认前团队 MUST 同时记录 `n_at_slo`、`tpot_mean`、`tpm_decode` 和 `chain_start_p95`，不得把任一版本写成最终排名事实。

#### Scenario: 选择最终优化目标
- **WHEN** 尚无组织方可追溯的评分确认
- **THEN** 候选必须先满足能力门与单档硬门，并同时保留冲突涉及的四项指标

### Requirement: 公开数据与正式负载边界

正式压力负载 SHALL 为隐藏内容和规模的固定真实多轮轨迹。公开开发集 `s1-dev-combined-v1` 只用于相对回归，MUST NOT 被当作正式 `N@SLO` 的预测。题包 manifest 引用的另一数据集及其 smoke fixture 说明与“submission.json-only”合同冲突，组织方确认前 MUST NOT 擅自把额外 fixture 加入正式提交。

#### Scenario: 使用公开开发集
- **WHEN** 团队用公开集比较候选
- **THEN** 报告明确标注其为相对 A/B 结果，不声称等于正式隐藏负载排名

### Requirement: 质量、安全与真实性

服务 MUST NOT 通过关闭 thinking、压低输出预算、截断历史或删除 tools 换取性能。团队 MUST NOT 攻击、探测或干扰平台，尝试还原隐藏题，写入明文密钥，伪造时间戳/token/cache 计数或虚假清缓存；复核发现上述行为时该提交全部成绩 SHALL 取消。

#### Scenario: 性能收益来自违规手段
- **WHEN** 候选的性能提升依赖质量削减、伪造计量或假清缓存
- **THEN** 该候选不得提交，并记录为违反题包约束

### Requirement: 赛期事实

团队 SHALL 记录仓库与群公告给出的开放时间为 2026-09-18 20:00、当前赛期为动态调整且组织方会在结束前通知；群内澄清的 LLM 提交上限为每天 3 次，单次评测预计约 10 小时。同时 MUST 记录题包 `roundStartAt`、`roundEndAt` 均为空且尚无精确截止时间，不得从评测耗时反推截止日期。

#### Scenario: 安排最终提交
- **WHEN** 组织方尚未发布可追溯的截止时间
- **THEN** 最终提交日期保持待确认，并在每天 3 次的上限内推进不依赖截止日期的准备工作
