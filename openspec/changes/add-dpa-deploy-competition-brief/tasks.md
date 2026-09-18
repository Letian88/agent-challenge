# Tasks

## 1. 解除题包硬阻塞

- [ ] 1.1 向组织方报告 challenge ID、实际下载路径和缺少 `instruction.md` 的文件清单，并以获得完整题包或书面处置意见作为完成验证
- [ ] 1.2 向组织方确认大资产应使用 version 4 还是 version 5，并记录最终版本、数据集 slug 与哈希作为完成验证
- [ ] 1.3 向组织方确认 `release_ready: false` 的含义或取得更新后的冻结配置，并以书面说明或新配置哈希作为完成验证
- [ ] 1.4 重新下载后定位唯一题包根目录，并以该目录同时存在 `instruction.md` 与 `asset/tools/arena.py` 作为完成验证；未通过前不得执行后续收费步骤

## 2. 账号、算力与数据准备

- [ ] 2.1 验证 Playground 身份和 Trisol `arena` team 状态，并以 `authenticated: true`、团队成员状态及配额查询成功作为完成验证
- [ ] 2.2 在完整题包根目录启动单 PPU 沙箱，并以脚本输出 `status: passed`、沙箱 ID 和 `/workspace/dpa-md` 题包路径作为完成验证
- [ ] 2.3 按组织方确认版本下载并恢复 128 个资产文件，并以 fetch 输出 `status: passed`、`files_verified: 128` 及哈希验证成功作为完成验证
- [ ] 2.4 运行基础 selftest，并以退出码 0、`status: passed` 和 `correctness.passed: true` 作为完成验证

## 3. 优化与验证

- [ ] 3.1 建立 `optimization.md`，记录基线身份、热点、改动、精度策略、构建复现步骤和已知限制，并以记录可追溯到具体运行目录作为完成验证
- [ ] 3.2 对候选做 profiling，选择有数据支持的主要耗时路径，并以报告中记录主机等待、设备同步、拷贝、邻居图或算子耗时作为完成验证
- [ ] 3.3 每项改动先运行全部数值回归，再运行至少一个配对场景，并以全部 frame 通过且配对结果可复现作为完成验证
- [ ] 3.4 在改动稳定后运行同一次完整九题评测，并以退出码 0、`status: passed`、`correctness.passed: true` 和九个 case 完整作为完成验证

## 4. 镜像与提交

- [ ] 4.1 将所有有效手工改动同步到实际 Dockerfile、脚本、补丁和 engine 配置，并以从不可变 digest 启动干净沙箱可复现作为完成验证
- [ ] 4.2 运行镜像接口自测，并以三个小体系均通过、`status: passed` 和 `correctness.passed: true` 作为完成验证
- [ ] 4.3 用官方工具整理 `results.json`、`optimization.md`、`build/`、`logs/` 及按需的 `image.json`，并以打包校验通过且 ZIP 不超过 512 MiB 作为完成验证
- [ ] 4.4 通过 Playground CLI 提交交付目录和真实会话 trace，保存 attempt ID，并以收件状态、审核状态和最终组织方结果的如实记录作为完成验证
