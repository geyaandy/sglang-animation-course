# G2 交付报告 · 审查材料（review/）

生成日期：2026-11-16
来源成片：`sglang-video/g2/out/G2-animatic-30s.mp4`（仓库源 commit `85e5a64`）
生成方式：全部由该 MP4 **只读派生**——未重新制作/渲染视频，未改动动画源码，未重新生成配音。

## 1. 来源成片参数（实测）

| 项 | 值 |
|---|---|
| 时长 | 30.04 s |
| 分辨率 | 1920×1080 |
| 帧率 | 25 fps |
| 视频编码 | H.264（libx264，CRF 18，yuv420p，faststart） |
| 音轨 | AAC 192 kbps，44.1 kHz，立体声 |
| 字幕 | 外挂 `out/G2-subtitles.srt`（5 条，与配音逐句对齐）；画面内另有烧录字幕条 |
| 音量 | mean −19.1 dB / max −2.7 dB（volumedetect） |

## 2. 审查材料清单

| 文件 | 规格 | 用途 |
|---|---|---|
| `preview.gif` | 960×540 · 240 帧 · 8 fps · 30.01 s · 16.5 MB · 两遍调色板（palettegen stats_mode=diff + paletteuse bayer_scale=4）· 无限循环 | 快速全览 30s 动画，无需播放器 |
| `contact-sheet.jpg` | 2880×1350 · 30 格（6×5）· 每秒一帧 t=0s…t=29s · 时间戳标注 | 逐秒节奏审查 |
| `frames/f_001.jpg … f_060.jpg` | 每张 1280×720 JPEG（q=4）· 每 0.5s 一张，共 60 张：`f_NNN.jpg` 对应 **t = (NNN−1)×0.5 s**，覆盖 0–29.5 s | 逐关键帧细审（含 GIF/JPEG 压缩伪影对照） |
| `audio.mp3` | 128 kbps · 44.1 kHz · 立体声 · 30.0 s · 481 KB · mean −19.1 dB / max −2.7 dB | 独立听审配音/音效 |
| `delivery-report.md` | 本文件 | 参数与验证记录 |

## 3. 验证记录（2026-11-16）

| 检查项 | 方法 | 结果 |
|---|---|---|
| GIF 可播放 | ffprobe：codec=gif，nb_frames=240，duration=30.01 s；帧率 8 fps（GIF 帧延迟以厘秒存储，12/13 cs 交替，实测平均 ≈8.0 fps） | ✅ |
| GIF 规格 | 960×540（16:9 保持） | ✅ |
| 音频完整性 | ffprobe duration=30.00 s；volumedetect 峰值 −2.7 dB 无削波 | ✅ |
| 关键帧时间连续 | 60 张 = f_001(t=0.0s) → f_060(t=29.5s)，步长 0.5s，无缺号 | ✅ |
| 联络表 | 目检 30 格时间戳 t=0s…t=29s，按时间顺序行主排列 | ✅ |
| 来源 MP4 未改动 | 生成过程仅 `-i` 只读输入 | ✅ |

## 4. 生成命令（可复现）

```bash
MP4=out/G2-animatic-30s.mp4
# GIF（两遍调色板）
ffmpeg -i $MP4 -vf "fps=8,scale=960:-2:flags=lanczos,palettegen=stats_mode=diff" /tmp/pal.png
ffmpeg -i $MP4 -i /tmp/pal.png -lavfi "fps=8,scale=960:-2[x];[x][1:v]paletteuse=dither=bayer:bayer_scale=4" -loop 0 review/preview.gif
# 联络表（本机 ffmpeg 无 drawtext 滤镜 → 先抽帧后用 PIL 合成时间戳）
ffmpeg -i $MP4 -vf "fps=1,scale=480:-2" /tmp/cs/t%02d.png   # 30 张
# Python/PIL：6×5 网格 + "t=Xs" 标注 → contact-sheet.jpg
# 关键帧
ffmpeg -i $MP4 -vf "fps=2,scale=1280:-2:flags=lanczos" -q:v 4 review/frames/f_%03d.jpg
# 音轨
ffmpeg -i $MP4 -vn -c:a libmp3lame -b:a 128k -ar 44100 review/audio.mp3
```

## 5. 说明与边界

1. `frames/` 共 60 张、覆盖 0–29.5 s（fps=2 对 30.04 s 时长的自然取整）；末尾 0.5 s 为静帧保持，无信息损失。
2. GIF 为 256 色格式，深蓝渐变与金色高光有轻微色带（bayer 抖动已最小化）；精确色彩请以 MP4/JPEG 为准。
3. 联络表时间戳由 PIL 绘制（本机 ffmpeg 构建未编译 drawtext 滤镜）；标签仅 ASCII 字符。
4. 配音为 macOS `say`（Tingting）占位音，节拍验证用途——正式版须重录并回填节拍（见 `sglang-video/README.md` §6）。
