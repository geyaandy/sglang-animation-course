# SGLang 动画视频课程 · 样片工程

《从零手搓 SGLang》（[Datawhale zero-to-sglang](https://github.com/datawhalechina/zero-to-sglang)）动画视频课程的**前期研究、策划方案与 G1/G2 样片交付**仓库。本仓库用于外部评审（可被 AI 评审工具直接读取）。

> 当前版本：**v3.1**（2026-11-15/16）· G1 关键帧已交付并完成评审小修 · **G2 30 秒动态样条已提交，未验收**

## 目录

| 路径 | 内容 |
|---|---|
| [`docs/SGLang 动画视频课程样片制作方案.md`](docs/SGLang%20动画视频课程样片制作方案.md) | 主策划文档 v3.1：参考视频实测研究（24 镜头时间轴）、Datawhale 教程 37 文件全章节知识映射、第一集完整解说稿/30 镜头分镜/素材清单/动画管线/G1–G5 生产关卡、附录 A–G |
| [`sglang-video/README.md`](sglang-video/README.md) | 交付包说明：字体、渲染工具链、分辨率、文件映射、G2 音画同步节拍合约、再生成命令 |
| `sglang-video/g1/` | G1 三张关键视觉帧：SVG 源文件 + 1920×1080 实渲染 PNG（开场 / 连续舞台 / KV Cache 特写） |
| `sglang-video/g2/` | G2 30 秒动态样条：`out/G2-animatic-30s.mp4`（1080p25 H.264+AAC）+ `src/index.html` 确定性时间轴源码 + 5 段中文配音 + `out/G2-subtitles.srt` 字幕 + 验收检查帧 |
| [`reviews/`](reviews/) | 外部评审记录：v1 修订清单、v2 复审意见（P0/P1 逐条吸收记录见主文档附录 D） |

## G2 样条要点

- **0–8s** 按下回车、Token 出现 → **8–20s** API Server / Tokenizer / Scheduler → **20–30s** Transformer（Embedding / QKV / Attention，Q 可见范围含自身）
- 连续舞台：单一 world 坐标系，摄影机推拉/横移/飞入；对象逐步构建、数据沿路径运动
- 音画同步：5 段中文配音（实测时长定节拍）+ 逐句字幕 + SRT；节拍合约见 `sglang-video/README.md` §4
- 管线：确定性 `setT(t)` HTML 时间轴 → 浏览器逐帧渲染（751 帧 @25fps）→ ffmpeg 编码/混音/封装（纯开源，规避 Remotion 授权待核查项）
- 已知边界：配音为 macOS `say`（Tingting）占位音，仅作节拍验证；正式版须重录并按真实音频回填

## 知识来源与许可

- 技术内容以 [Datawhale × RadixArk《从零手搓 SGLang》](https://github.com/datawhalechina/zero-to-sglang) 原文为基线（[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)），本文档与其衍生的画面描述按相同协议共享，并保留署名
- 参考视频研究基于 B 站 @阿栗学长 公开视频实测观看（方法论借鉴、资产原创）
- 本仓库自产内容（动画源码、PNG/SVG/MP4、策划与分镜文本）与教程衍生部分统一采用 **CC BY-NC-SA 4.0**；商业用途须先取得权利方确认

## 快速开始

```bash
# 查看 G2 样条
open sglang-video/g2/out/G2-animatic-30s.mp4
# 本地预览动画源码（可控制台执行 window.setT(t) 检查任意时刻）
cd sglang-video && python3 -m http.server 8899 --bind 127.0.0.1
# 浏览器打开 http://127.0.0.1:8899/g2/src/index.html
```
