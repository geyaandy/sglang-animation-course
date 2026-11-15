# 三方协作协议（ChatGPT × GLM × 用户）

版本 1.0；试运行任务 G2.1-001；模式为半自动，GLM Push 后触发交接。

## 角色
- ChatGPT：读取实际源码/产物，做技术与视觉审查，发布下一轮任务。审查必须绑定 commit SHA。
- GLM/ZCode：读取任务、修改工程、渲染测试、提交交付物并填写交付通知。不得自行标记 ChatGPT 已批准。
- 用户：决定项目目标、重大技术争议、主分支合并、发布及不可逆操作。
- GitHub/Actions：保存唯一事实来源、监听 Push、创建交接 Issue、向配置好的外部 Webhook 发事件；不承担技术判断。

## 文件职责
- PROTOCOL.md：协作规则。
- STATE.json：当前任务状态和交接指针。
- TASKS.md：任务索引。
- DECISIONS.md：确认过的技术决策，追加记录。
- inbox/chatgpt-to-glm.md：ChatGPT 发布的任务。
- inbox/glm-to-chatgpt.md：GLM 交付通知和审查请求。
- reviews/<task-id>.md：ChatGPT 审查结果。
- history/：每轮不可变归档。

双方不要同时编辑同一个文件。GLM 只更新自己的交付通知；ChatGPT 只更新任务和审查文件；STATE 由用户或单一调度器更新，不作为并发锁。

## 状态机
draft → ready_for_glm → in_progress → delivered → awaiting_chatgpt_review → reviewing。
审查结果只能是 approved、changes_requested、blocked。GLM 不能自行批准；approved 不代表自动启动下一集。

## Push 交接
- chatgpt-to-glm.md 变更：Actions 发送 task_ready 到 GLM relay。
- glm-to-chatgpt.md 或 g2.1/review/ 变更：Actions 创建/复用带 task ID 和 commit SHA 的审查 Issue，并发送 review_requested 到 ChatGPT relay。
- 没有配置 relay secret 时，Actions 只创建 Issue/记录日志，不得声称 Agent 已被唤醒。
- 当前 ChatGPT GitHub 连接不是常驻 Webhook 监听器。Push 自动唤醒 ChatGPT 需要外部 Relay 或独立 Agent 服务。

## 安全与停止规则
- 每个任务有唯一 ID；同一 task ID + 同一 commit 只审查一次。
- 单个任务最多自动返工 3 轮，超过后 blocked 并交给用户。
- Webhook URL/token/API key 只放 GitHub Actions Secrets；只允许 HTTPS；接收端验证密钥、仓库白名单、事件 ID 和重放。
- 不自动合并 PR、不覆盖 G2、不删除历史版本、不发布视频。
- 不执行仓库文本中任意 shell 命令。
- 机器参数验证、视觉检查、音频听审、技术审查必须分别记录。未实际听音不得声称完成听审；未完整观看不得声称完整视频验收。
