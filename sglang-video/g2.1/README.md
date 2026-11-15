# G2.1 动态样条（技术纠错 + 动画升级版）

对应任务：`.collaboration/inbox/chatgpt-to-glm.md` · **G2.1-001**（delivered，待 ChatGPT 审查）
基准：`main@4d34eeb` · 交付分支：`collaboration/g2.1-001`
完整验收记录：[`G2.1-review-report.md`](G2.1-review-report.md)

## 与 G2 的差异（G2 保持不变，本目录独立交付）

| P0 | 修正内容 | 时间点 |
|---|---|---|
| 1 | Tokenizer 节点补齐 5 Token ⇄ 5 ID（修复 5→3→5 断裂），输入/Tokenizer/内部三处一致 | 3.2–6.6 / 10.6–12.6 / 20.6–21.8s |
| 2 | 明确 ID→嵌入表→Hidden States→线性投影（W_Q/W_K/W_V）分流；RoPE 仅旋转 Q/K（双旋转站点），V 虚线旁路 | 20.6–26.3s |
| 3 | Attention 完整五阶段：Score（点积数值）→ Scale（÷√dₖ）→ Causal Mask（−∞）→ Softmax（权重 27/40/33/0/0%，和为 1）→ 权重×V 加权求和（Σ） | 26.7–29.8s |
| 4 | Causal Mask 改用**第 3 个 Token** 作 Query：可见 1/2/3，第 4、5 位 −∞/权重 0% | 26.5–28.3s |
| 5 | Scheduler 文案改为中性表述并标注"策略为场景示意" | 13.8s 起 |

P1 动画升级：Tokenizer 切分边界→分离→ID 映射逐段动画；嵌入表行抽取；Hidden States 分流三路径；RoPE 旋转对比；Attention 五阶段推进；修复 20–23s 视觉空档（20.3–24.0s 无空帧）。

## 文件

| 文件 | 说明 |
|---|---|
| `src/index.html` | 确定性 `window.setT(t)` 工程源码（含 P0 修正注释） |
| `out/G2.1-animatic-30s.mp4` | ★ 成片：30.04s · 1920×1080 · 25fps · H.264+AAC |
| `out/G2.1-subtitles.srt` | 字幕 5 条（文案已按 P0 修正） |
| `audio/` | 占位配音（Tingting；seg3/4a/4b 重录） |
| `review/` | preview.gif / contact-sheet.jpg / frames/×60 / audio.mp3 / delivery-report.md / evidence/P0-×8 |
| `G2.1-review-report.md` | 验收报告（机器验证/视觉检查/音频/技术审查分节） |

## 快速预览

```bash
open out/G2.1-animatic-30s.mp4
# 逐帧源码预览：本地服务后打开 src/index.html，控制台 window.setT(t)
```

配音为 macOS `say` 占位音，非正式配音；正式版须重录并回填节拍。
