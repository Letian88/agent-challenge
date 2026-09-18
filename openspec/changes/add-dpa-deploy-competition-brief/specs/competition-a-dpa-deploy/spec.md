# Spec Delta

## Purpose

定义 DPA4C Nano 单 PPU 生产部署优化赛的可验证参赛契约，使团队只在题包与资源一致时启动计算，并以统一正确性门槛、九场景综合加速比和可复现交付物推进参赛。

## ADDED Requirements

### Requirement: 任务目标与固定对象

参赛实现 SHALL 在单张 PPU810E 96GB 上优化固定 DPA4C Nano 模型驱动的完整 LAMMPS MD 步，并 MUST 保持规定的数值精度与物理行为。固定模型权重、模型结构、元素映射和 6 Å 物理截断距离 MUST NOT 被替换或修改。

#### Scenario: 确认优化对象
- **WHEN** 团队选择一项性能优化
- **THEN** 该优化必须作用于完整 MD 步的真实执行路径，而不是改变模型、物理问题或评测工作量

### Requirement: 题包与资源完整性门禁

团队 MUST 只在下载目录中定位到同时包含 `instruction.md` 与 `asset/tools/arena.py` 的唯一题包根目录后继续。若缺少任一文件、根目录不唯一、资源版本存在冲突，或冻结环境仍为 `release_ready: false` 且未经组织方解释，团队 MUST 停止、联系组织方，并 MUST NOT 启动收费沙箱。

#### Scenario: 当前题包缺失文件
- **WHEN** 全量遍历下载目录仍找不到 `instruction.md`
- **THEN** 状态必须记录为阻塞，且沙箱、数据恢复、优化和评测任务不得开始

#### Scenario: 资源版本冲突
- **WHEN** challenge/config/落盘目录指向 v5 而下载说明示例指向 v4
- **THEN** 团队必须取得组织方的明确版本与哈希确认后才可下载大资产

#### Scenario: 冻结环境尚未正式就绪
- **WHEN** 环境配置同时记录 `release_candidate_ready: true` 与 `release_ready: false`
- **THEN** 团队必须取得组织方解释或更新后的冻结配置，不得自行推断可以启动收费资源

### Requirement: 评测输入

公开评测 SHALL 覆盖 CuNi、Si、MgO 各小、中、大三个规模，共九个场景；CuNi 和 Si 使用 NVT，MgO 使用 NPT，步长 1 fs、thermo 每 10 步、轨迹每 1000 步。候选 MUST 接受评测器提供的新结构与工况，不得只支持公开起点。

#### Scenario: 完整九题评测
- **WHEN** 团队生成一份可提交成绩
- **THEN** 同一次评测必须包含三个材料的全部九个场景，缺题不得形成有效综合成绩

#### Scenario: 新输入兼容
- **WHEN** 平台在题面给定的原子数、250–600 K、0–1000 bar及 Cu 比例 0.25–0.75 范围内提供新输入
- **THEN** 候选必须在不硬编码公开结构或工况的情况下执行

### Requirement: 正确性门槛

候选 MUST 同时满足：能量误差不超过 `2e-5 eV/atom`、力 RMSE 不超过 `4e-5 eV/Å`、最大力分量误差不超过 `5e-4 eV/Å`、最大应力分量误差不超过 `4e-4 GPa`、5 ps NVE 漂移不超过 `6e-6 eV/atom/ps`，并通过周期边界、变胞、邻居表、对称性及 NVT/NPT 统计检查。

#### Scenario: 正确性失败
- **WHEN** 任一数值门槛、物理检查或九题 case 未通过
- **THEN** 该实现不得形成有效综合成绩或进入最终交付

### Requirement: 评分规则

每个场景 SHALL 进行三轮基线/候选交替配对测量，场景分数为三轮 `t_baseline/t_candidate` 的中位数；综合分 SHALL 为九个场景分数的等权几何平均。九题全部通过后按未舍入综合加速比降序排名，辅助指标 MUST NOT 额外加分。

#### Scenario: 计算有效综合分
- **WHEN** 九个场景均通过正确性检查且来自同一次完整评测
- **THEN** `results.json` 的 `geomean_md_speedup` 必须由九个场景分数等权计算，且不得拼接历史最好成绩

### Requirement: 允许与禁止的优化

团队 SHALL 允许使用压缩查表、算子融合、手写 kernel、编译优化、图捕获和通过正确性检查的混合精度。团队 MUST NOT 重训练、替换或修改模型权重/结构/元素映射，缩短物理截断距离，预存答案，或改变步数、步长、积分器、温压控制和输出频率来提速。

#### Scenario: 合规审查
- **WHEN** 团队准备保留一项加速改动
- **THEN** 必须证明该改动未改变模型、物理问题、输入或规定工作量，并已通过正确性门槛

### Requirement: 交付物与提交限制

每份提交 MUST 包含同一次完整评测生成的 `results.json`、真实 `optimization.md`、实际构建材料 `build/` 和完整 `logs/`；申请组织方运行或最终镜像评测时还 MUST 包含带不可变 digest 的 `image.json` 与通过的镜像接口自测。真实 agent 会话 JSONL MUST 通过 `--trace` 单独提交，且 MUST NOT 伪造或含凭据。每位选手每天 MUST NOT 超过 5 次提交。

#### Scenario: 打包普通自测提交
- **WHEN** 团队只提交赛期自测成绩
- **THEN** 提交目录必须至少包含通过的九题 `results.json`、优化记录、构建材料和日志，并附真实会话 trace

#### Scenario: 申请平台镜像评测
- **WHEN** 团队请求组织方运行候选镜像
- **THEN** 必须增加不可变镜像 digest、实际 `build/Dockerfile`、`build/engine.json` 和通过的干净镜像自测证据

### Requirement: 时间事实

团队 SHALL 记录题包未提供正式开始时间或截止时间；任何截止日期 MUST 以组织方后续公告为准，不得从空的 round 字段推断。

#### Scenario: 汇报时间节点
- **WHEN** 团队汇报赛程
- **THEN** 必须如实写明“题包未给出正式截止日期”，并仅列出已证实的每日提交上限和运行超时
