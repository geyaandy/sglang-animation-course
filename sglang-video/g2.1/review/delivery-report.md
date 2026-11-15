# G2.1 交付报告 · 审查材料（review/）

生成日期：2026-11-16
来源成片：`sglang-video/g2.1/out/G2.1-animatic-30s.mp4`（分支 `collaboration/g2.1-001`）
生成方式：全部由该 MP4 **只读派生**——未修改成片、未修改动画源码、未重新配音。

## 1. 来源成片参数（ffprobe 实测）

| 项 | 值 |
|---|---|
| 时长 | 30.04 s |
| 分辨率 | 1920×1080 |
| 帧率 | 25 fps |
| 视频编码 | H.264（libx264，CRF 18，yuv420p，faststart） |
| 音轨 | AAC 192 kbps，44.1 kHz，立体声 |
| 音量 | mean −18.3 dB / max −2.4 dB（volumedetect） |
| 字幕 | 外挂 `out/G2.1-subtitles.srt`（5 条）；画面内另有烧录字幕条 |
| 配音属性 | **占位音（macOS `say` · Tingting）**，非正式配音 |

## 2. 审查材料清单

| 文件 | 规格 | 用途 |
|---|---|---|
| `preview.gif` | 960×540 · 240 帧 · 8 fps · 30.01 s · 约 18MB · 两遍调色板 | 无播放器快速全览 |
| `contact-sheet.jpg` | 2880×1350 · 30 格（6×5）· 每秒一帧 t=0s…t=29s · 时间戳标注 | 逐秒节奏审查 |
| `frames/f_001.jpg … f_060.jpg` | 1280×720 JPEG · 每 0.5s 一张：`f_NNN.jpg` 对应 **t=(NNN−1)×0.5 s**，覆盖 0–29.5 s | 逐关键帧细审 |
| `audio.mp3` | 128 kbps · 44.1 kHz · 立体声 · 30.0 s · mean −18.3 dB / max −2.4 dB | 独立听审 |
| `evidence/P0-*.png` | 8 张 P0 验收证据帧（自成片抽取，文件名含时间点） | 技术审查证据 |
| `delivery-report.md` | 本文件 | 参数与验证记录 |

## 3. 验证记录（2026-11-16）

| 检查项 | 方法 | 结果 |
|---|---|---|
| GIF 可播放 | ffprobe：codec=gif，nb_frames=240，duration=30.01 s，960×540 | ✅ |
| 音频完整性 | ffprobe duration=30.0 s；volumedetect 峰值 −2.4 dB 无削波 | ✅ |
| 关键帧时间连续 | 60 张 = f_001(t=0.0s) → f_060(t=29.5s)，步长 0.5s，无缺号 | ✅ |
| 联络表 | 目检 30 格时间戳按时间顺序排列 | ✅ |
| 成片编码 | ffprobe：h264 / 1920×1080 / 25fps / aac 44.1kHz 立体声 / 30.04s | ✅ |
| 来源 MP4 未改动 | 生成过程仅只读输入 | ✅ |

## 4. 生成命令（可复现）

```bash
MP4=out/G2.1-animatic-30s.mp4
ffmpeg -i $MP4 -vf "fps=8,scale=960:-2:flags=lanczos,palettegen=stats_mode=diff" /tmp/pal.png
ffmpeg -i $MP4 -i /tmp/pal.png -lavfi "fps=8,scale=960:-2[x];[x][1:v]paletteuse=dither=bayer:bayer_scale=4" -loop 0 review/preview.gif
ffmpeg -i $MP4 -vf "fps=1,scale=480:-2" /tmp/cs21/t%02d.png   # 30 张 → PIL 合成联络表
ffmpeg -i $MP4 -vf "fps=2,scale=1280:-2:flags=lanczos" -q:v 4 review/frames/f_%03d.jpg
ffmpeg -i $MP4 -vn -c:a libmp3lame -b:a 128k -ar 44100 review/audio.mp3
```

## 5. 说明与边界

1. `frames/` 覆盖 0–29.5 s（fps=2 对 30.04 s 的自然取整）；末尾 0.5s 为静帧保持。
2. GIF 为 256 色格式，精确色彩以 MP4/JPEG 为准。
3. 联络表时间戳由 PIL 绘制（本机 ffmpeg 未编译 drawtext）。
4. 配音为 macOS `say`（Tingting）**占位音**，仅节拍验证；正式版须重录并回填节拍。
