# 已确认决策

## C-001：协作模式
用户选择半自动；GLM Push 后触发审查交接。ChatGPT 负责审查和下一轮任务设计；GLM 负责执行；用户保留重大决策权。

## C-002：G2.1 版本策略
G2 保持不变。G2.1 使用独立目录 sglang-video/g2.1/。G2.1 未经 ChatGPT 审查通过，不启动 G3。

## C-003：自动化边界
GitHub Actions 在 Push 后执行事件交接。ChatGPT 自动唤醒需要外部 Relay；目前尚未配置。Relay 未配置时以 GitHub Issue 作为待审入口。
