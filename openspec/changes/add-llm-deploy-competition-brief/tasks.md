# Tasks

## 1. 规则、资源与赛期确认

- [ ] 1.1 向组织方确认最终排序采用详细正文、摘要还是 `human_review_only` 元数据，并以可追溯回复更新规格作为完成验证
- [ ] 1.2 用只读命令核验 `arena` 成员关系和 quota，并以真实查询结果纠正“待审批/已批准”冲突作为完成验证
- [ ] 1.3 确认 `s1-dev-combined-v1`、manifest 数据集和 smoke fixture 的用途及正式提交是否严格 submission.json-only，并以书面结论作为完成验证
- [ ] 1.4 确认正式截止时间、时区和提交次数限制，并以组织方公告链接更新团队时间线作为完成验证

## 2. 独立实现仓库与基线

- [ ] 2.1 为比赛 B 创建独立代码仓库并登记到 README 项目表，以 store 中不出现模型、数据、镜像层或服务代码作为完成验证
- [ ] 2.2 固定获授权基镜像、GLM-5.3-Flash 模型和 tokenizer revision，并以 Trisol `w1` 可拉取且版本记录可复现作为完成验证
- [ ] 2.3 在资源冲突确认后获取公开开发集，并以题包所述 harness、固定 tokenizer 和请求清单完整作为完成验证
- [ ] 2.4 建立未经优化的同协议基线，记录四道 TTFT、TPOT、TPM、缓存命中和错误率，并以报告可由原始记录复算作为完成验证

## 3. 服务协议实现

- [ ] 3.1 实现 `/models` 与 OpenAI 兼容 `/chat/completions`，以目标模型名请求成功且最终答案位于 `choices[0].message.content` 作为完成验证
- [ ] 3.2 在引擎根实现 `/generate` SSE，原样处理预渲染 prompt、逐请求 `max_new_tokens` 和 `ignore_eos`，以输出 token 数精确且 `[DONE]` 正确结束作为完成验证
- [ ] 3.3 上报真实 `completion_tokens`、`prompt_tokens`、`cached_tokens`、`request_received_ts` 和 `prefill_finished_time`，以 token 与 TTFT 可独立复算作为完成验证
- [ ] 3.4 实现真实 `/flush_cache`，以同 prompt 的缓存命中在 warm 后上涨、flush 后回到清理状态作为完成验证
- [ ] 3.5 生成只含允许字段的 `/app/submission/submission.json`，以命令无需 shell 展开且文件不含 endpoint、team、cluster 或凭据作为完成验证

## 4. 分层验证与性能迭代

- [ ] 4.1 对四个接口做容器级冒烟和冻结 prompt token 一致性测试，以全部路径、协议、状态码与 token 数符合规格作为完成验证
- [ ] 4.2 在成员与 quota 确认后于 `arena` / `w1` / 8×A100 上启动基线并在测试结束后删除资源，以服务 running、报告落盘和资源释放作为完成验证
- [ ] 4.3 使用公开开发集执行 preflight、warmup、flush 和正式测量，以 coverage 100%、engine error 0 且报告标注“不能预测正式 N@SLO”作为完成验证
- [ ] 4.4 每轮只改变一个部署变量并记录质量门、十一项硬门及四个冲突排序指标，以结果可与固定基线比较作为完成验证
- [ ] 4.5 淘汰任何能力质量、服务协议或压力硬门失败的候选，以最终候选已有完整门禁证据作为完成验证

## 5. 构建与提交

- [ ] 5.1 通过 LBG 构建注册最终镜像并固定版本，以干净 Trisol 环境可拉取且四个接口通过作为完成验证
- [ ] 5.2 创建不含凭据的合格 trace，并以 `session_start` 加至少一条 user/assistant 消息通过 CLI 校验作为完成验证
- [ ] 5.3 用 challenge ID `llm-challenge-arena-v1` 发起提交并保存 attempt ID，以平台收件结果可追踪作为完成验证
- [ ] 5.4 如实记录能力门、各并发档硬门、最终指标与失败原因，以未完成评分不被描述为最终名次作为完成验证
- [ ] 5.5 保存获授权镜像、启动参数、代码版本和脱敏证据，以赛后可复核真实时间、token 计数和缓存清理作为完成验证
