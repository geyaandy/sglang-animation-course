# GLM → ChatGPT：交付通知

当前状态：delivered（等待 ChatGPT 审查；GLM 未自行标记 approved）

- Task ID：G2.1-001
- Base commit：4d34eeb（审查基准）
- Delivery commit SHA：`ed3512532bf842bc44e1930c5ed86b96bc28f840`（交付内容 commit；通知更新 commit 为分支 HEAD，见「交付方式」）
- Branch：`collaboration/g2.1-001`
- MP4 路径：`sglang-video/g2.1/out/G2.1-animatic-30s.mp4`
- 源码路径：`sglang-video/g2.1/src/index.html`
- SRT 路径：`sglang-video/g2.1/out/G2.1-subtitles.srt`
- Contact sheet 路径：`sglang-video/g2.1/review/contact-sheet.jpg`
- Preview GIF 路径：`sglang-video/g2.1/review/preview.gif`
- Delivery report 路径：`sglang-video/g2.1/review/delivery-report.md`
- G2.1 review report 路径：`sglang-video/g2.1/G2.1-review-report.md`

## 技术验收证据（逐项，详见 G2.1-review-report.md §1）

- **P0-1 Tokenizer**：输入/Tokenizer/内部三处均 5 Token（什么/是/KV/Cache/？）+ id₁–id₅ 一一对应；Tokenizer 节点补齐至 5 个（修复 G2 的 5→3→5）。证据帧 `review/evidence/P0-1_tokenizer-5token-t6.2.png`、`P0-1_tokenizer-5id-t12.4.png`；源码元素 `qc0–4 / tid0–4 / ich0–4`。
- **P0-2 Embedding/QKV/RoPE**：链路 ID→嵌入表→Hidden States(H₁–H₅)→线性投影 W_Q/W_K/W_V→Q/K/V 三行；RoPE 站点仅设在 Q/K 行入口（刻度随 t 旋转），V 行虚线旁路标注"不经 RoPE"。证据帧 `P0-2_embedding-hidden-t24.8.png`、`P0-2_qkv-ropelanes-t26.2.png`；源码元素 `itab/hsrow/splitq·k·v/wq·wk·wv/ropeQ·ropeK/vbyp`。
- **P0-3 Attention 五阶段**：Score（27.1–27.55s，4.1/5.2/4.6/3.9/3.7）→ Scale（÷√dₖ→1.45/1.84/1.63/1.38/1.31）→ Mask（27.6s，第 4、5 位 −∞）→ Softmax（28.15s→27%/40%/33%/0%/0%，进度条，和为 1）→ 权重×V（29.0s 起，×0.27/×0.40/×0.33/×0/×0 连线汇入 Σ）。Score 与 Softmax 权重由 setT 按 t 确定性切换，未混用。证据帧 `P0-3_score-t27.3.png`、`P0-3_weighted-sum-t29.6.png`；源码 `sc0–4/stv0–4/wbar0–4/wcon0–4/sum/outv`。
- **P0-4 Causal Mask**：Query＝第 3 个 Token（Q₃ 高亮 + 金色连线至 Score 列），可见位置 1/2/3；第 4、5 位 −∞→权重 0%。证据帧 `P0-4_mask-t28.3.png`；源码元素 `q3hl/q3line/mtxt/stv3–4`。
- **P0-5 Scheduler**：文案改为"调度器根据请求队列和调度策略，决定本轮执行哪些请求（策略为场景示意）"，原 Prefill 优先表述已删除；字幕第 3 条同步修正。证据帧 `P0-5_scheduler-copy-t16.3.png`；源码元素 `schpol`。

## 视觉自检

检查方式：浏览器对 t=5.8/12.4/16.3/24.8/26.2/27.3/28.3/28.6/29.6 逐点截图目检 + 联络表 30 格整排目检。
- 20–23s 空档已修复：20.3 标题 → 20.6–21.8 五 ID Chip → 21.8 查表箭头 → 21.3 嵌入表 → 22.1–22.9 表行点亮 → 23.0 抽取 → 23.1–23.9 Hidden States 组装。
- 转场：镜头推拉/横移均有叙事目的（输入特写→全景→调度特写→飞入引擎→跟随数据流）；主体放大（注意力段单元 92–108px）。
- 已目检修复：Q₃ 高亮框错套 Q₁（已改 x=3772）；Hidden 行与 W_K/K 行重叠（已重排：H 行 3060–3412，W 芯片 3450–3546，行 3580–4072）。
- 残留小问题：注意力段右侧 stage 标签（÷√dₖ/Mask/Softmax）与 Σ 空间较近，可读但偏紧。

## 媒体验证

- ffprobe：30.04s / 1920×1080 / 25fps / H.264(CRF18 yuv420p faststart) / AAC 192k 44.1kHz 立体声 / 9.32MB。
- 播放验证：成片抽帧（review/evidence 8 张）与联络表 30 格目检均正常；GIF 240 帧/30.01s 可播放。
- frames/ 60 张（0.5s 步长，覆盖 0–29.5s）无缺号；SRT 5 条与配音/画面逐句对齐。
- 音频：volumedetect mean −18.3dB / max −2.4dB，无削波。**配音为 macOS say（Tingting）占位音，非正式配音；未做人耳正式听审。**

## 未解决问题

1. 配音占位（Tingting），正式版须重录并回填节拍。
2. 联络表时间戳由 PIL 绘制（本机 ffmpeg 未编译 drawtext）。
3. Score/权重为教学示意数值（softmax 自洽），非 Qwen3 真实运行输出；如需实测可后续以 Qwen3-0.6B 真跑替换并挂"实测"徽章。
4. 注意力段右侧 stage 标签与 Σ 间距偏紧。
5. 基准说明：分支自 `origin/main@1131126` 创建（`4d34eeb` 为其祖先；4d34eeb→1131126 差异仅含 .collaboration/、.github/workflows/、README，不含 sglang-video/ 任何文件）。审查 diff 仍以 4d34eeb 计算：`git diff 4d34eeb..collaboration/g2.1-001 -- sglang-video/g2/` 为空，G2 与 STATE.json 未被修改。

## 请求 ChatGPT 重点审查

- t=27.0–28.7：Attention 五阶段数值与阶段划分是否准确、Score/权重区分是否清晰。
- t=25.4–26.4：Q/K/V 分流与 RoPE 仅 Q/K 的表达是否严谨。
- t=10.6–12.6 与 t=3.2–6.6：Tokenizer 5⇄5 一致性。
- t=13.8–17.5：Scheduler 中性文案与"场景示意"标注。
- 全片 20–23s：空档修复后的节奏与信息密度。

## 交付方式

交付内容 commit：`ed3512532bf842bc44e1930c5ed86b96bc28f840`（含全部 G2.1 文件）。
本通知更新 commit：推送后的分支 HEAD（含本文件更新）。审查绑定建议使用 HEAD SHA 或 ed35125 皆可，二者 diff 仅本通知文件。

交接记录：2026-11-16 首次推送。注：新分支首次推送时 GitHub `before` 为全零，collaboration-handoff.yml 会在分类前提前返回（已知局限），本行变更用于触发 handoff 以创建审查 Issue。
