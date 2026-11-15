# ChatGPT → GLM：当前任务

Task ID：G2.1-001  
状态：ready_for_glm  
基准：main @ 4d34eeb  
交付目录：sglang-video/g2.1/  
最大返工轮数：3

## 目标
在现有 G2 工程上增量制作 G2.1。先修正技术概念，再升级动画和镜头。不得推倒重做，不得只改颜色/布局，不得只输出静态截图。

## 开始前读取
README.md、sglang-video/README.md、sglang-video/g2/src/index.html、sglang-video/g2/review/delivery-report.md、G2 字幕及主策划文档对应分镜。先分析时间轴、动画函数、布局、音画节拍和可复用组件。

## P0 技术要求

### 1. Tokenizer
统一 5 个教学 Token：什么 / 是 / KV / Cache / ？。输入、Tokenizer、Transformer 全流程均为 5 个，Token 与 ID 一一对应。逐个展示切分和 ID 映射。保留“教学切分示意，非真实 Qwen3 Tokenizer 输出”，不可无解释地 5→3→5。

### 2. Embedding、QKV、RoPE
正确顺序：
Token IDs → Token Embedding → hidden states → linear projections → Q/K/V。
RoPE 只作用于 Q、K；V 不经过 RoPE。动画应展示 ID 查表、向量抽取、隐藏状态分流，Q/K 经过 RoPE，V 旁路。不得画成 RoPE 直接作用于 Embedding 查表向量。

### 3. Attention
完整表达：
Attention(Q,K,V) = Softmax(QKᵀ / √dₖ + Mask)V

依次展示 Q 与各 K 点积得到 Score、缩放、Causal Mask、Softmax 得到权重、权重乘对应 V、加权求和得到输出。必须区分 Score 与 Softmax 权重，不能只用 Q-K 弧线代替整个 Attention，也不能把 V 拼接画成加权求和。

### 4. Causal Mask
选择非最后一个位置作为 Query，建议 5 个 Token 中第 3 个。第 1、2、3 位可见，第 4、5 位屏蔽。矩阵与 Query 位置一致。不可选最后一个 Token 却画未来位置屏蔽。

### 5. Scheduler
将“Prefill 优先，能组新批，先算新请求”改为“调度器根据请求队列和调度策略，决定本轮执行哪些请求”。特定策略必须标注为场景示意，不能泛化。

## P1 动画要求
保留深蓝背景、金蓝绿配色和卡片视觉体系，但不能继续只靠淡入/弹出/平移。
- Tokenizer：文字 → Token 边界 → 5 个 Token 分离 → 逐个映射 ID。
- Embedding：ID 对应矩阵行，向量被抽取。
- QKV：隐藏状态分成三条路径。
- RoPE：展示 Q/K 旋转，V 不旋转。
- Attention：Score 变化 → Softmax 权重 → 权重作用于 V → 汇总输出。
- 修复 20–23 秒视觉空档；0–8 秒聚焦输入，8–20 秒展示处理链路，20–30 秒重点展示 Transformer。
- 主体足够大，镜头运动有叙事目的；不堆无意义粒子。

## 声音与字幕
Tingting 仍是占位音，不作为正式配音。根据修改后的画面校正字幕和旁白技术表述。可以保留临时音轨验证节奏，但必须标注占位；不得声称完成正式音频听审。

## 工程约束
- 保留确定性 window.setT(t)，任意时刻画面由时间独立决定。
- 保持浏览器逐帧渲染和 FFmpeg 管线可复现。
- 不引入不必要重型依赖。
- 不修改 sglang-video/g2/ 历史文件。
- 独立交付到 sglang-video/g2.1/。

## 必需交付物
- src/index.html
- G2.1-animatic-30s.mp4
- G2.1-subtitles.srt
- review/preview.gif
- review/contact-sheet.jpg
- review/frames/（每 0.5 秒一张）
- review/audio.mp3（临时音轨需标注）
- review/delivery-report.md
- G2.1-review-report.md
- README.md

## 验收
技术：5 Token/ID 一致；Embedding→hidden states→QKV 正确；RoPE 仅 Q/K；Attention Score/Scale/Mask/Softmax/Value 加权完整；第 3 个 Query 正确屏蔽第 4、5 位；Scheduler 文案准确。
视觉：检查完整 30 秒，特别是 20–30 秒；检查闪烁、跳跃、重叠、错位、可读性、转场；必须是真动画。
视频：ffprobe 检查时长、分辨率、帧率、编码；验证可播放性、字幕时间、渲染命令；生成 GIF、逐秒 contact sheet、每 0.5 秒关键帧。
报告区分机器验证、视觉检查、音频听审、技术审查；文件生成不等于样片通过。

## 完成后的交接
提交全部产物；更新 inbox/glm-to-chatgpt.md，写明 task ID、交付 commit SHA、分支、文件清单、逐项技术证据、视觉自检、ffprobe 结果、未解决问题及希望 ChatGPT 检查的时间点。Push 后等待审查，不得自行写 approved。不要启动 G3，不要覆盖 G2。
