# SGLang 动画课 · 交付包（G1 关键帧 + G2 动态样条）

版本：v3.1 配套交付包 · 2026-11-15
对应策划文档：`../SGLang 动画视频课程样片制作方案.md`（v3.1，附录 F 为交付记录）

## 1. 目录结构

```
sglang-video/
├── README.md                        ← 本文件
├── g1/                              ← G1 三张关键视觉帧（已评审通过，含小修）
│   ├── frame1-opening.svg           源文件：开场帧（S01 定格）
│   ├── frame2-stage.svg             源文件：连续舞台帧（S04/S05）
│   ├── frame3-kvcache.svg           源文件：KV Cache 特写帧（S18/S19）
│   └── rendered/
│       ├── G1-frame1-opening.png    1920×1080 实渲染 PNG
│       ├── G1-frame2-stage.png
│       └── G1-frame3-kvcache.png
└── g2/                              ← G2 30 秒动态样条（已提交，未验收）
    ├── src/
    │   └── index.html               全部工程源码（HTML+SVG+确定性时间轴 JS）
    ├── audio/
    │   ├── seg1.wav … seg4b.wav     5 段中文配音（macOS say · Tingting）
    ├── out/
    │   ├── G2-animatic-30s.mp4      ★ 成片：1080p25 H.264 + AAC，30.04s，约 8.3MB
    │   ├── G2-subtitles.srt         外挂字幕（与配音逐句对齐）
    │   ├── vo30.wav                 混音后的 30s 配音母带（44.1kHz 立体声）
    │   ├── g2video_silent.mp4       无声视频中间产物
    │   └── check_t5/t17/t27.png     从成片抽取的验收检查帧
    └── （渲染中间帧在 /tmp/g2frames，未入库；再生成命令见 §5）
```

## 2. 字体与设计体系

- 字体：系统 **PingFang SC**（回退 Hiragino Sans GB / Microsoft YaHei）；下标使用 Unicode 字符（id₁、t₁、K₁）
- 颜色语义（全项目统一）：金 `#f5c24b`=动作/选中；蓝 `#7fb3e8`=中性数据/K；绿 `#79c7a3`=V；紫=EOS；红 `#e0705f`=风险/掩码；青 `#58c2b8`=top-p/扫描/待追加
- 背景：深蓝 `#0a1524–#16294a` 径向渐变 + 点阵网格 + 透视地板线
- 所有"教学示意/原文计算/参考视频实测"角标随画面标注（徽章体系见方案 §3.5.3 S-02）

## 3. 渲染工具链

| 环节 | 工具 | 说明 |
|---|---|---|
| G1 关键帧 | ZCode 内置浏览器（Chromium）打开 SVG，视口 1920×1080，整页截图 → PNG | 无 cairo 依赖，中文渲染精确 |
| G2 动画源 | 单文件 `index.html`：`window.setT(t)` 确定性时间轴（画面完全由 t 决定），摄影机=world 容器 translate+scale 关键帧插值 | 连续舞台、推拉横移、逐步构建、数据沿路径运动 |
| G2 逐帧渲染 | 对 751 帧（t=i/25, i=0…750）逐帧 `evaluate(setT(t))` + 截图 → `/tmp/g2frames/f%05d.png` | 每帧约 0.6s，总计约 8 分钟 |
| 视频编码 | ffmpeg 8.1：`-framerate 25 -c:v libx264 -crf 18 -pix_fmt yuv420p` | 1080p25 |
| 中文配音 | macOS `say -v Tingting`（zh_CN），5 段分别生成后测时长 | 占位级 TTS，正式版建议人工/商用 TTS 重录 |
| 混音 | ffmpeg adelay（偏移 500/8200/14000/20300/25780 ms）+ amix + apad=30s，volume 1.6 | 峰值 -2.3dB，均值 -18.6dB |
| 封装 | `-c:v copy -c:a aac -b:a 192k -movflags +faststart -t 30` | H.264+AAC MP4 |

## 4. G2 音画同步表（节拍合约）

| 配音 | 起点 | 实测时长 | 画面节拍 |
|---|---|---|---|
| seg1 按下回车…切成 Token | 0.50s | 4.69s | 0.9s 回车按下→1.5–3.1s 文字输入→3.4–6.6s 五个问题 Token 逐块弹出→7.4–8.8s 舞台拉远 |
| seg2 号牌→Tokenizer→编号 | 8.20s | 4.67s | 8.2s API Server 节点弹现→8.6–9.9s 请求卡飞行→10.2s Tokenizer 弹现→10.8–12.4s 编号 Chip 逐个出现 |
| seg3 调度器排好队 | 14.00s | 3.98s | 13.5s 调度器+引擎弹现→14.0–14.8s 队列三卡滑入→15.2–16.2s 扫描线+Prefill 优先→16.6–17.5s #42 出发飞向引擎 |
| seg4a 进入 Transformer…Q/K/V | 20.30s | 5.28s | 18.6–21.4s 摄影机飞入引擎内部→20.8s 标题→21.0–22.2s 编号 Chip→22.2–23.2s 嵌入表行点亮→23.3–24.1s 向量行滑出→24.0–24.6s Q/K/V 三卡 |
| seg4b 和当前位置以及之前的 K 比对 | 25.78s | 4.21s | 25.4–27.6s 五条注意力弧线逐条画出（含自身·金色）→26.0–28.3s 权重条生长→27.8s 掩码幕布→28.4–29.8s V 加权流入 Σ |

## 5. 再生成命令

```bash
# 1) 起本地服务（仓库根 = sglang-video/）
cd sglang-video && python3 -m http.server 8899 --bind 127.0.0.1
# 2) 浏览器打开 http://127.0.0.1:8899/g2/src/index.html
#    控制台执行 window.setT(t) 可检查任意时刻画面
# 3) 逐帧渲染（示意）：对 i=0..750 执行 setT(i/25) 并截图为 /tmp/g2frames/f%05d.png
# 4) 编码+混音+封装（见 §3 命令参数）
```

## 6. 已知问题与边界

1. G2 配音为 macOS `say` 合成音（Tingting），仅作节拍验证用；正式版须重录人工/商用 TTS 并按真实音频回填节拍。
2. G2 为样条（animatic）级别：构图/节拍/摄影机已定，后续按整集分镜精修细节动画。
3. G2 时间轴为确定性（setT 驱动），与 G1 一致的设计令牌；若改 Remotion 主管线，本 HTML 时间轴可直接移植为 Remotion 组件（映射关系：setT→useCurrentFrame/25）。
4. Remotion 授权核查（方案 §3.6.1）尚未执行——本次样条改用"确定性 HTML + 浏览器渲染 + ffmpeg"纯开源管线，规避了该依赖；若切换 Remotion 须先完成授权登记。
5. Frame 3"写入层 N"标签与位置 6 格间距偏紧，G2 精修时顺带调整（方案附录 F 已记录）。
