# 《从零手搓 SGLang》动画视频课程 · 样片制作方案

> 项目：将 Datawhale 开源教程《从零手搓 SGLang》（zero-to-sglang）制作成高质量中文 2D 动画视频课程
> 本文档范围：① 参考视频实测研究；② Datawhale 原教程全章节知识映射；③ 第一集样片完整制作材料（解说稿 / 分镜表 / 素材清单 / 动画实现建议 / 质量审核）
> 本文档不含：交互式实验室、HTML 网站、整套 PPT、批量生成的后续章节
> 版本：**v3.1**（2026-11-15）· G1 三张关键帧已交付并按第一轮评审完成小修 · **G2 30 秒样条已提交，未验收**
> 完成日期：2026-11-15

---

## 0. 文档说明：信息来源与标注规范

本文档严格区分三类信息，全文统一使用以下标注：

| 标注 | 含义 | 示例 |
|---|---|---|
| 【原文】 | Datawhale 教程原文内容，附章节定位 | （原文：Part Ⅰ 第 4 章 §3） |
| 【网络补充】 | 教程之外的公开资料，附来源、版本与日期 | 【网络补充】NVIDIA A100 白皮书，2020 |
| 【教学推演】 | 为教学化简所做的推演/类比，不属于原文，需在成片中口头说明 | 【教学推演】将 GPU 画成"一间工厂" |
| 【原文计算】 | 教程由模型配置与公式推算得到的数值（**非运行实测**），画面徽章写"原文估算/计算" | 【原文计算】KV Cache 100 token ≈ 14.06（数值按 1024 进位，原文记作 MB） |
| 【参考视频实测】 | 参考视频作者真实运行得到的数字，非教程数据、非本项目数据 | 【参考视频实测】Qwen3-0.6B 词表 151,936 行 |

**研究方法声明：**

1. 参考视频 BV1jdpS6tEMQ（阿栗学长《按下回车后，AI 为了写出一句话，都干了什么？》，时长 536 秒）已**完整下载**（B 站免登录 720p 档，mp4 + m4a 合并）并**逐段实看**（切成 6 段、每段 90 秒，画面与解说音频均已核实）。本文档第一部分的所有时间轴描述均来自实际帧画面与实际解说字幕，非标题臆测。
2. 作者另两部同类型视频已抽样实看：BV1UV4e61ESj《一句话进 Transformer，都发生了什么？》（668 秒，看前 2 分 20 秒）与 BV1MjYD6gEJe《Transformer 到底在算什么？从 Token、QKV 到 KV Cache 与 FlashAttention》（2020 秒，看开头 2 分 30 秒与第 15～17.5 分钟一段），用于确认作者风格体系是否一致。
3. B 站 AI 字幕需登录才能获取（yt-dlp 实测提示 "Subtitles are only available when logged in"），该视频弹幕仅 1 条，故解说文字依据的是视频内嵌字幕条目与音频内容，时间轴精度 ±1 秒。
4. 教程原文取自 GitHub 仓库 `datawhalechina/zero-to-sglang`（git clone 因网络中断失败，改用 GitHub API 获取文件树 + raw.githubusercontent.com 逐文件下载，共 37 个中文 markdown 文件、658 KB，全部通读）。文件清单见附录 A。
5. 教程仓库许可证为 **CC BY-NC-SA 4.0**（README_zh.md 与 community/ch/PR_requirement.md 明确声明）。本方案目前**假设**课程为非商业教学用途——这是一个待确认前提，不是既定授权：若课程将用于付费课程、商业培训、带赞助发行等商业场景，须先与权利方（Datawhale）确认授权边界或取得单独许可；**对画面全部重绘并不豁免对教程文字与结构进行改编所需的许可**。无论何种用途，均须保留署名（Datawhale × RadixArk、仓库链接）并以相同方式共享（CC BY-NC-SA 4.0）。【原文：README_zh.md §许可证】

---

# 第一部分 参考视频研究（实测观看报告）

## 1.1 视频基本信息

| 项 | 内容 |
|---|---|
| 标题 | 按下回车后，AI 为了写出一句话，都干了什么？ |
| BV 号 / 时长 | BV1jdpS6tEMQ / 536 秒（8 分 56 秒） |
| 作者 | 阿栗学长（UID 3493142068202179，共 25 个视频） |
| 主题 | 自回归生成全流程：概率表 → 抽签采样 → 温度/top-p → Transformer 四站 |
| 实测模型 | Qwen3-0.6B（作者笔记本实测，视频内所有数字标注"实测"） |
| 视觉基调 | 深蓝科技底 + 点阵网格 + 透视地板；金色强调色；3D 质感卡片；卡通男孩吉祥物 |
| 验证方法 | 720p mp4 下载 → ffmpeg 切 6 段 → 逐段读帧 + 听解说 |

## 1.2 十项研究问题的总体回答

**① 整体叙事方式**：一条"问题链"驱动全片。开场抛出"这中间到底发生了什么？"，每一节结束都留一个新问题（"它到底是怎么从表里挑出那一个的？"→"那干嘛要抽？"→"这张表到底是怎么算出来的？"），问题答案又引出下一节。全片只有一条故事线（一个 prompt"用一句话形容秋天"），所有概念都挂在这条线上，从不平行铺开。结尾用"回头看这一路"把整条链重放一遍收束。

**② 开场如何引出问题**：三段式冷开（0:00–0:14）——先闪现全片最抓眼的三组画面（概率条形图、17 种分支树、"说胡话"乱码）做预告式混剪，配问句"你有没有想过，你给 AI 发出一句话，从按下回车，到屏幕上蹦出第一个字，再到它写完一整段、自己停下来，这中间到底发生了什么？"；然后压出大标题"AI 怎么写出一句话"（银色立体字）；最后落到最小可感事实："它每次只算一样东西，算的是什么？"。

**③ 每个镜头如何组织**：镜头是**连续舞台上的摄影机运动**，不是幻灯片切换。同一个"舞台"（深蓝网格地板）上，输入框、模型卡片、token 链、概率表、抽签桶、旋钮等道具长期在场；讲解某个对象时摄影机推近（放大到全屏），讲完拉远或横移到下一个对象。观众始终保有"这些东西在同一个系统里"的空间感。

**④ 动画是逐步构建还是整体切换**：**逐步构建**为主。概率表是一根柱一根柱长出来的；抽签池是 100 根签一根根落进格子的；分支树是一遍一遍分叉长出来的（1遍·1种 → 2遍·2种 → … → 20遍·17种，计数器同步跳动）。整屏切换只出现在章节级转场（约 5 次），且都用镜头快速横移/拉远带过，不是黑场硬切。

**⑤ 图形、箭头、颜色、局部放大如何解释概念**：
- **箭头**承载"数据流"：模型卡片 → 概率表（金色弧线箭头=输出整张表）；前文 → 模型卡片（弧线回环=自回归重新送入）；分支树的每条分叉线=一条可能的回答。
- **颜色编码稳定**：金色=动作/重点/选中；蓝色=中性数据/概率柱；紫色=EOS 结束符；红色=错误/风险（说胡话、循环、石头）；青色=top-p。
- **局部放大**用于"钻进去"：讲概率分布时把输入框一角放大出 151,936 网格；讲四站结构时把黑色模型卡片放大成四道彩色拱门；讲参数时把一层平面放大成数字块。
- **实测数字永远在场**：每个关键结论旁挂"实测·Qwen3-0.6B""实测·抽签池""实测：这一步抽了 12 次"徽章，数字精确到小数（78% / 3.8% / 99.997% / 16.7）。

**⑥ 复杂结构如何分层展示**：模型这个"黑盒"被拆成**四站拱门**（换成箭头 → 一层一层地推 → 比方向 → 换算成概率），每站独立成景、用编号拱门串联；"一层一层地推"再细分为"看别人（注意力）+ 自己加工（前馈网络）"两件事；层数用 28 张半透明平面叠起来的体块表达。分层逻辑始终是"先总览（四站地图）→ 再逐站细讲 → 最后拉回总览"。

**⑦ 字幕、文字标签、公式如何出现**：底部字幕全程逐句出现（白色中文，口语体）；概念词用**金色大字**砸在场景中央（Token、按概率抽、自回归、一步岔开、温度、top-p、注意力、Transformer）；公式极简，只出现两次（Softmax 定义、带温度的 Softmax），都配柱状图动画即时验证；"实测"徽章 + 数字是出现频率最高的文字元素。

**⑧ 旁白的语言风格和节奏**：完全口语化、第二人称、短句。平均每句 ≤ 18 字；大量设问与自答（"算的是什么？先玩个小游戏"）；用生活类比但立刻声明边界（"就像一个人玩故事接龙：每接一个词，都把前面的从头再读一遍"）；观众互动点明确（"打在评论区，一会儿看看"）；关键转折用否定先行（"是打字机特效吗？不是"）。语速约 4.3 字/秒，每 20–40 秒一个小结论。

**⑨ 画面和旁白如何同步**：逐句锁帧。旁白说"四十多根写着也是"的瞬间，42 根蓝色签才落位；说"前十名加在一起也只占四分之一"时，"前 10 名合计 25%"进度条才从 100% 条上切出；说"结束了"时结束符概率曲线跳到 99.997%。动画触发与关键词严格对齐，几乎没有"画面先讲完了在等旁白"的情况。

**⑩ 背景音乐、音效、转场**：全程一条低沉氛围电子底（低调、无旋律冲突，让位解说）；音效高度功能性——"咚"（大标题/大字落定）、"嗒"（token 弹出）、"唰"（镜头横移）、抽签桶的"哗啦"、错误时的低沉 buzz。转场不用花哨特效，靠摄影机运动 + 音乐微变奏实现章节感。

## 1.3 时间轴镜头分析表（24 个代表性镜头，均来自实看帧）

| # | 时间 | 画面内容 | 动画方式 | 解说内容（据内嵌字幕） | 镜头转场 |
|---|---|---|---|---|---|
| 1 | 0:00–0:05 | 概率条形图、分支树"1遍·17种"、"说胡话"乱码三个画面快闪 | 预告式混剪，每幅停留约1.5秒 | "你有没有想过，你给 AI 发出一句话" | 快切混剪 |
| 2 | 0:05–0:11 | 输入框"用一句话形容秋天。"+回车按钮；黑色模型卡片弹出金色"？"；token 链逐个弹出"秋天 是 收获 的 季节 ， 也是 生命的 季节 。" | 输入框居中，token 金色下划线逐个出现，模型卡片每次吐字时轻微跳动 | "从按下回车，到屏幕上蹦出第一个字 / 再到它写完一整段、自己停下来 / 这中间到底发生了什么？每一个字，究竟是怎么选出来的？" | 同舞台推进 |
| 3 | 0:11–0:14 | 大标题"AI 怎么写出一句话"银色立体字；吉祥物男孩挥手 | 标题带金属质感落下，配"咚"音效 | "今天，我们就把这个过程从头到尾拆一遍" | 全景定格 |
| 4 | 0:15–0:23 | 模型卡片吐出第一个 token"秋天"；旁边小问号盒子 | 卡片+1 token，问号闪烁 | "按下回车之后，AI 背后的模型并没有直接吐出一句话。它每次只算一样东西，算的是什么？" | 推近 |
| 5 | 0:24–0:41 | "小游戏"木牌落下；男孩与你（"你"头像小人）分立两侧；气泡"我想喝一杯……"填入 token 链；水/咖啡/奶茶/石头图标逐一浮现，石头灰色暗淡 | 图标带弹跳入场，石头做"不可能"摇晃 | "先玩个小游戏。我说半句你来接：我想喝一杯……打在评论区，一会儿看看。水、咖啡、奶茶都说得通，'石头'基本不可能" | 新场景 |
| 6 | 0:42–0:54 | 词表网格（旬/副局长/闺/咖啡…）镜头拉远，计数器 203→1,510→64,910→**151,936 个候选**；概率柱状图逐根生长，顶部 100% 进度条填满 | 数字滚动+柱状图逐根长出 | "它认识十五万多个字和词，每一个都是候选。每个候选它都打出一个概率，接在后面的可能性有多大。加起来，正好是百分之百" | 放大新元素 |
| 7 | 0:55–0:59 | "实测·Qwen3-0.6B"徽章盖在柱状图上 | 徽章带盖章音效落下 | "这是千问的一个开源小模型，在我笔记本上真跑出来的结果" | 同景叠徽章 |
| 8 | 0:59–1:08 | 柱子标签+百分比逐个揭示：水3.8% 咖啡3.6% 奶茶3.0% "，"2.8% 可2.3% 啤酒2.2% 茶2.0% 酒1.9% 冰1.6% 柠檬1.6%；"前 10 名合计 25%"从 100% 条上切出 | 标签逐个浮现，占比条切分动画 | "排在最前面的，正是水、咖啡、奶茶。可前十名加在一起，也只占四分之一" | 同景揭示 |
| 9 | 1:09–1:13 | 摄影机沿概率长尾向右横移：第34名→第100名→第1,000名→第10,000名→(第100,000名) | 长尾横移镜头，名次文字掠过 | "后面还拖着一条十五万长的尾巴" | 横移 |
| 10 | 1:13–1:17 | "Token"金色大字砸下；token 链首尾标"读/写" | 大字+标签 | "这些字和词，统一叫 Token。模型读的是它，写的也是它" | 同景强调 |
| 11 | 1:17–1:30 | 金色弧线箭头从模型卡片甩到概率表；"概率表"角标 | 弧线箭头+角标 | "所以模型每算一次，交出来的不是一个字，而是这样一整张表……这也是它唯一会做的事" | 箭头引导 |
| 12 | 1:30–1:41 | "挑最高的"气泡→打叉；抽签桶（白桶红"签"字）+彩色签棒飞入；"概率多大，签就多少" | 气泡打叉，签棒倾泻 | "你大概会猜：挑概率最高的那个。平时聊天，它偏不这么干。它抽签，规矩只有一条：概率多大，签就多少" | 新道具入场 |
| 13 | 1:46–2:06 | "第 1 个字的表"：78% 秋天 / 16% 秋 / 4% 金；切到逗号后：抽签池平铺 100 根 6 种——也是42根 万物19根 色彩15根 枫12根 是8根 果实4根，"实测·抽签池"徽章 | 柱状图→签池逐根落位，分组标签逐个标数 | "就拿开头那句来说……逗号之后，签筒里有一百根签，一共六种：四十多根写着'也是'" | 场景切换 |
| 14 | 2:06–2:16 | 第1次/第2次两次抽取：也是/万物；"实测：这一步抽了 12 次"，各组头顶跳出"中5次/中4次/中2次/中1次" | 签棒高亮被抽中者，计数器跳动 | "同一个问题我问了两次，第一次抽到'也是'，第二次抽到'万物'……概率大的更容易中，但不是每次都中" | 同景实验 |
| 15 | 2:16–2:36 | 抽中的"也是"接回 token 链，"前文"角标；金色弧线把整段重新送回模型卡片；新表弹出：生命的26% 变换21% 季节15% 沉8% 时光7% 自然7% 色彩5% 离4% 凉4% 思念4%；"自回归"金色大字+副标"每写一个字，都把前面的整段再算一遍" | 弧线回环+新表生长+大字 | "它把刚抽中的词接到前面所有文字的后面，再把这一整段重新送进去，算一张新的表再抽一个。就像一个人玩故事接龙……这种写法，叫自回归" | 回环构图 |
| 16 | 2:36–2:42 | 计数器"算了 9 次 / 写出 9 个 Token"；单独一根 100% 柱"这一步只有一种签"；打字机图标打叉 | 计数器滚动，打字机图标碎裂 | "是打字机特效吗？不是。它真的是算一个，出一个" | 同景否定 |
| 17 | 2:42–3:00 | 两条回答并排生长："第1次 秋天是收获的季节，**也是**生命的季节。" / "第2次 秋天是收获的季节，**万物**在凋零与绽放之间交替。"；下方左右两张表对比（接了"也是"→共10种 vs 接了"万物"→71%在 29%凋 共2种）；"一步岔开"金色大字 | 双行逐 token 生长+双表对照 | "那刚才两次只差一个词，最后会差多少？……前文变了，表就跟着变。一步岔开，后面就是另一条路" | 分屏对照 |
| 18 | 3:00–3:10 | 分支树逐遍生长：1遍·1种 → 2遍·2种 → … → 20遍·17种；"每一步有几条路 1 2 3 6 6 6 10 12 13 13 14 14 14 13 11 7 5 5 5 4 3 2"；"重新生成"按钮出现，顶部句子随机切换 | 树枝分叉动画+计数器+按钮点击 | "同一个问题，我问了它二十遍……最后是十七种不同的回答。你每点一次'重新生成'，就是让它重新走一趟" | 生长式全景 |
| 19 | 3:10–3:50 | 词表网格中一格染紫："结束符 不代表任何字，只代表'到此为止'"；结束符概率随生成进度的折线：句号前≈0%，句号后 5.5%→37.7%→98.5%→**99.997%** | 折线逐点爬升，末点爆闪 | "那它写到什么时候才停？……'说完了'这件事，也是抽出来的" | 折线特写 |
| 20 | 3:50–4:42 | "你"头像嘀咕"有点别扭…"；关掉抽签开关：20 遍·1 种，"重新生成"按钮划掉；"今天天气不错，"贪心续写绕成圆环计数 12/30/80/**120 个 Token** 还没停；四个开头卡片 2 个标"打转" | 开关拨动、圆环回旋、计数爆红 | "抽签这么飘……那干嘛要抽？……挑最高的……两个都这样：前文里已经有这句话，照着再写一遍就成了最有把握的选择" | 实验对照 |
| 21 | 4:42–5:10 | "贪心"标签定名；回到喝一杯场景，"温度"旋钮（低/高红弧）+"top-p"开关入场；柱子上冒红色"分数"点 | 旋钮弹出，分数点点亮 | "这种每一步只选第一名的做法，叫贪心……万一抽中一个离谱的呢？这要靠两个旋钮。模型其实是先给每个候选打一个分" | 道具入场 |
| 22 | 5:10–5:40 | Softmax 公式出现；温度旋钮拧动 0.3（前10名合计94%，老三样65%）→1.5（前10名合计7%，闭眼乱指）→2.0（当场说胡话："秋天里DH浸润深深的暗.In'C Baghdad awful Vampire"，写了 40 个 Token 还没停）→0.0（一根 100%："拧到零，每一步只选第一名"） | 旋钮实时改柱状形状，胡话词乱飞 | "第一个旋钮装在换算之前……温度调低，差距拉大……调高，差距抹平……一路拧到头，它当场说起了胡话" | 公式+实验 |
| 23 | 5:40–6:35 | "top-p"青色旋钮：按概率排序，累计 52.3%→76.6%→**80.2%** 处竖线切走，后 5 根重新归一化 34.7/30.7/16.0/12.2/6.5；温度1.5 加 top-p0.8 前后半通不通 vs 通顺对照；签筒砍完只剩"季节 100 根""没得抽"；大网格：**297 步里 147 步只有一个候选**（青/黄双色格），"抽着写，却不乱" | 累计动画+切割+双色网格统计 | "第二个旋钮就是干这个的……很多时候砍完只剩一种……所以它虽然是抽着写的，却不乱" | 组合实验 |
| 24 | 6:35–8:56 | 模型卡片放大成四站拱门：①换成箭头（大字典 151,936 行×1,024 列，"喝"那行实测数值，三支金色箭头"方向=意思"，引用往期《万物皆向量》）②一层一层地推（看别人=注意力+自己加工=前馈网络，28 层体块）③比方向（最后一支箭头="接下来该出现什么"，151,936 个候选箭头扇形，实测分数 水16.7 咖啡16.7 奶茶16.5 石头8.4）④换算成概率（Softmax→开头那张表）；参数揭秘（1层×15,728,896 + 字典 → **596,049,920 个参数**）；训练演示（床前明月□ 猜"山"✗ 猜"光"✓）；GPT 的 T=Transformer；三个开放问题钩子；全程回顾循环重放；"是抽出来的"金色大字收束；"凋｜零"切两块引出下期 | 拱门逐站点亮、平面叠层、扇形展开、数字块放大、循环重放 | "钻进去看，里面一共四站……这台四站的机器就叫 Transformer……回头看这一路……每一个字，是怎么选出来的？是抽出来的。不过，你打给它的字，它一个都没看见——可为什么'秋天'是一整块，'凋零'却被切成了两块？下一期，我们从第一站讲起" | 总-分-总结构 |

## 1.4 作者其他同类型视频（抽样实看记录）

| 视频 | 实看范围 | 与参考视频的风格一致性 |
|---|---|---|
| BV1UV4e61ESj《一句话进 Transformer，都发生了什么？从分词走到输出，把每个模块串起来》（668 秒） | 前 140 秒 | **同一作者、另一套视觉主题**：奶白底+米黄地面+桌面纹理，token 白瓷块配彩色描边。沿用同一套方法论：真实词表编号（"手机"=58405，词表 151,669 个 Token）、嵌入表实测尺寸（151,936 行 × 1024 列）、QKV 投影矩阵（128 行 × 1024 列）、逐 token 点积实测（46.28 / 57.30 / 46.87 / 52.26…）、Softmax 曲线 + 权重求和=1，并出现**论文引用卡片**（Attention Is All You Need, Vaswani et al. 2017, §3.2.1 Footnote 4）。确认"实测数字 + 论文背书 + 逐步推导"是其贯穿风格 |
| BV1MjYD6gEJe《Transformer 到底在算什么？从 Token、QKV 到 KV Cache 与 FlashAttention》（2020 秒） | 0–150 秒、900–1050 秒 | 第三套主题：奶油色房间+吉祥物书桌+**顶部管线轨道**（Tokenizer → 内部 ID 序列 → E+位置 → PreNorm 各层 → 最终 LN·f → 词表投影 → 概率 → 选择），讲解时高亮当前站。最突出的是**认识论自律**：画面角落常驻"概念示意，未运行具体分词器；无真实 ID、语义距离矩阵""轮廓示意，不指定词例""本期基础生成结构·数字符号示意"等免责角标；KV Cache 部分用"第一行改变；第二行保持"表格逐行展示新旧权重（旧:全可见 [0.330238, 0.669762] → 新:因果 [1,0]）；残差用 10+(−2)=8 的整数例子；LayerNorm 逐行推导 μ/σ；FFN 用 W_eff=W₁W₂ 合并证明"无非线性则多层塌缩成一层"。这套"符号概览 + 概念/机制分离 + 常驻免责标注"的方法对本项目有直接参考价值 |

**结论**：参考作者的可复制方法论可归纳为七条——①单线故事+问题链；②连续舞台+摄影机运动；③逐步构建动画；④实测数字徽章；⑤金色大字概念锚点；⑥总-分-总与片尾循环回顾；⑦概念示意/实测结果的显式区分。我们的样片将继承这七条，但**选题差异化**：参考视频只讲"模型内部如何选出下一个字"，完全不涉及服务端；第一集样片的核心增量正是 zero-to-sglang 的立身之本——**推理引擎视角**（Tokenizer、调度器、Prefill/Decode 分工、KV Cache 的工程意义、流式返回），参考视频恰好是其前置知识。

---

## 1.5 可迁移制作规范（本项目直接采纳）

1. **一集一条故事线**：全片围绕"一个请求"从回车到流式返回的旅程，所有概念挂在这条线上。
2. **连续舞台**：浏览器窗口（用户侧）+ 服务端流水线舞台共享一个 3D 空间，摄影机推拉横移切换视角，不做黑场硬切。
3. **徽章体系（区分数据性质）**：数字徽章分三类——**【原文计算】**（教程由配置/公式推算，如 Qwen3-4B KV Cache 14.06）、**【参考视频实测】**（参考作者真实运行，如 Qwen3-0.6B 词表 151,936）、**【本项目实测】**（仅当我们真实运行并记录后才可使用）。严禁把公式推算值标成"实测"。
4. **概念示意显式化**：动画中的小模型、小词表、小批处理一律带"教学示意"角标，与真实系统区分（学参考作者 KV 视频的免责角标做法）。
5. **金色大字锚点**：Prefill、Decode、KV Cache、自回归、Top-p、EOS 等术语首次定义时用大字+短句定义，之后只复用图形。
6. **公式最小化**：样片只出现 3 个公式（Softmax+温度、KV Cache 显存公式、注意力得分形式），全部配即时动画验证。
7. **片尾循环回顾**：结尾把整条流水线重放一遍（约 30 秒），每个环节闪现一次核心结论。
8. **原创性边界（风格迁移只借"语法"，不借"资产"）**：借鉴叙事方法与动画语法（连续舞台、问题链、逐步构建、镜头推拉、徽章体系、免责角标），**不照搬**其场景美术、吉祥物形象、道具外形、字体组合、音效素材或镜头编排。本项目建立自有视觉识别：颜色语义体系、token 形状与图标库、字体、背景空间、音效语义。可保留"深色科技底 + 金色强调"这一抽象方向，但资产全部原创；如启用吉祥物，须为全新原创形象，不与参考作者形象存在混淆可能。
---

# 第二部分 Datawhale 原教程完整分析与知识映射

## 2.0 课程总体结构（据 README_zh.md）

《从零手搓 SGLang》由 Datawhale × RadixArk（SGLang 团队创立的公司）共同发起，编写始于 2026-08-24，截至 2026-11-10 全部章节中英文完成；**官方配套讲解视频预计 2026 年 11 月底发布**——本项目即属于该配套视频方向的制作方案。课程定位（README 原文概括）：

- Part 0 开课之前（编码伦理 + 部署第一个 SGLang 服务）
- Part Ⅰ 基础概念（无代码、无 GPU）：LLM / 推理 / GPU / KV Cache / Benchmark
- Part Ⅱ 从零手搓 mini-sglang（10 章，概念+代码走读，参考官方 mini-sglang 提交 `9a91cfa`）
- Part Ⅲ 高级推理技术（深入真实 SGLang，SGLang 官方成员编写）：注意力后端 / 量化 / 分层缓存 / 横向扩展 / PD 分离
- Part Ⅳ 如何为 SGLang 做贡献：Cookbook 部署 / Profiling 与 Trace / PR 工作流
- Community 社区专区（手搓记录 / 踩坑 / 学习笔记，含 llm-engine-viz 可视化笔记）

课程骨架是一条**问题驱动链**（Part Ⅱ 第 1 章原文）：

> 先读懂请求如何流转 → 保证生成正确 → 发现历史计算重复 → KV Cache → 无法服务用户 → HTTP/Streaming → 多请求互相等待 → Continuous Batching → 缓存占用与碎片 → Paged KV Cache → 共享前缀 → Prefix Caching → 单卡不足 → Tensor Parallelism → Decode 逐 token 受限 → Speculative Decoding

这条链同时就是视频课程的**分集骨架**（见 2.26 分集映射总表）。

---

## 2.1 Part 0

### 2.1.1 Part 0.1 编码伦理与开源精神

1. **学习目标**：在技术教学之前建立开源协作伦理——对自己代码负责、沟通说人话、性能工作从 Profile 开始、理解开源精神。
2. **原文知识点清单**：AI 时代代码产出与审查成本的不对称；提 PR 前自查四问（motivation/成果复现/侵入性/单函数理解）；"AI slop" 式回复的反例与正例对照；Profile 永远是第一步（profile→找瓶颈→改→再 profile→对比）；性能 PR 三要素（前后对比数据、可复现命令、Profile 证据）；五种不受欢迎的第一个 PR；好 PR 标准（小、完整、解决 existing issue、有验证）；开源精神（我为人人、人人为我；贡献不只是代码）。
3. **核心概念**：射箭画靶（优化只占 3% 耗时的部分）；隐性工作量转移（写 500 行 30 秒，审 500 行大量时间）。
4. **关键技术流程**：正确的性能优化顺序：先测占比 → 改 → 复测对比。
5. **公式参数**：无。
6. **代码实现**：无（本章为理念章）。
7. **难点与易误解**：易误以为"开源反对用 AI 写代码"——原文明确不反对，反对的是提交者没读过自己提交的代码；易把"刷 PR 数量"当贡献目标。
8. **动画表达形式**：对话框对照动画（AI slop 长文 vs 一句话实测回复）；时间条动画演示"3% 部分优化到零端到端只快 3%"。建议做成 1 集番外或课程片头 90 秒。
9. **推荐补充资料**：SGLang Contribution Guide（docs.sglang.io）；教程 Part Ⅳ 第 3 章（PR 工作流）。

### 2.1.2 Part 0.2 部署你的第一个 SGLang 服务

1. **学习目标**：在自己的 GPU 上用 SGLang 跑起 Qwen3-0.6B（权重约 1.5GB，4GB 显存可跑）并发出第一个请求。
2. **原文知识点清单**：环境要求（NVIDIA ≥4GB 显存、Linux/WSL2、不支持 macOS 原生）；`nvidia-smi` 验证 CUDA Version ≥12.x；conda/venv 独立环境；按 CUDA 版本两条安装路径（CUDA 13 默认 / CUDA 12 三条 force-reinstall）；国内加速（HF_ENDPOINT=hf-mirror 或 SGLANG_USE_MODELSCOPE）；启动命令 `python3 -m sglang.launch_server --model-path Qwen/Qwen3-0.6B --host 0.0.0.0 --port 30000`；成功标志日志 `The server is fired up and ready to roll!`；curl 发 `/v1/chat/completions`；openai 兼容 Python 客户端；Qwen3 默认输出 `<think>...</think>` 不是 bug，可关 `enable_thinking` 或加 `--reasoning-parser qwen3`；常见问题（OOM 用 `--mem-fraction-static 0.6 --context-length 8192`；老卡换 `--attention-backend triton`；端口占用换端口等）。
3. **核心概念**：SGLang 与 OpenAI API 兼容接口；thinking 模式开关；显存占比参数。
4. **关键技术流程**：装环境 → 启动 server → health 检查 → 发请求 → 读回答。
5. **公式参数**：无；命令参数见上。
6. **代码实现**：无源码走读；给出启动命令与 curl/Python 调用（原文完整给出）。
7. **难点与易误解**：export 只对当前终端生效；WSL2 驱动装在 Windows 侧；`address already in use` 换端口；CUDA 驱动版本与安装包不匹配的报错对照。
8. **动画表达形式**：终端录屏 + 命令逐条高亮；建议做 3–4 分钟"动手番外"，与正片分开。
9. **推荐补充资料**：SGLang Cookbook（docs.sglang.io/cookbook）；官方安装文档（docs.sglang.io/get_started/install）。

---

## 2.2 Part Ⅰ 基础概念

### 2.2.1 第 1 章 LLM 入门

1. **学习目标**：说清 LLM 定义与发展脉络与边界；了解典型应用；掌握 Transformer 核心组件；熟悉贯穿全课程的基础概念。
2. **原文知识点清单**：LLM 本质（给定文本预测下一个 token 的概率模型，规模涌现能力）；2017 Transformer → 2018–19 BERT/GPT → 2020 GPT-3 → 2022 至今 ChatGPT 与开源模型演进（RoPE、GQA/MLA、RMSNorm、SwiGLU）；能力边界（幻觉、知识截止、上下文有限、缺真因果推理）；应用（对话问答/文本生成/代码工具）及其共同依赖——推理成本与延迟；Transformer Block 结构（编码器/解码器堆叠）；位置编码（正余弦公式与多尺度设计）；多头注意力（单头局限→多头切分→拼接与输出投影；原论文参数 d_model=512、h=8、d_k=64、√d_k=8）；LayerNorm 与残差（公式、Post-Norm 顺序 `LayerNorm(x+Sublayer(x))`、Pre-Norm 对比）；FFN 与激活函数（`FFN(x)=max(0,xW₁+b₁)W₂+b₂`，d_ff=2048，ReLU；合格激活函数三特质：非线性/可微/高效）；SOTA 对照表（Decoder-only/MoE、RoPE、MQA→GQA→MLA→NSA/DSA、RMSNorm、Pre-Norm、GLU 家族、PagedAttention、FP16→FP8/INT4 等十条演进线）；关键概念（Prompt/上下文工程/Skill/RAG/Agent/Harness；Token——deepseek 一个 token 约等于 1.6–1.7 个汉字；Embedding；参数；自回归生成；上下文窗口——主流旗舰约 1M，约 1.5–2 本《红楼梦》）。
3. **核心概念**：预测下一个 token；自注意力；残差+归一化使深层可训练。
4. **关键技术流程**：文本→token→嵌入+位置编码→N 层 Block（注意力跨 token 通信 + FFN 单 token 加工）→输出。
5. **数学公式**：正余弦位置编码 `PE(pos,2i)=sin(pos/10000^(2i/d_model))`、`PE(pos,2i+1)=cos(...)`；`Attention(Q,K,V)=softmax(QKᵀ/√d_k)V`；`LayerNorm(v)=γ(v−μ)/σ+β`；`Output=Input+Layer(Input)`；FFN 公式。
6. **代码实现**：仅多头切分的 reshape/transpose 两行示意（原文 §4.2.3）。
7. **难点与易误解**：多头"分工"只是举例，无法确定每个头的实际分工（原文明确声明）；残差是学 F(x)=H(x)−x 而非直接学 H(x)；激活函数不可缺非线性（否则多层退化为单层线性）；LLM 是概率模型故有幻觉。
8. **动画表达形式**：Token 链→嵌入向量→箭头空间（语义聚类）→注意力弧线→层堆叠；SOTA 对照表做成"进化树"一闪而过。**本章节是第一集样片 3.10 节（Transformer 结构速览）的原文依据。**
9. **推荐补充资料**：原文列出 Attention Is All You Need（arXiv:1706.03762）、Diy-LLM 第十三章。

### 2.2.2 第 2 章 推理入门

1. **学习目标**：区分训练与推理；掌握一个 token 的完整生命周期；理解 Prefill/Decode 分工；会用 Roofline model 判断 compute-bound / memory-bound。
2. **原文知识点清单**：推理=用已训练模型做前向计算；训练并行优化参数、推理串行逐步输出（步间串行、步内并行）；8B 模型成本对照（训练一次 6ND≈7.7×10²¹ FLOPs ≈ 90 张 H100 一天；推理一次 2N×(512+128)≈1×10¹³ FLOPs，百万次/天月成本数万美元）；训练与推理协作闭环（推理是训练的组成部分；架构决定推理上限：70B FP16 权重 140GB；GQA/量化需训练时设计）；**token 生命周期四步**：请求调度→Prefill（并行处理所有输入 token、构建并写入初始 KV Cache、采样出第一个输出 token）→Decode（每次只输入上一个生成的 token、追加 KV、直到 EOS 或最大长度）→输出返回；LLaMA-7B Prefill 实例（N=3000：单层 24Nd²+4N²d，32 层 ≈43.4 TFLOPs；权重 14GB+KV 写入 1.5GB，算术强度高→compute 倾向）；Decode 实例（每 token 14.4 GFLOPs 但需读 15.5GB→理论 9.7ms，memory-bound；KV 随生成增长 1.5→2.5GB）；Roofline model（横轴算术强度 AI=FLOPs/Bytes，纵轴性能；拐点=峰值算力/带宽，A100=312/1.6≈195 FLOP/Byte；GEMM AI=MKN/(MK+KN+MN)；Prefill AI≈100、Decode AI≈1）；场景分析（Agent 协作：重复 Prefill+上下文膨胀+跨 Agent 通信；多轮对话：TTFT 随轮次线性增长 200ms→2000ms，KV Cache 数十 GB）。
3. **核心概念**：Prefill=一次完整前向+建缓存+出首 token；Decode=逐 token+读缓存+追加缓存；步内并行步间串行。
4. **关键技术流程**：token 生命周期四步；Roofline 三步判断法（查硬件参数→算 AI→对比拐点）。
5. **数学公式**：`P(token_i | token_1..i-1)`；`6ND` 训练计算量；`2N(len_in+len_out)` 推理计算量；Prefill/Decode FLOPs 与 Bytes 公式（原文 §5.2 给出完整推导）；`拐点=峰值算力/HBM带宽`。
6. **代码实现**：无。
7. **难点与易误解**：Prefill 在 batch=1、N=100 时仍是 memory-bound（AI≈100<195），只是可通过增大 batch/序列推向 compute-bound（原文明确"虽然仍是 memory-bound，但 AI 值较高"）；Decode 的 AI 趋近常数 1 与序列长度几乎无关；"步间串行"与"步内并行"并存。
8. **动画表达形式**：**token 生命周期四步是第一集样片的主干地图**（原文图 2-3 直接对应样片 3.2 节全景图）；Roofline 折线图动画；Prefill/Decode 双 lane 时间线对比。
9. **推荐补充资料**：NVIDIA H100/A100 数据表；LMSYS 大规模 EP 博文；Diy-LLM 第 9/10 章。

### 2.2.3 第 3 章 GPU 入门

1. **学习目标**：GPU 架构基础（GPC→TPC→SM→CUDA/Tensor Core）；SIMT 执行模型（Block/Warp/Thread 与内存层次）；LLM 推理在 GPU 上的执行流程与两阶段瓶颈。
2. **原文知识点清单**：GPU 起源（1981 CGA→1985 ATi→1994 Glint300SX→1999 GeForce 256 命名 GPU→2001 可编程着色器→2006 G80 统一架构+CUDA→2012 AlexNet）；CPU vs GPU 设计哲学（低延迟 vs 高吞吐；A100 6912 个 FP32 CUDA 核 vs CPU 4–64 核）；A100 PCIe 80GB（GA100 542 亿晶体管 7nm 826mm²，产品屏蔽 1 GPC+2 TPC 得 108 SM；7 GPC→54 TPC→108 SM→6912 CUDA core+432 Tensor Core+5 HBM2 堆栈；双槽 267mm 300W 被动散热）；Tensor Core（TF32/FP16-BF16 312 TFLOPS 稠密/624 稀疏，2:4 结构化稀疏=每 4 个数至少 2 个 0）；SIMT 执行（Block→SM 一对一映射；Warp=32 线程公交车型同步；Thread 私有寄存器 ≤255 个；Warp Divergence 分支串行化损失约一半；SM 内 4 条指令流水线、192KB L1/共享内存）；内存层次（全局内存 HBM2e 80GB ~2TB/s 290 周期→L2 40MB ~5TB/s 200 周期→L1/共享 192KB/SM 20–33 周期→寄存器 256KB/SM 零延迟；分层的物理原因：光速与片上/片外距离）；GPU 与 CPU 内存本质区别（吞吐 vs 延迟）；LLM 推理离不开 GPU 四条理由（海量 GEMM、显存带宽 ~2TB/s、Warp 切换隐藏延迟、CUDA 生态）；GPU 执行推理流程（CPU 分词→PCIe 传入→GEMM 分解 Tiling→Block→Warp→Thread→共享内存→寄存器→Tensor Core→写回）；Prefill=GEMM 计算密集用 Tensor Core，Decode=GEMV 访存密集瓶颈在 HBM（对比表）。
3. **核心概念**：吞吐换延迟；SIMT；内存墙。
4. **关键技术流程**：矩阵乘在 GPU 上的任务分解与数据流动（全局内存→共享内存→寄存器→计算→写回）。
5. **公式参数**：A100 各精度算力表（INT8 624 TOPS / FP16 312 TFLOPS / TF32 156 / FP64 19.5，稀疏翻倍）。
6. **代码实现**：无。
7. **难点与易误解**：GPU 不是"更快"而是"更宽"；共享内存是手动控制的；L2 是所有 SM 共享的总枢纽；2:4 稀疏要求固定模式不是随机零。
8. **动画表达形式**：芯片解剖分层动画（板卡→裸片→GPC→SM→核心）；Warp 公交车比喻（需标【教学推演】）；内存金字塔与延迟数字标注。建议做独立 1 集（第 5 集左右），样片第一集不展开 GPU。
9. **推荐补充资料**：NVIDIA Ampere 白皮书；A100 数据手册；斯坦福 CS336；Stas Bekman 拆机记录。

### 2.2.4 第 4 章 推理的核心数据结构：KV Cache

1. **学习目标**：从 Attention 公式推导为什么 Decode 需要 KV Cache；描述 Cache 生命周期；按模型配置估算显存占用；解释 naive 管理的问题。
2. **原文知识点清单**：Q/K/V 语义（Q=当前 token 想获取什么，K=各 token 的被匹配特征，V=实际携带的信息）；Prefill 一次性计算并缓存整个输入序列的 K/V、**Q 无需缓存**（仅用于当前一次 Attention）；Decode 每步新 token 的 K/V 拼接到历史后、新 Q 与全部 K 做因果自注意力；按层保存 `[(K⁽¹⁾,V⁽¹⁾),...,(K⁽ᴸ⁾,V⁽ᴸ⁾)]`；`K_{1:t+1}=concat(K_{1:t},k_{t+1})`；极简数值实例（词表 10、4 维、单层单头、W=I："I love AI" 的 3×3 因果注意力矩阵逐行 softmax，预测 "very" 0.63 贪心选中；Decode 输入是 "very" 的 embedding 而非上一步的 h₃）；生命周期四阶段（分配→Prefill 批量写入→Decode 反复读取并追加→释放回空闲池或保留待复用；EOS/最大长度/取消时结束）；显存公式 `KV_bytes=B×T×L×2×n_kv×d_h×s`（Prefill）与 Decode 追加/总量式；Qwen3-4B-Thinking-2507 实例（L=36, n_kv=8, d_h=128, bf16：T=100 时 14.06MB；每 token +0.141MB；100+200 总量 42.19MB；4K→32K 单请求 0.56→4.50GB；8 个 32K 请求 ≈36GB，权重仅 7.45GB）；长上下文时 KV 超过权重成为显存主导；激活值是临时工作区而 KV Cache 是长期驻留的历史状态；naive 连续分配四问题（碎片与预留浪费、无法共享重复前缀、调度回收复杂、不利于 Continuous Batching）。
3. **核心概念**：缓存的是每层投影后的 K 与 V，不是 hidden state、不是 attention 矩阵、不是 logits、不是只缓存最后一层。
4. **关键技术流程**：`新 token id → embedding → q,k,v → 追加 k,v → 用 q 读历史 KV → 预测下一个 token`。
5. **数学公式**：`q_t=h_tW_q` 等；`o_{t+1}=softmax(q_{t+1}K_{1:t+1}ᵀ/√d_h)V_{1:t+1}`；显存公式如上。
6. **代码实现**：无（概念章；工程实现见 Part Ⅱ 第 4 章）。
7. **难点与易误解**：Prefill 的最后一个位置产出首 token 的 logits（Prefill 完成两件事：建缓存+出首 token）；Decode 送入的是新 token 的 embedding 不是上一步隐藏状态；真实系统中追加"并不总是对应一块连续增长的显存"（分页/共享/回滚）。
8. **动画表达形式**：**第一集样片 3.5/3.6 节的原文依据**——K/V 逐 token 落入分层缓存格、Q 用完即弃的对比动画；显存条随生成长度线性增长；多请求共享前缀的重复浪费演示。
9. **推荐补充资料**：HiCache 博文（lmsys 2025-09-10）；Unified Radix Cache 博文（lmsys 2026-08-11）；Qwen3-4B-Thinking-2507 config.json。

### 2.2.5 第 5 章 Benchmark 入门

1. **学习目标**：区分模型能力 Benchmark 与推理负载 Benchmark；掌握延迟/吞吐指标体系；会设计可复现测试并正确解读。
2. **原文知识点清单**：两类 benchmark 的区分（能力：MMLU/GSM8K/HumanEval/SWE-bench；负载：TTFT/TPOT/吞吐/P99）；benchmark 三作用（性能边界、量化优化效果、容量规划）；延迟四指标（TTFT 含排队+Prefill；TPOT 均值；ITL 逐次可暴露抖动；E2E）；实例（输入 1000 生成 200：TTFT 250ms+TPOT 25ms→E2E≈5.2s，长输出优化 TPOT 收益大、短输出长输入优化 TTFT）；速率参考（25ms/token=40 token/s；行业下限 100ms/token≈10 token/s）；吞吐两指标（Throughput；Goodput=满足延迟约束的吞吐）；吞吐与延迟此消彼长；场景-指标对应表（对话→TTFT；长文→TPOT；离线→吞吐；语音/Agent→TTFT+尾延迟）；SGLang 四层 benchmark 工具（bench_serving / bench_one_batch_server / bench_offline_throughput / bench_one_batch 及各自覆盖层次）；bench_serving 四种请求集（sharegpt/random/generated-shared-prefix/agentic-trace）；运行参数固定（并发/速率、硬件模型精度、预热、输出长度分布）；解读三原则（先 SLA 后吞吐；看分布不看均值——P50 202ms/均值 246ms/P99 862ms 实例；逐项核对条件）；九条常见误区（只看吞吐、只看均值、未预热、把 bench_one_batch 当线上、长度分布不同、跨模型比 token/s 忽略分词器、不看 goodput、单机结论外推多机、未固定随机种子致 5% 波动）。
3. **核心概念**：Goodput；P99 尾延迟；可比性。
4. **关键技术流程**：设计 benchmark = 定评测对象 → 选对应数据集 → 固定运行参数 → 按指标体系解读。
5. **公式参数**：`剩余生成时间=TPOT×(len_out−1)`；`E2E=TTFT+剩余生成时间`。
6. **代码实现**：工具命令表（原文 §4.1 表格）。
7. **难点与易误解**：TTFT 高≠Prefill 慢（可能排队）；吞吐高≠服务好（P99 可能已恶化）；bench_one_batch 绕过调度器数值显著偏高。
8. **动画表达形式**：请求时间轴标注动画（TTFT/TPOT/ITL/E2E 四段标尺）；P50/P99 分布直方图。适合做 1 集；样片第一集仅在流式返回处口头提"TTFT"一次。
9. **推荐补充资料**：SGLang Benchmark 文档；NN/g Response Times；LLM benchmark 综述 arXiv:2508.15361。

---

## 2.3 Part Ⅱ 从零手搓 mini-sglang（概念+代码走读成对，按单元映射）

> 说明：Part Ⅱ 每章由"概念篇"与"代码走读篇"两个文件组成，映射按单元合并；代码走读篇的要点单独列在"代码实现"栏。全部代码引用以官方 mini-sglang 提交 `9a91cfafe754aa85daee49998176275667eb58f2`（2026-09-16）为基线【原文：Part Ⅱ 第 1 章 §3.3】。

### 2.3.1 第 1 章 mini-sglang：推理引擎长什么样

1. **学习目标**：区分"模型调用"与"推理引擎"；画出最小引擎模块与数据流；描述请求状态；理解 `Req/Batch/Context` 对象；理解 Part Ⅱ 的增量实现顺序。
2. **原文知识点清单**：朴素自回归循环五步（encode→forward→sample→append→decode）及其两处不足（重复计算整段历史、无法服务多用户）；推理引擎四项核心职责（输入输出管理 / 请求编排 / 计算调度 / 状态与内存管理）；**教学四层**（接口层/编排层/执行层/状态与内存层）与**进程视图**（API Server、Tokenizer Worker、Detokenizer Worker、Scheduler Worker、Engine）；控制路径 vs 数据路径（控制传请求状态，数据传 input_ids/位置/logits/KV）；**简化状态机** `WAITING→PREFILL→DECODE→FINISHED`（另有 ABORTED；教学抽象，不表示源码同名状态）；八步请求示例（"请解释什么是 KV Cache？" max_new_tokens=8）；源码目录索引（attention/benchmark/distributed/engine/kernel/kvcache/layers/llm/message/models/moe/scheduler/server/tokenizer/utils/core.py/__main__.py）；四个核心对象（`SamplingParams`：温度/top_k/top_p/ignore_eos/max_tokens；`Req`：input_ids、当前序列长度、**已缓存长度 cached_len**（与前者是不同概念，`extend_len` 是二者之差）、uid、采样参数、缓存句柄；`Batch`：承接调度结果、含 phase=prefill/decode；`Context`：KV Cache 池+Attention 后端+当前 Batch）；实现路线表（第 2→10 章各自"当前限制→新增能力→验证指标"）。
3. **核心概念**："模型负责完成神经网络计算，推理引擎负责组织这些计算"；决定"为谁算、何时算、算哪些 token、算完如何保存"。
4. **关键技术流程**：八步请求流转（见样片 3.2 全景图）；`Context→Batch→多个 Req→SamplingParams+CacheHandle` 包含关系。
5. **公式参数**：无。
6. **代码实现**：`python/minisgl/` 目录树与 core.py 四对象（原文给出结构示意）。
7. **难点与易误解**：逻辑分层≠进程一一对应；scheduler 与 engine 在 mini-sglang 中同进程（函数调用），模块边界不等于进程边界（第 2 章原文重申）；"能调用模型"只是起点。
8. **动画表达形式**：**四层架构图+进程视图+状态机三联画**；是视频课程第 6 集（引擎总览）的原材料；样片第一集 3.2 节的全景流水线图即取自本章图 1.3（控制/数据路径）并简化为五站。
9. **推荐补充资料**：mini-sglang 仓库 structures.md；core.py。

### 2.3.2 第 2 章 一个请求的旅程（概念+代码）

1. **学习目标**：画出请求从进入到返回的完整链路并说明各组件输入输出；区分调度器与计算引擎及 prefill/decode 位置；解释流式输出为何需保留解码状态；说明多进程设计的利弊。
2. **原文知识点清单**：五环节（接收请求→分词→调度与计算→反分词→返回结果；五环节≠五进程）；API Server 前台职责（收 HTTP、读输入与采样参数、分配内部唯一标识 uid、保留响应状态）；tokenizer（文本→token id；多轮对话先过 chat template；只改表示）；scheduler 排班（等待队列；决定执行哪些请求、组成怎样的 batch、显存能否容纳 KV Cache；**scheduler 决定"算谁"，engine 负责"怎么算"**；mini-sglang 中两者同进程）；logits 与采样（greedy 直接取最高分；带温度的随机采样先调整 logits 再 softmax 转概率再抽取——注意原文此处写"带温度的随机采样先调整 logits"）；detokenizer（不能逐 token 单独解码拼接：有的 token 只含一个字符的部分字节，会出现替代字符或不完整文本；需保留每条请求的解码状态，只放行完整新增部分）；流式返回（SSE 分块；优化 TTFT 与用户体验；非流式则等全部生成完）；结束与取消（达上限/满足结束条件→停止；用户断开→引擎取消请求回收资源，"避免继续执行已经无人接收的计算"）；多进程三收益（重叠工作规避短板原理——tokenizer 处理 B 时 scheduler 同时推进 A；多卡协同组织；按瓶颈扩容组件——mini-sglang 支持配置 tokenizer 数量、SGLang 多 tokenizer 模式）与成本（ZMQ 消息传递；故障恢复仍需协调；SGLang 常规单 tokenizer 模式：HTTP Server+TokenizerManager 同进程，scheduler 与 detokenizer 是子进程）。
   代码走读要点：进程名（`minisgl-TP0-scheduler` / `minisgl-tokenizer-0` / `minisgl-detokenizer-0` / 主进程 FastAPI）；消息类型 `TokenizeMsg / UserMsg / DetokenizeMsg / UserReply / AbortMsg / AbortBackendMsg`；`server/launch.py` 用 multiprocessing 拉起子进程；`api_server.py`：`uid = state.new_user()` 发号、`TokenizeMsg`、`StreamingResponse`、`listen` 协程按 uid 存结果并 `event_map[msg.uid].set()` 唤醒；Ctrl+C → `AbortMsg` → tokenizer 转 `AbortBackendMsg` → scheduler 取消；`tokenizer/server.py`：tokenize 后装 `UserMsg` 发给 scheduler；`scheduler/scheduler.py` `normal_loop` 四步（收消息→`_schedule_next_batch` 组批【先 prefill 后 decode】→`_forward`→`_process_last_data`）——组批不只是取请求，还要检查 KV Cache 放不放得下、有无已缓存前缀可复用、token 预算；结束判断 `finished = not req.can_decode` 或 `next_token == eos_token_id`（`ignore_eos` 时不因 EOS 停止）；`engine/engine.py` `forward_batch`：`logits = self.model.forward()` → `next_tokens = self.sampler.sample(logits, args)` → 拷回 CPU；`sample.py` 两条路径（`temperatures is None` → `torch.argmax`；否则 `sampling.softmax(logits.float(), temperatures)` + `sampling_from_probs`，sampling 指 `flashinfer.sampling`，top-k/top-p 在内部分支省略）；**greedy 不需要真的算 softmax**（不改变大小顺序）；`tokenizer/detokenize.py`：`read_str[len(surr_str):]`，未拼完时 `find_printable_text` 只放行安全部分；对照真实 SGLang 文件表（entrypoints/http_server.py、managers/tokenizer_manager.py、managers/scheduler.py `event_loop_normal/event_loop_overlap`、managers/tp_worker.py + model_executor/model_runner.py、managers/detokenizer_manager.py，站间通信均为 ZMQ）。
3. **核心概念**：uid 串联始终；scheduler/engine 分工；detokenizer 状态机。
4. **关键技术流程**：请求五环节 × 三次表示转换（文本→token ids→新 token→增量文字）。
5. **公式参数**：无。
6. **代码实现**：如上（函数名均出自原文）。
7. **难点与易误解**：生成循环从 scheduler 组批开始而非 API Server；流式返回是体验优化不是性能优化；非流式请求期间其他请求仍可推进。
8. **动画表达形式**：**第一集样片的主地图**（3.2 节五站流水线即本章图 2-1 的教学简化）；取消路径适合做番外；"半个字节"问题适合做 15 秒特写。
9. **推荐补充资料**：mini-sglang structures.md / launch.py / scheduler.py；SGLang entrypoints/engine.py。

### 2.3.3 第 3 章 前向与生成（概念+代码）

1. **学习目标**：说清 token 从输入表示到 Forward Pass 产生 logits 再到采样变成下一个 token 的完整链路；区分 Forward/Generation/Prefill/Decode/KV Cache 各自解决的问题。
2. **原文知识点清单**：tokenize 三步（"我爱datawhale"→["我","爱","datawhale"]→[123,456,789]）；常见方法 BPE/WordPiece/SentencePiece；**tokenizer 只做映射**，不理解语义不参与计算；**embedding 是查找表 `[vocab_size, d_model]`，id=123 取第 123 行**；此时是可学习的静态向量不含上下文；`[B,T,d_model]`；位置编码（"猫追狗"/"狗追猫"顺序问题；RoPE 在 Attention 里对 Q/K 做旋转，现代主流）；Attention=第一个让 token 交换信息的站（Q=我在找什么 / K=我有什么 / V=我实际传递什么；多头拆分各头独立再拼回；**Causal Mask 把未来位置分数设为负无穷**）；Transformer Block 两件事（Attention 跨 token 通信 + MLP 单 token 消化；Pre-Norm 现代标配；残差让梯度直通）；N 层后向量高度上下文化+final norm；**LM Head=最顶部线性层，把隐藏状态映射到词表空间得 logits**；logits 是未归一化原始分数（训练用所有位置算交叉熵，推理只取最后一个位置）；结构变体表（MoE/MQA-GQA/RMSNorm/Pre-Norm/RoPE/FlashAttention——不改变 Forward 本质）；为什么需要采样（语言生成开放、多解、概率性；相同提示词不同结果可能就是采样随机性）；Greedy（确定性，适合事实问答，易重复死板）；**温度**（`p_i=exp(logit_i/T)/Σexp(logit_j/T)`；T=1 原始 / T<1 尖锐 / T>1 平坦 / T→0 贪心 / T→∞ 均匀）；**Top-k 与 Top-p 都是截断采样**（保留合理候选→截掉长尾→重新归一化→再采样；top-p 候选数由分布形状决定）；策略对照表（Greedy≈top-k=1 或 T→0；k=词表大小等于不截断；p=1 不截断）；自回归循环定义与停止条件（**采到 EOS / 达到 max_new_tokens / 达到模型最大上下文长度**；另提命中停止字符串、结构化输出完成、外部中断）；Prefill 三件事（并行处理完整提示词；建立 KV Cache；产出第一个新 token；决定 TTFT）；Decode 特征（每步一个新 token；串行循环六步；决定 TPOT）；核心区别（输入长度与并行性；Prefill 计算密集瓶颈算力、Decode 显存带宽密集瓶颈搬运）；概念总结表（Hidden State/Causal Mask/MLP/LM Head/Logits/Softmax/采样/停止条件/EOS/max_new_tokens/最大上下文/TTFT/TPOT/ITL）。
   代码走读要点（Qwen3 实现）：`Qwen3Model.forward`：embed→循环 layers→`self.norm.forward(x, residual)`；打包输入 `[N,C]`（N=本轮 token 总数）而非 `[B,T,C]`，两种布局不要混用；`Qwen3DecoderLayer.forward` 五行（input_layernorm→self_attn→post_attention_layernorm→mlp）；**残差相加没有显式 `x+attention(x)`，被放进 `RMSNormFused`**（数学顺序仍 `h=x+Attention(RMSNorm(x))`、`out=h+MLP(RMSNorm(h))`）；`RopeAttn.forward`：`qkv_proj`（合并三投影）→`attn`（内部 Q/K 各自 RMSNorm+RoPE，V 不做 RoPE）→`o_proj`；GQA 教学展开（`repeat_interleave` 仅为看清矩阵运算；`Hq` 须被 `Hkv` 整除；head_dim 独立配置，不能假设 C=Hq×D）；`GatedMLP.forward`：`gate_up_proj`→`silu(gate)*up`→`down_proj`；`ParallelLMHead.forward`：**prefill 时用 `get_last_indices` 只取每条请求最后位置做词表投影**（供采样的是每请求一行 logits，不是全部 [B,T,V]）；单请求教学版 `generate()` 循环（forward_all→`[0,-1]`→argmax→cat→eos break；会反复计算整个前缀，引出第 4 章）；`Sampler.sample` 两条路径（argmax / 温度+top_k+top_p）；形状自查表（B=1,T=3,C=32,Hq=4,Hkv=2,D=8,V=64）与三条因果性检查。
3. **核心概念**：logits≠概率；采样策略四件套；"第一个新 token 由 Prefill 产出"。
4. **关键技术流程**：完整前向链路（文本→id→查表→+位置→N 层→final norm→LM Head→logits→采样→追加）。
5. **数学公式**：温度 softmax（见上）；attention 核心在代码篇为教学简化（Q/K 归一化+RoPE 后 `q@kᵀ/√D`+下三角 mask+softmax+`@v`）。
6. **代码实现**：如上（函数名出自 mini-sglang 9a91cfa）。
7. **难点与易误解**：推理只取最后一个位置的 logits；greedy 与 argmax 等价无需 softmax；打包布局 [N,C] 与概念布局 [B,T,C] 的区别；教学简化代码不是 GPU 后端源码。
8. **动画表达形式**：**第一集样片 3.3/3.4/3.8 节的原文依据**——查表动画、QKV 三张卡、因果掩码下三角、只取最后位置高亮、logits→概率条形图、采样决策树。
9. **推荐补充资料**：Attention Is All You Need；RoFormer（arXiv:2104.09864）。

### 2.3.4 第 4 章 KV Cache 优化（概念+代码）

1. **学习目标**：解释无 Cache 的重复计算根源；说清缓存对象；推导 O(n²)/O(n³)→O(n) 收益；区分 Prefill/Decode 读写方式。
2. **原文知识点清单**：无 Cache 循环（每步输入整个历史；只关心最后一个位置的 logits；历史 token 的 K/V 反复投影）；重复量级两个口径（K/V 投影累计 Σt=O(n²)；注意力矩阵元素累计 Σt²=O(n³)；"通常标题里的 O(n²)→O(n) 指 decode 单步不再重算完整历史，当前 query 只查历史 cache，单步注意力从 T×T 变 1×T"）；小例（[x₁,x₂]→生成 3 token，注意力矩阵 2×2→3×3→4×4，x₁/x₂ 的 K/V 算了三遍）；**缓存对象五不**（不缓存 Query/原始 token/attention 矩阵/logits/只缓存最后一层——恰相反每层都要）；RoPE 时 Key 在应用旋转位置编码之后被缓存、Value 一般不做旋转；模型 forward 无状态、复用靠推理系统外部缓存；复杂度表（单步 K/V 投影 O(T)→O(1)；n 步累计 O(n²)→O(n)；单步注意力 O(T²)→O(T)；n 步累计 O(n³)→O(n²)；缓存显存 0→O(T)）；**强调"降到 O(n) 不是说整个推理都变成线性"**——注意力累计仍 O(n²)，除非滑窗/稀疏/线性注意力；含 Prefill 总复杂度（注意力 O((p+n)²)、K/V 投影 O(p+n)）。
   代码走读要点：三件事分权（保存 K/V 张量=`MHAKVCache`；记录写哪=`page_table`+`Batch.out_loc`；能否复用=`BasePrefixCache`/`CacheManager`）；`kvcache/base.py` 接口 `k_cache(index)/v_cache(index)/store_kv(k,v,out_loc,layer_id)`+device/dtype/num_layers；`mha_pool.py` `_kv_buffer` 形状 `(2, num_layers, num_pages, page_size, local_kv_heads, head_dim)`（第 0 维区分 K/V；`div_even(..., allow_replicate=True)` 算本地 KV 头数）；`k_cache/v_cache` 是 view 不复制；`free_slots=torch.arange(num_pages)*page_size` 按页记录空闲；`_page_to_token` 页起始展开为 token slot；`engine.py` 初始化 KV 池+全局 page_table（`num_pages+1` 多一页给 CUDA Graph dummy request；page_table 表项是原始 token slot 不是 page id）；`core.py Req`：`cached_len/device_len/extend_len=device_len−cached_len/complete_one()`；**`match_req` 只匹配 `input_ids[:input_len−1]`——prompt 最后一个 token 不能跳过，还要用它的 logits 采样首 token**；`PrefillAdder`：两次容量检查（lock 前后）、`estimated_len=extend_len+req.output_len`、命中 token 的 ID 写 token_pool、命中物理 slot 写 page_table、命中位置必须 lock 否则可能被淘汰重分配；`_prepare_batch`：`allocate_paged` 补页→`_make_positions`→`out_loc = page_table[input_mapping]`（当前 batch 新增 token 的写入地址列表，长度=所有请求 extend_len 之和）；`attention/fa.py forward`：**先 `store_kv(k,v,batch.out_loc,layer_id)` 写入，再 `_fa_sgl_impl` 读取**——顺序不能反；`store_kv` 用扁平视图 `[num_pages*page_size, local_kv_heads, head_dim]`+indices 写入；FlashAttention 后端把全局页表按页转换 `div_(page_size)`；请求结束 `cache_req(req, finished=True)`：insert_prefix 登记、unlock 旧句柄、free 三类位置（原本已命中的前缀/本请求新计算且成功登记的完整前缀/不能登记的尾部或已被他人登记的重复位置）；"归还不等于清零 K/V 张量"。
3. **核心概念**：缓存=每层投影后的 K/V；单步注意力 1×T；三组件分权。
4. **关键技术流程**：分配→写入（out_loc）→读取（page_table）→保留或归还。
5. **数学公式**：复杂度表（见上）。
6. **代码实现**：如上。
7. **难点与易误解**：O(n²)→O(n) 的准确含义（只指被缓存消除的重复 K/V 计算部分）；KV Buffer 第 0 维是 K/V 不是层；out_loc 不是整张页表；store_kv 先于 attention 读取。
8. **动画表达形式**：**第一集样片 3.5/3.6 节 KV Cache 专讲的原文依据**——"没有 KV Cache 时 K/V 算三遍 vs 有 Cache 时只算一遍"对比动画；out_loc 写入指针动画；第 8 集（KV Cache）可用 store_kv/fa.py 做代码走读画面。
9. **推荐补充资料**：FlashAttention 论文；mini-sglang kvcache/ 目录。

### 2.3.5 第 5 章 HTTP 服务与并发（概念+代码）

1. **学习目标**：解释客户端/服务端/HTTP 关系；读懂请求与响应；描述请求穿过推理系统的阶段；区分并发/并行/批处理；比较流式与非流式；理解结束/取消/清理。
2. **原文知识点清单**：客户端=发起请求的程序（网页/手机/命令行，服务器上的业务程序也可以是客户端）；服务端=接收并执行；HTTP 只约定消息结构，"HTTP 本身不会运行模型，也不会决定 GPU 如何执行任务"；URL 四段（协议/host/端口/路径）；请求体字段（model/messages/role/content/max_tokens——**是 token 数不是字符数也不是保证生成这么多**/stream）；响应（200/finish_reason）；**messages 是对话列表，服务端不会自动记住上次对话**；请求七阶段表（发出→API Server 接收检查→Tokenizer→Scheduler→Engine→Detokenizer→返回；按职责划分不等于每行一个进程）；并发定义（同一时间段管理多项未完成任务）；请求隔离（A1→B1→B2→A2 交错返回靠请求标识归属）；**并发/并行/批处理三区分**（能否同时管理未完成 / 是否同一时刻执行 / 能否组织进同一次模型执行；"HTTP 并发不会自动提高 GPU 利用率"）；流式 vs 非流式（流式改善首段等待体验，**不减少模型前向次数，也不保证整段更快**；非流式不等于阻塞其他请求）；SSE（data: 事件空行分隔；delta.content 增量；`[DONE]` 是接口约定不是 SSE 标准；**一个事件不一定对应一个 token 或一个网络包**）；请求结束/取消（关闭连接后端未必知道→取消通知从 HTTP 入口传到后端；协作式取消："用户点击停止"与"所有资源已完成回收"是两个不同时刻；慢客户端积压→背压）。
   代码走读要点：`OpenAICompletionRequest`（model/prompt/messages/max_tokens=16/stream）；**教学实现的兼容性声明**（token 用量是占位值、model 字段不会切换模型、n/stop 未传后端、结束原因不区分长度上限与 EOS——"接口形式相似不等于已覆盖完整协议"）；`v1_completions` 三件事（uid 发号+ack_map/event_map 创建、装 `TokenizeMsg`（含 SamplingParams：ignore_eos/max_tokens/temperature/top_k/top_p）、异步 ZMQ 发给 Tokenizer Worker）；async 不自动消除阻塞（反例 `blocking_generate` 无 await）；`FrontendManager`：`ack_map: dict[int, list[UserReply]]` + `event_map: dict[int, asyncio.Event]`（一个存数据一个做通知，不能互相替代）；`new_user()`；共享 `listen` 协程（生产者）+ 每请求 `wait_for_ack`（消费者）：wait→clear→取出列表换空列表→逐条 yield（clear 到换列表之间无 await，不会被 listen 插入）；非流式 `full_content += reply.incremental_output`；流式 `stream_chat_completions`：首块带 `role="assistant"`（角色与文本可能同块）、`cmpl-{uid}`/`text_completion.chunk`、finished 后发 finish_reason="stop" 块再发 `data: [DONE]\n\n`；`stream_with_cancellation`（is_disconnected→CancelledError→`abort_user`：sleep 0.1s→删 ack_map/event_map→发 AbortMsg→tokenizer 转 AbortBackendMsg→scheduler `abort_req`）；取消后迟到消息被 listen 忽略（uid 不在 ack_map）；**参考实现已知清理缺口**（wait_for_ack 尾部 del 在提前 break 时不执行；非流式不检查断连；AbortMsg 不通知 DetokenizeManager 删状态；输入超长时只记警告不回错误——原文逐一列出，"指出缺口是为了区分理解主路径和生产稳定"）；慢客户端背压（Scheduler 不降速，ack_map 积压）。
3. **核心概念**：uid 隔离；ack_map+event_map 双件套；协作式取消。
4. **关键技术流程**：请求七阶段；取消链（客户端断开→API Server→AbortMsg→Tokenizer→AbortBackendMsg→Scheduler→资源释放）。
5. **公式参数**：无。
6. **代码实现**：如上。
7. **难点与易误解**：async def 不等于不阻塞；Event≠消息计数器（多次 set 合并成一次唤醒但消息都在列表里）；流式不减少计算量；`[DONE]` 非标准。
8. **动画表达形式**：**第一集样片 3.8/3.9 节（流式返回+停止条件）的原文依据**；交错返回 A1→B1→B2→A2 的 uid 分拣动画适合做第 9 集（HTTP 并发）素材。
9. **推荐补充资料**：MDN HTTP 概述/消息/SSE；FastAPI StreamingResponse 文档。

### 2.3.6 第 6 章 Continuous Batching 与调度（概念+代码）

1. **学习目标**：解释固定批处理为何被长请求拖住；区分等待队列/运行集合/本轮 batch；区分请求槽位/prefill token 预算/KV 容量；解释 prefill 优先与分块 prefill 对延迟的影响；区分整体吞吐与单用户速度。
2. **原文知识点清单**：batching 收益（权重读取复用、分摊固定开销；**batch 内请求各自独立，不把几人的对话拼成一段上下文**）；打包布局（input_ids=[123,456,789,321,654,987] 六 token 一维打包，逻辑可见范围=块对角下三角 mask，实际不必构造 [6,6] 矩阵）；固定批处理局限（任务长度不一致无法打包同时结束；A 结束后 C 仍要等 B）；**连续批处理=在模型执行的轮次之间调整成员**（不在算子中途塞入；不消除单请求自回归依赖）；A/B/C 完整调度表（2 槽位、C 第 2 轮接入、B 第 2 轮暂停一次 decode——"让新请求更早开始也可能影响已开始生成的请求"）；调度器三状态（等待队列 `PrefillManager.pending_list` / decode 运行集合 `DecodeManager.running_reqs` / 本轮 batch）；调度循环四步伪代码（receive→select_prefill_if_possible→否则 select_running_decode→forward→update）；新请求接入三检查（**请求槽位**=表行数、与 prompt 长度无关；**KV 容量估计**=尚未缓存输入+最大输出+其他请求预留（例：768+256+1024=2048>1800 则继续等）；**prefill token 预算**=本轮最多处理多少未缓存输入 token）；分块 prefill（2500 token 分 1024/1024/452 三轮；块间调度器有机会重新选择；本版本把未完成 prefill 放回队首继续优先 prefill，不保证块间插入 decode）；五请求调度实例（A–E 时间线，P/D₁/D₂ 标注）；prefill 优先 vs decode 优先（C 的 TTFT vs B 的 ITL；两种规则都可能让一方连续等待）；吞吐三视角（token/s 总吞吐、token/s/user、GPU 忙碌时间；"整体吞吐提高时每位用户生成速度仍可能降低"）；批处理提效机理（[1,d]×W vs [B,d]×W 同一份权重；分摊启动开销；一次前向仍含多个 CUDA kernel）；**GPU-Util 85% 的定义**（采样窗口内至少有一个 CUDA kernel 在执行的时间占比；不说明用了多少 SM/有效算力/带宽——两个配置都 85% 吞吐可以不同）；请求结束处理（结束条件：can_decode 耗尽或 EOS（ignore_eos 时忽略）；从运行集合移除→归还槽位→缓存管理器处理 KV（可复用前缀可保留）→"结束一条、补进一条"）。
   代码走读要点：`normal_loop`（blocking=not(runnable or runnable)）；`_schedule_next_batch`：`prefill_manager.schedule_next_batch(self.prefill_budget) or self.decode_manager.schedule_next_batch()`——**or 是策略**，等待队列非空不代表能组出 prefill batch；`PrefillAdder` 三约束对应源码（`table_manager.available_size` / `token_budget` / `estimated_len+reserved_size`）；`reserved_size` 初始来自 `decode_manager.inflight_tokens`；`_add_one_req` 用 `min(token_budget, remain_len)` 切块生成 `ChunkedReq`（中间片段不提交输出 token、不进 decode 集合）；`DecodeManager.filter_reqs`（union 后保留 can_decode）；EOS 判断在 `_process_last_data`（`finished |= int(next_token.item()) == self.eos_token_id`）；`_free_req_resources` 归还表行+`cache_req(finished=True)`；A/B/C 手算表（0~5 轮）。
3. **核心概念**：轮次间调整成员；三资源约束分立；prefill 优先策略及其代价。
4. **关键技术流程**：调度循环四步；接入检查三步；结束回收三步。
5. **公式参数**：KV 容量估计示例（768+256+1024 vs 1800）。
6. **代码实现**：如上。
7. **难点与易误解**：连续批处理不是"每轮必混批"（本版本每轮只执行 prefill 或 decode 之一）；chunk 中间块不能提交 token；GPU-Util≠有效算力利用率。
8. **动画表达形式**：双 lane 时间线（P/D 块与气泡）；三检查闸门动画；五请求甘特图。对应视频课程第 10 集；样片第一集只在调度站口头一句"调度器决定这一轮算谁"。
9. **推荐补充资料**：NVIDIA 矩阵乘性能说明；nvidia-smi 指标说明（custhelp a_id 3751）。

### 2.3.7 第 7 章 Paged KV Cache 与显存管理

1. **学习目标**：理解连续分配的三类浪费；掌握 Block/Slot/Block Table 抽象与映射；理解 PagedAttention 按块计算与在线 Softmax 的数学等价性。
2. **原文知识点清单**：传统方案（按最大长度预分配连续显存 `[batch,layers,max_seq_len,heads,head_dim]`）；**内部碎片**（预分配 2048 实际生成 200 → 约 90% 闲置；即使提前知道长度预分配仍低效）；**外部碎片**（3.60GiB 总空闲切成 512/256/1024/768/1024MiB 五块，第十一个请求要 1.50GiB 连续→失败 OOM）；动态扩展难题（重分配拷贝；无法高效支持并行采样/Beam Search 的前缀共享）；核心矛盾（显存按需高效分配 vs Attention 需要完整 K/V 序列）；分页三步解法（固定 Block；Block Table 映射；按块读取）；效果（内部碎片限制在最后一个块内；外部碎片彻底消除；动态扩展自然）；借鉴 OS 虚拟内存；抽象三概念（Block=显存分配最小单元，常见 16 token，可配 32/64；Slot=块内微观单位；Block Table=逻辑块号→物理块号的映射数组，CPU/GPU 各一份——CPU 供调度决策 GPU 供 kernel 实时查表）；映射四步（逻辑块划分→物理块分配→地址翻译→跨块连续访问；50 token/16=4 块，块 3 仅 2 个有效 slot）；PagedAttention 读取机制（逐块迭代：查表→读块内 slot→块内算分加因果掩码→在线 Softmax 累积；`p=jb+t`；无效 slot 权重 0）；**在线 Softmax**（三运行统计量 m_i 运行最大值 / ℓ_i 指数和 / O_i 加权输出；初始化 −∞/0/0；合并公式 `m'=max(m,m_j)`、`ℓ'·e^{m−m'}+ℓ_j·e^{m_j−m'}`、O 同步缩放——"ℓ 和 O 必须始终共享同一个基准"；核心代数 `e^{s−m_i}·e^{m_i−m_i'}=e^{s−m_i'}`；最终 `o_i=O_i/ℓ_i` 与标准 softmax 数学等价，浮点舍入差异非算法近似）；出处（Milakov & Gimelshein《Online Normalizer Calculation for Softmax》arXiv:1805.02867；FlashAttention 在此基础上做分块注意力）；等价性前提（分数可逐 token 计算、输出为 V 的加权和、分块只改变计算顺序）；非连续访存优化（内存合并访问、共享内存暂存、Block 大小调优 16/32/64）；**"PagedAttention 仅改变了 K/V 的内存寻址方式，未改变算子内部的数学运算逻辑"**。
3. **核心概念**：逻辑连续与物理连续解耦；Block Table 查表；在线 Softmax。
4. **关键技术流程**：映射四步；逐块读取+合并。
5. **数学公式**：块内分数 `s=q·k/√d` + 因果掩码；在线 Softmax 三统计量与合并公式（原文完整给出）。
6. **代码实现**：无（概念章；mini-sglang 对应实现在第 4/8 章代码篇）。
7. **难点与易误解**：PagedAttention 不是近似算法；块内独立 softmax 再相加是错的（必须在线合并）；FlashAttention 计算的是与标准注意力数学上完全相同的结果而非稀疏/低秩近似。
8. **动画表达形式**：虚拟内存类比动画（页表→块表）【教学推演，需注明类比来源 OS】；外部碎片"有空地盖不了房"演示；在线 Softmax 三账本动画（m/ℓ/O 三条仪表）。对应视频课程第 11 集。
9. **推荐补充资料**：vLLM PagedAttention 论文（arXiv:2309.06180）；Online Softmax 论文（arXiv:1805.02867）。

### 2.3.8 第 8 章 RadixAttention 与前缀缓存（概念+代码）

1. **学习目标**：理解为何只复用前缀不复用任意片段；掌握 Radix Tree 对比 Trie 的优势；走通 match/lock/allocate/insert/unlock/evict/free 生命周期。
2. **原文知识点清单**：传统 Cache 局限（生命周期绑定单请求；大量请求共享 system prompt/工具描述→跨请求重复计算）；前缀缓存=所有权转移到"可复用前缀序列"；**为什么只能前缀**（因果注意力：h_i 融合左侧上下文→后续层 K/V 携带前缀；"天气很好"/"心情很好"同后缀"很好"但左侧上下文不同→KV 不同；加 RoPE 后 Q/K 显式依赖位置）；RadixAttention 名字误导——**Attention 公式与计算流程完全不变**，它只是 KV Cache 的组织与调度（KV Cache Pool + Radix Tree + Cache-aware scheduler 三件套）；数据结构对比表（逐 token Trie 节点多 / Radix Tree 压缩边）；节点组成（`RadixTreeNode`：parent/children/key_fn/ref_count/uuid/timestamp/_key(token id 序列)/_value(物理槽位)/_length；len(key)==len(value) 根节点为空）；A/B/C 构建实例（[101,11,12,21,22]→[101,11,12,31,32] 在第 3 位分裂→[101,11,99] 在第 2 位再分裂；**分裂只重组树的元数据，共享前缀仍指向原物理槽位，不需要复制**）；对象三件套（Tree key=token id 序列；Tree value=token slot/page indices；KV Pool=每层 K/V 张量；**Tree 本身不保存巨大 K/V 张量**）；生命周期七步（Match 产生候选（此时"匹配到"≠"已安全持有"）→Schedule 决定谁先跑（mini-sglang 按到达顺序；SGLang 有前缀感知策略 lpm/dfs-weight/hrrn/shortest-prefill-first 与不感知策略 fcfs/lof/random/routing-key）→Lock（沿节点向根 ref_count+=1；evictable_size→protected_size；**加锁只负责保护，不负责分配新 page**）→Allocate 只为未命中部分申请缺口（[p6,p7]）→Insert 把新计算登记进树（竞争态发现重复 page 立即 free 防泄漏）→Unlock（ref_count−1；降 0 变可淘汰）→Evict（只挑引用次数为 0 的叶子节点按策略摘除）与 Free（无策略归还）；**成功共享的前缀 page 不会在请求结束时被 free，只 unlock**；SGLang evict 直接释放空间而 mini-SGLang 的 evict 只摘节点返回索引、实际释放由调度层完成）。
   代码走读要点：`_get_key_fn(page_size)`（page_size=1 用下一 token id，>1 用元组）；`_tree_walk`（get_match_len 用 fast_compare_key；**匹配长度按 page_size 向下对齐，不共享非整页前缀**；整节点命中刷新 timestamp 供 LRU；停在中部才 split_at）；`match_prefix` 返回 `MatchResult(RadixCacheHandle(prefix_len, node))`；`handle.get_matched_indices()` 沿 parent 走到 root 收集 value 反转拼接；`_try_allocate_one` 两次 available_size 检查（lock 改变可淘汰规模）；`allocate_paged` 只为 `[cached_len, device_len)` 涉及的新页分配；`_allocate` free list 不够时先 evict；`cache_req`（insert_prefix→unlock→free 被他人占用的中间段→finished 则 free 尾部否则 lock 新句柄）；`check_integrity`（free_slots+cache_pages==num_pages；free_slots 页对齐断言）；ChunkedReq 重写 `can_decode=False`；Decode 不逐步 insert 树（只请求结束时归档）；取消与正常结束共用 `_free_req_resources`；`lazy_free_region` 批量拼回 free_slots、`finished_reqs` 防二次释放；对照真实 SGLang 表（mem_cache/radix_cache.py、向 unified_cache 收敛、schedule_policy.py、kernels/ 与 sgl-kernel）。
3. **核心概念**：token 粒度前缀共享；引用计数保护；LRU 叶子淘汰。
4. **关键技术流程**：七步生命周期。
5. **公式参数**：无。
6. **代码实现**：如上。
7. **难点与易误解**：RadixAttention 不改注意力数学；匹配到≠安全持有（要 lock）；evict 只能从无引用叶子开始；page_size>1 时不共享非整页前缀。
8. **动画表达形式**：前缀树生长动画（A→B 分裂→C 再分裂，配"分裂只拆索引不搬数据"字幕）；ref_count 灯条；LRU 淘汰时钟。对应视频课程第 12 集。
9. **推荐补充资料**：SGLang 论文（arXiv:2312.07104）；LMSYS SGLang 发布博文（2024-01-17）；Unified Radix Cache 博文（2026-08-11）。

### 2.3.9 第 9 章 多进程与张量并行（概念+代码）

1. **学习目标**：区分 Multi-process 与 TP 的职责；掌握控制面/数据面通信；掌握 Column/Row/Vocab 切分与 QKV/KV 头规则；串起从启动到处理的流程。
2. **原文知识点清单**：单卡装得下≠目标负载能稳定运行（Qwen2.5-32B BF16 权重 64GB；KV/激活/通信缓冲/CUDA Graph 缓冲都要显存）；1.6T 参数 BF16≈3.2TB 必须多卡；TP 引入跨卡通信与同步，"多卡部署并不意味着计算一定会更快"；**Multi-process 是系统组织方式**（绕过 GIL；API Server/Tokenizer/Detokenizer/Scheduler（每 rank 一个，绑定一张 GPU）/Engine）；**TP 是模型并行策略**（同一层权重/词表切到多 rank 共同完成同一批输入；每个 token 由整个 TP 组协同计算）；控制面（**ZMQ**：API Server↔分词反分词 worker↔rank0 scheduler，rank0 经 Pub/Sub 向其他 scheduler 广播；**Gloo CPU 进程组**：barrier、NCCL Unique ID 广播、每轮非阻塞接收消息数量同步；只有 rank0 对外通信）；数据面（**NCCL/PyNCCL**：All-Reduce `Y=Y₀+…+Y_{p−1}`（Row Parallel 用）与 All-Gather `Y=cat(Y₀,…)`（LM Head 用）；缺少通信则输出严重偏差）；切分原则（RMSNorm 等同步部分全复制；大权重按维度切）；四种切法（Column Parallel 按输出维度切——qkv_proj、gate_up_proj；Row Parallel 按输入维度切——o_proj、down_proj，接 All-Reduce；Vocab Parallel 按词表切——pad 到整除后均分，logits All-Gather；Query 头按 head 切要求 num_heads%tp_size==0，KV 头 num_kv_heads≥tp_size 时切分、**<tp_size 时复制 KV head**）；tp_size 选择考虑（参数量/显存、结构约束与数学等价）；启动三步（start_subprocess 按 rank 拉起 scheduler 进程名 `minisgl-TP{i}-scheduler`→各 Scheduler 内创建 Engine（绑卡 cuda:{rank}、meta device 建结构再 load 切分后权重、KV Cache/Attention Backend/CUDA Graph capture）→API Server 监听）。
   代码走读要点：`DistributedInfo(rank,size)`；通信三配置表（tp=1 Gloo；tp>1 默认 PyNCCL communicator+Gloo WORLD；--disable-pynccl 走 torch.distributed NCCL+另建 Gloo group）；`DistributedCommunicator.plugins` 列表取最后一个 plugin；`models/weight.py` `_shard_tensor`（`_SPLIT_DIM_0=[q_proj,k_proj,v_proj,gate_proj,up_proj]` 按输出维 chunk；`_SPLIT_DIM_1=[o_proj,down_proj]` 按输入维 chunk；GQA 且 num_kv_heads<n 时按 head 取 `value[head_idx*head_dim:(head_idx+1)*head_dim]`；lm_head/embed_tokens 按 vocab 切 div_ceil；RMSNorm 完整保留）；**先 shard 再 merge**（q/k/v 拼成 qkv_proj、gate/up 拼成 gate_up_proj；若先全局拼接再均分会破坏配对；流式加载控制峰值 CPU 内存；MoE 各 expert stack 成专家维）；`VocabParallelEmbedding.forward`（tp>1 时 all_reduce）；`ParallelLMHead.forward`（prefill 只取最后位置、tp>1 时 all_gather 拼回完整词表 logits 并截断）；请求广播（rank0 `_recv_msg_multi_rank0`：Gloo 广播条数+ZMQ Pub 发内容；rank1 `_recv_msg_multi_rank1`：收条数再逐条取；**相同请求集合+相同顺序+相同状态才是可协同计算的 Batch，是 RadixAttention 前缀缓存正确协同的前提**；运行时不每轮广播完整 Batch 对象）；对照真实 SGLang 表。
3. **核心概念**：控制面保证"做同一件事"，数据面保证"结果能正确合成"。
4. **关键技术流程**：启动→切分加载→广播→协同前向。
5. **公式参数**：All-Reduce/All-Gather 定义式。
6. **代码实现**：如上。
7. **难点与易误解**：TP 翻倍≠显存减半速度翻倍（复制开销/KV 头数限制/通信成本）；单卡也初始化 Gloo；先 shard 后 merge 的顺序原因。
8. **动画表达形式**：权重矩阵切块着色动画；All-Reduce 汇聚动画；rank0 广播总线图。对应视频课程第 13 集；Part Ⅲ 第 4 章承接。
9. **推荐补充资料**：Qwen2.5-32B config；NCCL 文档。

### 2.3.10 第 10 章 投机解码（概念+代码）

1. **学习目标**：解释普通 Decode 的串行依赖与投机方案；走完起草→验证→接受→纠正→提交；区分贪心/随机验证；判断是否划算。
2. **原文知识点清单**：普通 Decode 每调用一次模型只确定下一个 token（KV Cache/CB/TP 都不消除相邻 token 依赖）；**收益来源=低并发 Decode 场景提高 GPU 利用率**（并发已高时开启未必有收益——原文加粗强调）；Target Model（大，定规则）vs Draft Model（小，低成本提候选；"参数少本身不是目的，需要的是起草足够便宜且连续猜中足够长"）；投机可能增加总计算量，争取的是减少串行调用；基本流程三阶段（起草：草稿逐步提出 k 个候选；验证：目标模型对含候选的序列做因果前向并行计算各候选位置的预测分布 `p(·|x)`、`p(·|x,y₁)`、…、`p(·|x,y₁..yk)`；提交：只把允许接受的部分写入正式输出）；贪心验证（草稿 token=目标选择→接受；首次不一致→提交目标选择的纠正 token 结束本轮；后续候选预测依赖失效前缀直接丢弃；"提前算出了预测，不代表拒绝后还会继续判断"；与目标单独贪心生成一致）；随机采样验证（接受概率 `α(y)=min(1,p(y)/q(y))`；首次拒绝后从归一化正残差抽取 `r(v)=max(p(v)−q(v),0)/Σmax(p(u)−q(u),0)`；p=[0.4,0.6], q=[0.8,0.2] 实例：A 接受概率 0.5 贡献 0.4、B 全接受贡献 0.2、拒绝分支 0.4 全补 B→最终 [0.4,0.6] 回到目标分布；**拒绝后直接从 p 重抽会破坏结果**；保持的是分布不是相同 seed 的逐 token 相同）；温度/Top-k/Top-p 的位置（用于调整 p、q 各自的分布，用处理后的分布验证，不需要新的接受流程）；本轮输出（连续接受 a 个+1 个纠正或 bonus；最多 k+1）；停止条件以正式输出为准（草稿 EOS 被拒绝不会结束请求）；缓存边界（下一轮从实际提交的前缀继续；保留 `I like` 的 KV、不能继续用 `green tea` 的 KV、black 还要算自己的 KV）；划算判据（`T_token,spec≈T_round/E[m]`；例：普通 10ms/步，投机一轮 16ms 提交 3.2 个→5ms/token≈2 倍加速；只提交 1.2 个→13.3ms 反而更慢——"假设数字，只用于说明计算方法"）；草稿长度/模型/负载三因素；评估注意（固定目标模型硬件数据采样参数与输出限制；单请求加速比≠服务吞吐提升）。
   代码走读要点：本篇定位真实 SGLang 固定提交 `1291f317`（mini-sglang 9a91cfa 无投机模块）；入口（`standalone_worker_v2.py` StandaloneDraftWorker/StandaloneWorkerV2；`eagle_worker_v2.py` `forward_batch_generation`：draft→spec_info→verify→`_draft_extend_for_decode`；`eagle_info.py` EagleVerifyInput 的 draft_token/positions/custom_mask/retrieve_*）；教学片段（贪心起草循环 min(draft_steps, remaining)；验证行号表 `L−1..L+k−1`；`greedy_commit` 在首次不一致处返回；随机路径 `acceptance=min(1,p[y]/q[y])`+`residual.clamp_min(0)`+multinomial）；缓存恢复（真实引擎可能已为未接受草稿写入缓存，首次拒绝后需恢复到已确认路径；纠正/bonus token 尚未完成自己的前向，不能把被拒候选的 KV 当它们的缓存）。
3. **核心概念**：先猜后验；拒绝校正保分布。
4. **关键技术流程**：起草→验证→提交→（缓存恢复）→下一轮。
5. **数学公式**：`α(y)=min(1,p(y)/q(y))`；残差公式；`T_token,spec≈T_round/E[m]`。
6. **代码实现**：如上。
7. **难点与易误解**：投机解码不改变输出分布（随机路径）；连续性（首次拒绝即停）；bonus token 只有全部接受时才有；EOS 需验证。
8. **动画表达形式**：双模型分工动画（大模型法官+小模型猜词人【教学推演】）；验证行表格逐行判定动画；残差补概率的柱状重组。对应视频课程第 14 集。
9. **推荐补充资料**：Leviathan et al.《Fast Inference from Transformers via Speculative Decoding》（arXiv:2211.17192）。

---

## 2.4 Part Ⅲ 高级推理技术（深入真实 SGLang）

### 2.4.1 第 1 章 注意力后端

1. **学习目标**：区分 Attention 机制/kernel/backend 三层；解释为何需要不同后端；说明 FlashInfer/Triton/FA3/FlashMLA 思路与取舍；会按场景选择验证。
2. **原文知识点清单**：三层概念表（机制=MHA/GQA/MLA 结构；kernel=具体 GPU 程序；backend=准备输入并调 kernel 的适配层）；backend 三职责（准备元数据——batch 中各请求新 token 数/历史长度/KV 位置/允许关注范围；处理 KV 读写——分页也要按正确顺序参与计算；按阶段调路径）；对应 SGLang `AttentionBackend` 接口（`init_forward_metadata` / `forward_extend` / `forward_decode`）；为何多后端三角度（Prefill 多 Query 瓶颈 compute 倾向复用片上数据 vs Decode 单 Query 瓶颈 memory 倾向高效读历史 KV+split-KV 拆段并行、模型结构（MHA/GQA/MLA 缓存布局不同；滑动窗口/特殊 mask/投机验证树/KV 数据类型）三、GPU 架构（Hopper TMA+WGMMA 异步流水线））；朴素 Attention 的搬运瓶颈（32k 序列单层单头分数表 2GiB，写出读回 4GiB）；FlashAttention 两个直觉（分块进片上 SRAM；边算边归一化——"记两笔账"调整此前累积输出比例；Q/K/V 仍要从 HBM 读入，长序列 Prefill 计算量仍二次增长）；FlashInfer 三步（先知道数据在哪——索引=地址清单，支持分页 kernel 按清单读不必复制；Plan 把工作拆成适合并行的任务——长 KV 用 split-KV、均衡工作量；Run 按计划读数据计算合并；**Plan 与 Run 分离，兼容层复用 Plan**，每层 Q/K/V 不同所以每层 Run）；FlashInfer 代价（索引与计划有开销；请求短或 batch 已满时拆分未必更快）；Triton（语言+编译器；易读改扩展；性能依赖编译调优；首次编译有成本；"不能仅凭代码容易写推断一定更慢"）；FA3（TMA+异步矩阵乘流水线；限制在硬件与形状匹配）；FlashMLA（MLA 缓存压缩来自模型结构，FlashMLA 负责高效执行；当前主要在 Decode 路径调用，EXTEND 走继承的 Prefill 实现——"一个 backend 可以组合多种 kernel"）；四后端对比表；SGLang `--attention-backend` 全部 24 个内置选项速查表（flashinfer/triton/fa3/fa4/flashmla/torch_native/flex_attention/trtllm_mha/trtllm_mla/cutedsl_mla/tokenspeed_mla/hpc_ops/dsa/qsa/dsv4/minicpm_*/aiter/wave/ascend/intel_amx/intel_xpu + 2 个弃用别名）；选择与验证（先筛兼容；未指定时 SGLang 自动选择作初始基线；`--prefill-attention-backend`/`--decode-attention-backend` 可分阶段拼接但不能随意；比较指标 TTFT/TPOT/吞吐/显存/正确性；长输入短输出看 Prefill、短输入长输出看 Decode）。
3. **核心概念**：backend=模型执行与计算实现之间的适配层；切换后端通常不改变模型结构（浮点误差需验证）。
4. **关键技术流程**：朴素→分块融合→面向推理服务组织（索引/Plan/Run）。
5. **公式参数**：32k 分数表 2GiB 估算（原文给出）。
6. **代码实现**：SGLang 后端文件链接表（base_attn_backend.py / flashinfer_backend.py / triton_backend.py / flashattention_backend.py / flashmla_backend.py，引用版本 1d019ce）。
7. **难点与易误解**：FlashAttention 省的是中间矩阵搬运不是计算本身；FlashMLA 不能用于任意 MHA 模型；后端名称≠每阶段同一实现。
8. **动画表达形式**：三层蛋糕图（机制/kernel/backend）；朴素 4×4 表整存整取 vs 分块流水对比；Plan/Run 双轨时间线。对应视频课程第 15 集。
9. **推荐补充资料**：FlashAttention 论文（arXiv:2205.14135）；FlashInfer 论文（arXiv:2501.01005）与文档；Triton Fused Attention 教程；FA3 作者博客；FlashMLA 仓库。

### 2.4.2 第 2 章 量化与低精度推理

1. **学习目标**：说明量化如何省存储与提速、为何引入误差；从 Decode 搬运与 KV 容量两方面解释收益；发现精度回归；读懂量化标签。
2. **原文知识点清单**：量化定义（16 位→8/4 位；π 各格式舍入表：FP32 误差 <1e-7 / FP16 ≈0.001 / FP8-E4M3 ≈0.11 / FP4-E2M1 ≈0.14）；W/A 记号与方案表（W4A16 AWQ/GPTQ 只量化权重计算前反量化；W8A8-FP8 直接 FP8 Tensor Core（Ada/Hopper/Blackwell）；W8A8-INT8；NVFP4 W4A4 Blackwell 原生、旧卡退回 W4A16；MXFP4 gpt-oss MoE 专家权重；FP8 KV Cache 单独 `--kv-cache-dtype fp8_e4m3`）；位分配图（FP16 1+5+10 最大 65504；BF16 1+8+7 范围同 FP32；FP8 E4M3 1+4+3 最大 448、E5M2 1+5+2 最大 57344；INT8 −128..127；INT4 −8..7；FP4 E2M1 16 个值）；scale `x≈q×s`（INT4 s=0.5 对应 −4.0..3.5）；8B 模型实测体积（BF16 16.38GB / FP8 9.44GB / AWQ 6.10GB / NVFP4 6.40GB——多出的是 embedding+lm_head 保 16 位约 2.49GB 与 scale（AWQ 每 128 个一组 0.14GB、NVFP4 每 16 个一个 FP8 scale 0.43GB））；提速机理（Qwen 官方 Speed Benchmark：SGLang 0.4.6.post1、H20 96GB、bs=1、生成 2048、输入 1→12.9 万：FP8 1.84×→1.19×、AWQ 1.76×→1.02×；输入越长加速越小——权重读取占比下降；W4A16 反量化开销+BF16 矩阵乘 vs FP8 原生）；单步耗时下限 `权重字节/显存带宽`（H20 4TB/s：16.4GB→4.1ms / 9.4GB→2.4ms / 6.1GB→1.5ms；实测 BF16 每步 12.2ms 含其余开销）；KV 容量（Qwen3-32B 每 token KV=2×64×8×128×2B=256KiB；1024+512 请求 384MiB；H20 81.6GB 预算下 BF16 权重 65.5GB 剩 16.1GB≈6.1 万 token、FP8≈18 万、AWQ≈23.8 万、KV 也 FP8≈47.5 万；请求 39→154 个；Qwen3-8B 换 INT4 容量只 +16%——权重占比越高收益越大）；精度代价（离群值把 scale 撑粗——同组 7 个小权重+离群值 4.0 时 5 个被舍成 0；缩小分组/按通道/token）；掉点实测（Qwen3-8B 思考模式 AIME24 76.0→71.3（−4.7）、MMLU-Redux −1.1；32B 思考模式 GPQA +0.6 偶尔升高；GPT-OSS-120B KV Cache FP4：GSM8K 0.9161→0.9152 几乎不变但 AIME25 0.7533→0.3533——**评测要覆盖目标任务的难度与输出长度**）；评测清单五条；实战（读 config 的 quantization_config：AWQ bits:4 group_size:128 zero_point:true；FP8 fmt:e4m3 weight_block_size:[128,128] activation_scheme:dynamic；NVFP4 hf_quant_config.json group_size:16 kv_cache_quant_algo:FP8；gpt-oss-120b quant_method:mxfp4 但 Attention/router/embedding/lm_head 不量化；"名字只是线索，配置文件才是依据"）；加载三条命令（预量化直接路径自动 awq_marlin；`--quantization fp8` 在线转；`--kv-cache-dtype fp8_e4m3`）。
3. **核心概念**：位数↓→搬运↓+容量↑；离群值；分组 scale。
4. **关键技术流程**：量化→加速估算→精度评测→上线验证。
5. **数学公式**：`x≈q×s`；`每步耗时≳权重字节/带宽`；KV 每 token 字节公式。
6. **代码实现**：启动命令三条（原文）。
7. **难点与易误解**：INT4 下载体积≠位宽估算（embedding/lm_head/scale）；GSM8K 不掉点≠没影响（AIME 掉一半）；量化标签要查 config。
8. **动画表达形式**：数轴刻度稀密动画（FP16/FP8/INT4 的 π 落点）；离群值撑粗刻度对比；显存蛋糕重切动画（权重/KV 占比）。对应视频课程第 16 集。
9. **推荐补充资料**：AWQ（arXiv:2306.00978）、GPTQ（arXiv:2210.17323）、Microscaling（arXiv:2310.10537）、DeepSeek-V3 技术报告（arXiv:2412.19437）；Qwen Speed Benchmark 文档。

### 2.4.3 第 3 章 分层缓存（HiCache）

1. **学习目标**：解释为何有前缀缓存还需分层；说明 L1/L2/L3 存什么与命中含义；描述备份→驱逐→恢复过程；比较读回与重算成本；判断适用场景。
2. **原文知识点清单**：动机（长文档两问场景：GPU 显存有限被驱逐后失去复用机会；Paged 管分配回收、前缀缓存避免重复计算、HiCache 回答"GPU 暂时放不下的能否先留在别处"）；三层表（L1 GPU 显存直接用；L2 本实例 Host 内存搬回 GPU；L3 可选外部存储——共享配置时支持跨实例，先读到 Host 再搬回）；跨层匹配三步（先匹配本地 L1/L2 查索引；再按需查 L3——按缓存页生成查找 key=前缀指纹，结合当前页 token 与此前前缀，避免相同文字不同上下文混在一起；确定能连续复用多长——各层命中必须从开头连续接起来不能跳段；查到副本≠就绪，L2/L3 仍要搬运到 GPU）；实例表（1024 token：1–512 L1 直接用 / 513–768 L2 恢复 / 769–1024 L3 先读 Host 再恢复；最后一段无副本则只能复用 768）；比较 token 完全一致，不能意思差不多就复用（system prompt/模板不同也影响）；混合架构还要检查 SWA 窗口 KV、MAMBA 递归状态（Unified Radix Cache）；生命周期（第一次请求：Prefill 算 KV→按策略备份到 Host/L3——GPU 原数据可继续用；备份有带宽与容量成本；第二次请求：L1 命中直接用/L2 命中分配 GPU 槽位搬回/L3 命中先 Host 后 GPU/无缓存重算；**缓存了文档不意味着回答新问题不再使用文档内容**——新增 token 的 Attention 仍要读文档 KV）；性能取舍（`T_restore≈T_storage→Host+T_Host→GPU`；只有恢复比重算快才有直接延迟收益；假设例：重算 200ms vs L2 恢复 40ms 划算 vs L3 250ms 不划算——"数字不是实测"）；备份成本（大量前缀以后不再用=白写）；prefetch 预取（B 排队时把 L3 读到 Host，与 A 的计算交叠；能否隐藏取决于排队窗口与存储速度）；三策略（best_effort 不等、wait_complete 等完、timeout 折中——没有固定优劣）；按层恢复（第 0 层 KV 就绪即可开始第 0 层计算，后续层边搬边算——不取消数据依赖）；场景判断（多轮对话/长文档问答/Agent 相同前缀受益；热点都能留 L1 则收益有限；几乎无相同前缀/输入短/链路拥堵则未必）；`--enable-hierarchical-cache` 启动（不自动配共享 L3）；验证（重复相同长前缀+中间插流量；观察 TTFT/输入吞吐/尾延迟+各层实际复用 token 数/Host 占用/搬运负载）。
3. **核心概念**：L1/L2/L3；前缀指纹；恢复 vs 重算。
4. **关键技术流程**：备份→驱逐→命中→恢复（或重算）。
5. **公式参数**：`T_restore≈T_storage→Host+T_Host→GPU`。
6. **代码实现**：`HiCacheController`（write/load/prefetch 对应三类动作）；`--enable-hierarchical-cache`。
7. **难点与易误解**：命中率增加≠TTFT 改善（恢复有成本）；预取不能取消数据依赖；HiCache 不提高最大上下文长度、不加速 Decode。
8. **动画表达形式**：三层金字塔+KV 块在层间搬运的动画；恢复/重算天平。对应视频课程第 17 集。
9. **推荐补充资料**：SGLang hicache_design/hicache_best_practices 文档；LMSYS HiCache 博文（2025-09-10）；AI Infra Visualized HiCache 课程。

### 2.4.4 第 4 章 横向扩展（DP Attention / EP / PP）

1. **学习目标**：理解 TP 的两个限制（通信代价、KV 切分极限）；掌握 DP Attention 按请求分工；掌握 EP 专家分发合并；理解 PP 流水线气泡；会选择组合。
2. **原文知识点清单**：TP 通信代价（每层 Attention/FFN 输出投影都要通信；4→8 卡省的计算可能不够新增通信；小 batch Decode 尤甚；跨机还要看网络带宽）；**KV 切分极限**（GQA：8 个 KV 头 8 路 TP 每卡 1 头，16 路时没头可分部分卡复制；MLA：latent 压缩状态无法按"每卡不同 KV 头"切，复制实现下同一请求缓存在多卡各一份——"仅仅增大 TP 未必让每张卡少存 KV"）；DP Attention（请求间 Attention 互不依赖→不同 GPU 处理不同请求各存各的 KV；4 请求例：TP 下每卡存 4 份共 16GB→DP Attention 每卡 1 份共 4GB；每卡 4GB 可放 4 请求→4 卡 16 个=容量×4（来自四重复制；原本能均匀切分就没这么多省）；还要扣 Attention 权重副本/缓冲；**Attention 独立不代表整层无通信**——FFN/MoE 仍需 TP/EP 数据交换，历史 KV 留在负责卡）；DP Attention vs 普通 DP（普通 DP=完整模型副本独立服务；DP Attention 只对 Attention 按请求分工）；MoE 回顾（多套专家网络 router 选 top-k；专家是训练出的参数不是"数学专家/写作专家"——原文明确打比方警示；权重常驻全部专家都要显存）；EP 实例（8 专家 4 卡各 2 个；GPU0 token 分到 E2/E7：dispatch 发激活→专家计算→combine 按路由权重合并；All-to-All 通信；负载不均问题——一批 token 都选 E2 则 GPU1 忙死）；PP（24 层分 3 stage 各 1–8/9–16/17–24 层；各存自己权重和 KV；跨 stage 传中间激活；**一个 stage 可多卡协作；PP 划分的是层，PD 划分的是输入处理与生成两阶段**）；气泡（单批数据三卡轮流；micro-batch A/B/C/D 四份交错——6 个时间片 vs 12；头尾仍有气泡）；micro-batch 代价（太小计算量不足+调度通信次数多；各 stage 耗时要接近；**同一请求必须先生成当前 token 才能算下一个——不能拿同一请求未生成的 token 填流水线**）；组合方法表（TP/DP Attention/EP/PP 各按什么分工、解决什么、代价什么；实际可 PP 分 stage+stage 内 Attention 按请求+MoE 按专家+部分 TP；DeepSeek-V3 §3.4 实例：Attention 用 TP+DP，MoE 用 EP；"不能直接把所有并行度相乘"）。
3. **核心概念**：按维度/按请求/按专家/按层四种切法。
4. **关键技术流程**：dispatch→专家计算→combine；micro-batch 流水。
5. **公式参数**：无。
6. **代码实现**：无（引用 SGLang 文档与博文）。
7. **难点与易误解**：容量×4≠吞吐×4；EP 负载不均；PP 单请求 Decode 不会因加 stage 更快。
8. **动画表达形式**：四卡切蛋糕动画；专家路由飞线；流水线甘特图与气泡灰格。对应视频课程第 18 集。
9. **推荐补充资料**：GQA（arXiv:2305.13245）；DeepSeek-V2 MLA（arXiv:2405.04434）；SGLang v0.4 博文（2024-12-04）；96 H100 大规模 EP 博文（2025-05-05）。

### 2.4.5 第 5 章 Prefill-Decode 分离

1. **学习目标**：解释长 Prefill 为何让其他请求停顿；说明 P/D 分工与传输数据；估算 KV 传输时间与两端 GPU 配比；判断适用场景；理解与前缀/分层缓存的配合。
2. **原文知识点清单**：workload 差别（Prefill 大矩阵算力倾向 vs 小 batch Decode 带宽倾向）；停顿场景（A 输出中 B 长文档到达→A 等待用 ITL 观察，TPOT 可能平均掉——连回 Benchmark 章）；Chunked Prefill 的作用与限制（缩短单次等待但共用 GPU；切小了 B 轮数多切大了 A 等久；请求持续到达则反复影响）；PD 分离定义（P/D 各一端，可以是一张卡或一组卡，甚至异构 NPU——"异构 PD 分离"）；请求三步（P 处理输入算 KV 采首 token→传输 KV+首 token+请求信息→D 收到后从第二个 token 继续生成）；**两端都要完整模型权重**（PP 分层，PD 分阶段——原文明确对比）；分别调优空间（P 集中大输入 batch；D 按生成数控制每轮耗时；D 接收太多输出仍会变慢）；代价三件（KV 传输时间 `T≈S/B_eff`：1GiB@20GiB/s=50ms、@5GiB/s=200ms；多条请求分享链路带宽；模型副本与显存——两端各一份权重+传输期间 KV 双份+D 要留生成空间）；资源配比（文档问答输入长输出短→P 多配；长文生成→D 多配；看排队位置判断；前缀缓存降低 P 工作量）；适用场景表（长文档问答/长回答生成/少量短问短答——分离收益小还多一次传输）；DeepSeek-V3 实测（SGLang 团队 2025-05 博文：Prefill 4 节点 EP32 2K 输入 54,543 input token/s；Decode 9 节点 EP72 22,282 output token/s；**Decode 相对 TP16 最高约 5.2×但同时用了 EP 与更大 batch——不能全归给 PD**）；KV 传输优化三向（量化少传/带宽 RDMA/计算与传输重叠——P 算好一部分先发）；与缓存配合（P 命中缓存少算；D 也有兼容前缀且传输支持时可只传缺失部分；HiCache 让 P 保留恢复更多前缀）。
3. **核心概念**：阶段分工；KV 交接；异构 PD。
4. **关键技术流程**：P 算 KV+首 token→传输→D 接管生成。
5. **公式参数**：`T_transfer≈S_transfer/B_effective`；示例带宽 20/5 GiB/s。
6. **代码实现**：SGLang `disaggregation/prefill.py`（引用）。
7. **难点与易误解**：两端不是各负责一半层；公开加速比不能全归 PD；短问短答场景分离反而亏。
8. **动画表达形式**：双 lane 分卡动画+KV 包裹传输动画（带宽天平）。对应视频课程第 19 集。
9. **推荐补充资料**：SGLang PD 文档；LMSYS 96 H100 博文（2025-05-05）。

---

## 2.5 Part Ⅳ 如何为 SGLang 做贡献

### 2.5.1 第 1 章 用 Cookbook 部署 SGLang

1. **学习目标**：会找合适 recipe；理解启动参数与课程概念的对应；会验证服务并用数字描述一次配置调整。
2. **原文知识点清单**：Cookbook=官方部署 recipe 集合（docs.sglang.io/cookbook）；使用前提核对（模型版本/GPU 型号数量/权重精度/SGLang 版本）；Qwen3 recipe 实例（Interactive Command Generator 选 H100/8B/BF16/Parser Disabled）；启动命令 `--model-path Qwen/Qwen3-8B --dtype bfloat16 --tp-size 1 --host 127.0.0.1 --port 30000`；参数-概念对照表（model-path→加载权重；tp-size→切分与通信；dtype→精度与量化；默认前缀缓存→RadixAttention，`--disable-radix-cache` 可关）；验证三步（curl 请求 non-thinking 模式 `chat_template_kwargs:{"enable_thinking":false}`；看日志与 nvidia-smi——显存占用高不等于泄漏；bench_serving 小规模测试 `--dataset-name random --random-input-len 256 --random-output-len 64 --num-prompts 80 --max-concurrency 4 --seed 42`）；三常见问题（显存不足/下载失败/端口占用）；随机文本测试不能判断回答质量。
3. **核心概念**：recipe 是起点不是终点；每次只调一个配置。
4. **关键技术流程**：跑通→验证→单变量对照测试。
5. **公式参数**：无。
6. **代码实现**：启动/curl/benchmark 命令（原文给出）。
7. **难点与易误解**：能返回回答≠高并发性能达标≠精度达标；旧版入口 `sglang.bench_serving` 与新版 `sglang.benchmark.serving` 不同。
8. **动画表达形式**：命令参数-概念连线图；对照实验流程卡。对应视频课程第 20 集（动手向）。
9. **推荐补充资料**：SGLang Cookbook；Server Arguments 文档；Bench Serving Guide。

### 2.5.2 第 2 章 Profiling 与 Trace 分析

1. **学习目标**：分清 benchmark/profiling/trace；掌握 trace 三看（耗时长/重复多/空隙）；走通"症状→假设→trace→修改→验证"全流程。
2. **原文知识点清单**：benchmark 黑盒 vs profiling 白盒（TTFT 高可能排队也可能 Prefill 慢——单看指标分不清）；工具链（PyTorch Profiler 采集→Perfetto 打开 trace；Profiler 记录、trace 是数据、viewer 展示）；trace 读法（时间线；色块=事件宽度=时长；三关注+重叠执行——"把所有 duration 加起来不等于请求延迟，真正影响完成时间的是必须等待的那条执行路径"）；方法四步（固定环境与负载→提可检查假设（TTFT 高查排队/prefill；TPOT 抖查 decode 是否被 prefill 插入阻塞；吞吐低查实际 batch/并发/计算通信等待）→短 trace 对应代码→改一个问题→关 profiler→同条件 benchmark+正确性检查）；**实例 PR #13587**（BBuf 提交 2025-11-21 合并；Kimi K2 优化系列；MoE `moe_align_block_size()`：超 4096 元素时先 `sorted_ids.fill_()` 再 alignment=多一次独立 kernel；改动=把 padding 交给已有 kernel（`fuse_sorted_ids_padding` 参数恒 True）；CUDA 实现 `pad_sorted_token_ids` 为真时填 numel=无效 token 标记——"省掉的是独立执行的开销，正确性需要的填充仍然保留"）；trace 读图（Before 7.854μs → After 6.544μs，该段 −16.7%）；benchmark（DeepSeek-R1 TP8、random 1000/1000、5 warmup：并发 1 −0.25% / 4 +0.28% / 16 +1.02% / 32 +1.10%）；正确性（GSM8K 1319 题 accuracy 0.938 invalid 0——但没给成对 main/PR accuracy，不能声称完全等价）；PR 价值示范点（scope 小、证据对应代码、多并发档位限定收益范围）。
3. **核心概念**：局部加速≠端到端等比提升；证据链。
4. **关键技术流程**：症状→假设→trace→改→benchmark+正确性。
5. **公式参数**：7.854→6.544μs 等实测（原文标注来源）。
6. **代码实现**：PR diff 片段（原文引用合并 commit bfcf15a）。
7. **难点与易误解**：stream 空隙≠GPU 全闲；kernel 变快≠端到端等比；CI 通过≠自动合并。
8. **动画表达形式**：trace 甘特图动画（Before/After 色块对比）；优化证据链流程图。对应视频课程第 21 集。
9. **推荐补充资料**：PR #13587 原文与截图；PyTorch Profiler 教程；SGLang profiling 文档。

### 2.5.3 第 3 章 SGLang PR 工作流

1. **学习目标**：知道提问/修改前准备什么；理解 Fork/分支/CI/Review 各保护什么；会选范围合适的首次贡献。
2. **原文知识点清单**：三件小事（定义明确的问题——性能要同条件前后 profiling，bug 要最小复现/log/版本/命令/预期 vs 实际；较大改动先 RFC/issue 讨论，错别字可直接 PR；PR 保持有限 scope——一个 PR 一个问题；用 AI 辅助也必须自己读懂并如实说明未测部分；不能批量制造低质量 issue/PR 形成"DDoS 式负担"）；PR 生命周期六步表（Fork→分支→提交开 PR→CI 检查→Review→合并，及每步"保护什么"）；写好 PR 三问（原来什么问题/改完会怎样/凭什么认为改对了；没有 GPU 实测就直接说明，不把"文档构建通过"写成"部署验证通过"）；CI 细节（GPU 测试需 `run-ci` 标签及权限触发；失败先读日志分清代码错误与环境问题，不要反复重跑碰运气）；Review 往返（继续向原分支 push 即更新原 PR；新增提交需重新验证——先前的绿色检查不能代替新版本）；贡献阶梯表（修文档→补 Cookbook→复现定位问题→修小 bug+回归测试）。
3. **核心概念**：scope 有限；证据匹配改动。
4. **关键技术流程**：PR 六步生命周期。
5. **公式参数**：无。
6. **代码实现**：无。
7. **难点与易误解**：CI 通过≠合并；重新 push 后旧检查失效。
8. **动画表达形式**：PR 生命周期流程图（GitHub 界面示意）；贡献阶梯。对应视频课程第 22 集（可并入社区番外）。
9. **推荐补充资料**：SGLang Contribution Guide；文档 CONTRIBUTING.md。

---

## 2.6 Community 社区专区

1. **定位**：收录学习者自己的实践（builds 手搓记录 / pitfalls 踩坑 / notes 学习笔记），与正文互补；提交需满足 PR_requirement.md 十条（环境信息必须完整、日志用代码块不截图、不点名其他推理项目对比、必须自己验证过、用自己的话写、CC BY-NC-SA 4.0）。
2. **已收录笔记**：KevinEZ06《llm-engine-viz：交互式 LLM 推理可视化》（2026-09-14）——浏览器可视化站点，页面覆盖一个 Token 从输入到输出（网关→K8s→P/D→输出）、连续批、PagedAttention、KV 调度（Swap/Recompute）、Radix、推测解码、TP/DP/EP/CP、Ring Attention、PD 分离（KVPoll 分卡交接）、路由、K8s+Mooncake。**对本项目的参考意义**：它是社区已验证的"教程知识→可视化"映射先例，其页面-章节对应关系可交叉校验我们的分集映射；但其形态是交互式网页，与本项目"连续 2D 动画视频"的产品形态不同，不采纳其技术路线。
3. **动画表达形式**：不单独成集；在相关集中以"社区实践提示"角标提及（如 PagedAttention 集引用其交互页面作延伸阅读）。
4. **推荐补充资料**：llm-engine-viz 站点（kevinez06.github.io/llm-engine-viz）。

---

## 2.26 章节→视频集映射总表（不遗漏校验）

> 依据教程问题链（Part Ⅱ 第 1 章 §4.2）与各章体量排序。每集时长 8–15 分钟；Part Ⅲ/Ⅳ 章节由 SGLang 官方成员编写，视觉上沿用同一体系但信息密度更高。本表只做映射与排期，具体分镜在每集开拍前单独产出（样片为第 1 集）。

| 集 | 主题 | 教程来源（原文章节） | 核心问题（问题链位置） |
|---|---|---|---|
| 1（样片，**定位＝全景导入集**） | 按下回车之后，大模型究竟如何生成一句回答？ | Part Ⅰ 第 1/2 章；Part Ⅱ 第 2/3/4/5 章（概念部分） | 请求如何流转；生成如何正确 |
| 2 | Token 是怎么切出来的？（分词与词表） | Part Ⅱ 第 3 章 §2.1；Part Ⅰ 第 1 章 §5.7 | 参考《万物皆向量》式的下一站 |
| 3 | 训练好的模型为什么会算？（训练 vs 推理） | Part Ⅰ 第 2 章 §3 | 能力从哪来 |
| 4 | Prefill 与 Decode：一个 token 的两次旅程 | Part Ⅰ 第 2 章 §4 | 两个阶段为何特征不同 |
| 5 | GPU 是怎么干活儿的？ | Part Ⅰ 第 3 章 | 硬件基础 |
| 6 | 推理引擎长什么样？（mini-sglang 总览） | Part Ⅱ 第 1 章 | 模型调用≠推理引擎 |
| 7 | 一个请求的旅程（引擎视角完整版） | Part Ⅱ 第 2 章（概念+代码） | 请求流转 |
| 8 | KV Cache：从 O(n²) 到 O(n) | Part Ⅱ 第 4 章 + Part Ⅰ 第 4 章 | 历史计算重复 |
| 9 | HTTP、并发与流式返回 | Part Ⅱ 第 5 章 | 程序无法服务用户 |
| 10 | Continuous Batching 与调度器 | Part Ⅱ 第 6 章 | 多请求互相等待 |
| 11 | Paged KV Cache：显存的虚拟内存 | Part Ⅱ 第 7 章 | 缓存占用与碎片 |
| 12 | RadixAttention：前缀共享 | Part Ⅱ 第 8 章 | 请求间共享前缀 |
| 13 | 多进程与张量并行 | Part Ⅱ 第 9 章 | 单卡容量与算力不足 |
| 14 | 投机解码：先猜后验 | Part Ⅱ 第 10 章 | Decode 逐 token 受限 |
| 15 | 注意力后端：FlashInfer/Triton/FA3/FlashMLA | Part Ⅲ 第 1 章 | kernel 层深入 |
| 16 | 量化与低精度推理 | Part Ⅲ 第 2 章 | 少搬数据 |
| 17 | 分层缓存 HiCache | Part Ⅲ 第 3 章 | 放不下的历史 |
| 18 | 横向扩展：DP Attention / EP / PP | Part Ⅲ 第 4 章 | 更大规模 |
| 19 | PD 分离 | Part Ⅲ 第 5 章 | 两阶段互相干扰 |
| 20 | 用 Cookbook 部署 + Benchmark 入门 | Part Ⅳ 第 1 章 + Part Ⅰ 第 5 章 | 动手验证 |
| 21 | Profiling 与 Trace 分析（PR #13587 实例） | Part Ⅳ 第 2 章 + Part 0 第 1 章 | 怎么找到瓶颈 |
| 22 | 你的第一个 PR | Part Ⅳ 第 3 章 + Community 规范 | 参与开源 |
| 番外 | 部署你的第一个 SGLang 服务（动手实录） | Part 0 第 2 章 | 开课之前 |

**第 1 集（全景导入集）与后续集的防重复分工**：第 1 集按任务要求完整覆盖 11 项教学内容并保留深度，但它同时是全系列的全景导入集。为避免后续集重复播放同一段内容，约定如下分工——
- 第 2 集（分词）讲切分算法（BPE/WordPiece/SentencePiece）与真实词表案例；第 1 集只立"Token≠汉字、tokenizer 只做映射"两个结论，不展开算法；
- 第 3 集（训练 vs 推理）讲能力从哪来；第 1 集只在回顾中一句带过；
- 第 4 集（Prefill/Decode 专集）引入 Roofline、算术强度与 TTFT/TPOT 定量分析；第 1 集只立定性结论与分段 TTFT 标尺；
- 第 8 集（KV Cache 专集）讲工程实现：out_loc/页分配/前缀匹配与后续分页、Radix；第 1 集只讲"缓存什么、为什么、多大、怎么用"；
- 第 9 集（HTTP 专集）完整展开 uid/ack_map/event_map、取消链路、背压与已知清理缺口；第 1 集只做约 20 秒的取消小结；
- 第 5 集（GPU）第 6 集（引擎总览）为第 1 集镜头的深化重制，重制时引用第 1 集画面做"前情提要"而非重讲。

**遗漏校验（v2 修正计数）**：教程共 37 个中文 markdown＝**33 个教学文件**（Part 0 2 个 + Part Ⅰ 5 个 + Part Ⅱ 18 个〔10 个概念篇 + 8 个代码走读篇，第 7 章无独立代码篇〕 + Part Ⅲ 5 个 + Part Ⅳ 3 个）+ **3 个社区文件** + **1 个 README_zh**；WRITING_TEMPLATE.md 为写作规范未纳入。33 个教学文件全部进入上表或样片；概念篇与代码走读篇按单元合并，代码走读特有要点已并入对应单元"代码实现"栏。逐文件核验清单见附录 A。

---

# 第三部分 第一集样片策划

## 《按下回车之后，大模型究竟是如何生成一句回答的？》

| 项 | 设定 |
|---|---|
| 目标时长 | 10 分 30 秒～11 分 30 秒（含无旁白动画段落；压缩方案可到 9 分 30 秒，见 3.3 结尾说明） |
| 画面规格 | 1920×1080，25fps；风格承接参考作者体系：深蓝科技底 + 点阵网格 + 透视地板 + 金色强调（与样片视觉稿同步产出） |
| 叙事主线 | 一个请求 `什么是 KV Cache？`（沿用教程 Part Ⅱ 第 2 章的示例 prompt）从回车到流式返回的完整旅程 |
| 与参考视频的差异 | 参考视频讲"模型内部如何选出下一个字"（采样为主）；本集的差异化核心是**推理引擎视角**：Tokenizer→调度→Prefill/Decode 分工→KV Cache 的工程意义→Detokenizer/SSE 流式返回，四类环节严格分镜 |

## 3.1 教学目标与知识边界

**学完本集观众应能回答：**
1. 按下回车后，请求依次经过哪些站点？各自负责什么？
2. Tokenizer 为什么必要？Token 和汉字是什么关系？
3. Prefill 和 Decode 分别做什么？为什么一个并行一个串行？
4. KV Cache 缓存的是什么、不缓存的是什么、为什么会越来越大？
5. logits 和概率的区别？温度/Top-k/Top-p 在哪一步起作用？
6. 屏幕上"逐字蹦出"的真相是什么？流式返回改变了什么、没改变什么？

**严格区分（本集的准确性红线，对应任务要求）：**

| 概念 | 本集的处理 | 原文依据 |
|---|---|---|
| Prefill | 只发生在引擎内：并行处理整段 prompt、建初始 KV Cache、产出**第一个**输出 token | （原文：Part Ⅰ 第 2 章 §4.1–4.2；Part Ⅱ 第 3 章 §4.1） |
| Decode | Prefill 之后：每步只输入上一个生成的 token、读缓存、追加缓存、串行循环 | （原文：Part Ⅰ 第 2 章 §4.1；Part Ⅱ 第 3 章 §4.2） |
| 模型计算 | engine 的 forward+sample，"scheduler 决定算谁，engine 负责怎么算" | （原文：Part Ⅱ 第 2 章 §2.3） |
| 请求调度 | scheduler 独立分镜：等待队列、组批、prefill 优先；明确"队列里有请求≠这一轮一定算它" | （原文：Part Ⅱ 第 6 章 §4.2） |
| 服务端流式返回 | API Server+SSE 分镜：改善首段等待体验、不减少模型前向次数、不保证整段更快；`[DONE]` 是接口约定非 SSE 标准 | （原文：Part Ⅱ 第 5 章 §6） |
| 采样 | 同一套规则同时作用于 Prefill 首 token 与 Decode 每步；展开细节留给后续集 | （原文：Part Ⅱ 第 3 章 §3） |

**本集不展开**（预告而非展开）：GPU 执行细节、Roofline、Paged/Radix 缓存管理、投机解码、PD 分离——各留一句"后面的集数会讲"。

## 3.2 知识点—原文对照表（样片内容全部有出处）

| # | 样片知识点 | 出处（教程） | 备注 |
|---|---|---|---|
| K1 | 请求四站：API Server→Tokenizer→Scheduler+Engine→Detokenizer→SSE 回传 | Part Ⅱ 第 2 章 §2（图 2-1） | 站点顺序、职责与原文一致；"四站≠四个进程"也照讲 |
| K2 | uid 请求标识贯穿链路 | Part Ⅱ 第 2 章 §2.1；第 5 章 §3.2 | 画面用"取号牌"视觉隐喻【教学推演】 |
| K3 | tokenizer 切分+映射 id；只做映射不理解语义；Token≠汉字 | Part Ⅱ 第 3 章 §2.1；Part Ⅰ 第 1 章 §5.7 | "deepseek 一个 token≈1.6–1.7 汉字"为原文数字，但属其他模型，样片只说"取决于分词器规则"避免误导 |
| K4 | embedding 查找表 `[vocab_size, d_model]`，id=123 取第 123 行；静态向量不含上下文 | Part Ⅱ 第 3 章 §2.1 | 词表尺寸挂【参考视频实测】徽章：Qwen3-0.6B 词表 151,936 行（来源：阿栗学长 BV1jdpS6tEMQ，B 站 pubdate 时间戳 1791766543 ≈ 2026-10；观看日期 2026-11-15）。教程未给出 0.6B 词表数字，画面注明来源 |
| K5 | Q/K/V 语义；Q 查 K 匹配、V 加权汇总；Causal Mask 负无穷 | Part Ⅱ 第 3 章 §2.3；Part Ⅰ 第 1 章 §4.2 | — |
| K6 | Block=注意力+FFN；残差+Norm 堆 N 层；final norm | Part Ⅱ 第 3 章 §2.4 | 层数样片用 Qwen3-0.6B 的 28 层【网络补充：参考视频实测画面】，或改说"几十层" |
| K7 | Final Norm → LM Head → logits（未归一化分数）；推理只取最后一个位置 | Part Ⅱ 第 3 章 §2.4–2.5、代码篇 §4.1 | Final Norm 在 LM Head 之前（讲稿第 5 节与 S14 已体现） |
| K8 | Prefill 三件事：并行整段+建 KV Cache+出首 token | Part Ⅱ 第 3 章 §4.1 | TTFT 两种口径分列（见 S16）：**Benchmark TTFT（教程口径）**＝请求发出→第一个输出 Token 生成，包含排队与 Prefill；**客户端首段显示时间（first visible chunk）**＝请求发出→浏览器收到可显示文本，另含 Tokenizer/Detokenizer/网络/SSE 开销。两者不得混写为同一指标 |
| K9 | KV Cache：按层存 K/V；Q 不缓存；随生成追加 | Part Ⅰ 第 4 章 §2.1；Part Ⅱ 第 4 章 §3 | — |
| K10 | KV 显存公式与 Qwen3-4B-Thinking-2507 数字（L=36,n_kv=8,d_h=128,bf16；每 token 147,456 字节；100 token≈14.06；101 token≈14.20；300 token≈42.19；32K 单请求≈4.5G；8×32K≈36G） | Part Ⅰ 第 4 章 §4 | 画面挂"**原文计算/理论稠密占用**"徽章（非实测）；数值按 1024 进位（原文记作 MB/GB，即 MiB/GiB），画面角标注明；注明 B=1、不含权重/激活/碎片/对齐与引擎额外开销 |
| K11 | Decode：只输入上一个 token；K/V 追加、Q 读历史；memory-bound 直觉（读权重+缓存） | Part Ⅰ 第 4 章 §2.1；Part Ⅰ 第 2 章 §4.3；Part Ⅰ 第 3 章 §4.3 | 本集不引入 Roofline 公式，只给"瓶颈在搬运"的定性结论 |
| K12 | 采样全链顺序：logits → **Temperature 缩放 logits**（T<1 差距拉大/T>1 抹平/T→0 贪心）→ **Softmax** → Top-k/Top-p 截断 → 对保留候选重新归一化 → Sampling | Part Ⅱ 第 3 章 §3.2 | 顺序以本表为准，与讲稿第 8 节、S24、§3.6.3 第 5 条一致 |
| K13 | 停止条件：EOS / 生成长度上限 / 最大上下文长度 | Part Ⅱ 第 3 章 §3.3 | 字段名随接口而异：概念讲解用 max_new_tokens，mini-SGLang `SamplingParams.max_tokens` 与 OpenAI 兼容请求用 `max_tokens`——语义均为生成长度上限，以所讲对象的代码/API 定义为准（S23 挂此角标） |
| K14 | detokenizer 保留解码状态：Token 边界可能落在某个 UTF-8 字符的字节序列中间（部分字节序列），单独解码出现替代字符或不完整文本；只放行完整新增部分 | Part Ⅱ 第 2 章 §2.4 | 表述统一为"部分 UTF-8 字节序列"，不使用"半个字节/左半字节" |
| K15 | SSE 逐块返回、`data: {...}`、`[DONE]` 约定；流式改善 TTFT 体验不减少计算量 | Part Ⅱ 第 5 章 §6 | — |
| K16 | 取消：断开→AbortMsg→AbortBackendMsg→scheduler 回收；协作式取消两时刻 | Part Ⅱ 第 5 章 §5.2 | 本集只做 20 秒小结，细节留第 9 集 |
| K17 | 调度：等待队列、prefill 优先、KV 容量与预算检查的存在 | Part Ⅱ 第 6 章 §4–5 | 一句带过+地图角标，不展开 |

## 3.3 A. 完整中文解说稿（可直接用于 TTS / 人工配音）

> 语速按 4.2 字/秒估算；〔〕内为制作提示，不朗读；每节末尾标注该节预计时长（含动画停顿）。
> **时长口径声明（v2.1）**：本文出现的全部时间码（章节区间、S01–S30 镜头时间、旁白秒数、停顿秒数）均为 **T0 估算基线**，仅供排期与评审；字数推算与分镜区间之间存在未闭合的间隙（约 100 秒），属已知问题。**禁止以本估时锁码**：进入 G2 后先用 TTS/人工录音生成完整旁白音频，导出逐句时间戳，再按真实音频回填各节时长与 S01–S30 镜头区间；回填要求——①镜头时间码连续闭合（相邻镜头首尾相接、无空洞无重叠）；②纯动画停顿、转场、字幕停留单独标注；③S01–S30 总时长＝成片总时长。

### 第 1 节 开场（0:00–0:40，约 32 秒旁白）

你有没有留意过，在对话框里按下回车之后，回答不是一下子出现的——是一个字一个字蹦出来的。〔token 链逐字弹出〕
这既不是打字机特效，也不是谁在慢慢打字。在这背后，是一整套系统在精确接力。
今天，我们把这个过程从头到尾拆一遍：从你按下回车，到第一个字出现在屏幕上，再到它写完、自己停下来——中间，到底发生了什么？〔大标题落下〕

### 第 2 节 全景地图（0:40–1:30，约 40 秒旁白）

先看一张地图。你的问题，要穿过四个站点。〔流水线全景展开〕
第一站，前台：API Server，负责接收请求、给它发一个编号。〔取号牌动画〕
第二站，Tokenizer，把你的文字切成模型认识的小块。
第三站是核心：调度器和引擎。调度器决定这一轮算谁，引擎负责真正地算。〔两条 lane 分色〕
第四站，Detokenizer，把算出来的小块再变回文字，由前台通过一条一直开着的连接，一段一段发回你的浏览器。〔SSE 波纹〕
提醒一句：这里是**四类逻辑组件**——API Server 在请求入口和响应出口各出现一次，所以路径图上有五个节点，但它们不是五个独立程序。接下来，我们跟着这句话——"什么是 KV Cache？"——一站一站走。〔prompt 落入输入框，回车键按下〕

### 第 3 节 Tokenizer（1:30–2:30，约 44 秒旁白）

第一站，Tokenizer。模型不认识文字，只认识数字。〔文字被栅栏切成 token 块〕
它干两件事：先把文本切成小块，每一块叫一个 Token；再按词表，把每个 Token 换成一个编号。〔token 块翻面出现编号〕
注意一个容易误会的地方：一个 Token，不一定是一个字。它可能是一整个词，也可能只是词的一部分，甚至只是一个标点。切多细，取决于分词器的规则。〔"什么"合体、"Cache"分裂演示——教学示意角标〕
Tokenizer 只做映射，它不理解语义，也不参与计算。〔原文结论字幕〕
切完之后，你的问题就从一句话，变成了一串编号。从这一刻起，进入模型的，是数字。〔token 块整体翻面成数字串〕

### 第 4 节 调度器（2:30–3:10，约 32 秒旁白）

第二站，调度器。如果服务器同时收到很多请求，先算谁、后算谁、几个人拼成一拨一起算——这是它的事。〔多个请求排队，调度器灯光扫过〕
教程的实现采用这样的规则：能组出新请求的处理批次，就优先处理；否则，让正在生成的请求，继续算下一个 Token。〔优先级闸门动画〕
而且，队列里有请求，不等于这一轮就能算它——还得检查缓存空间够不够。〔闸门上加一把容量锁〕
我们的这句话排到了队首。现在，轮到它上场。下一站，就是整个推理最核心的一步——Prefill。〔请求飞入引擎大门〕

### 第 5 节 Prefill（3:10–5:40，约 95 秒旁白 + 15 秒纯动画停顿）

Prefill，是什么？模型把你的整段输入，一次性送进去，并行处理所有 Token。〔整段 token 块同时点亮〕
它内部有三步。
第一步，查表。每个编号，到一张嵌入表里取出一行向量——这张表有多少行，就是词表有多大。〔大表弹出，若干行被取出〕此时，每个向量只带着"我是谁"，还不知道上下文。〔向量之间虚线问号〕
第二步，进 Transformer。进注意力之前，还差一样东西：位置。向量本身不知道自己排第几位——现代模型常用 RoPE：在注意力里，对 Q 和 K 做一次旋转，让匹配分数带上"你在第几位"的信息。〔环形刻度盘给 Q/K 各转一个角度〕
一层内部，是一条四步的小流水线。〔层平面展开〕
第一小步，归一化：先把输入稳定到统一尺度。〔小_norm 盒〕
第二小步，注意力：让每个 Token 抬头看别人——这叫注意力。〔弧线生成〕每个 Token 会生成一份查询 Q、一份钥匙 K、一份内容 V。〔Q/K/V 三色卡片〕当前 Token 的 Q，和**当前位置以及之前**所有位置的 K 比对——包括它自己；越匹配，权重越大；再按权重，把它们的 V 加权汇总。〔点积连线：最后一块的 Q 连到 1～5 号位的 K，自身 K 高亮描边〕
同时有一个因果掩码，把未来位置的分数设成负无穷——谁也不能偷看后面。〔右上三角变暗〕
第三小步，残差相加：把注意力的结果，原样加回主路——给信息留一条直通车道，层数再深也不容易走丢。〔⊕合流〕
第四小步，再归一化、再过前馈网络——每个 Token 自己消化——最后再残差相加。〔第二组 norm+FFN+⊕〕
这四小步合起来，是一层；真实模型，会堆几十层。〔层体块快速堆叠〕
第三步，只取最后一个位置。整段算完之后，先经过 Final Norm，把输出稳定到统一尺度；再由 LM Head，投影到整个词表上——得到每个候选 Token 的分数。这个分数，叫 logits。〔Final Norm 小盒一闪 → 一列分数条从最后位置升起〕
注意：logits 不是概率——它还没有归一化，总和不是一。〔"概率"字样打叉〕
所以，Prefill 干了三件事：并行处理完整提示词；建立后面要用的 KV Cache；产出第一个新 Token。〔三件事打勾〕
衡量"等多久"，要看两个时刻。第一个：从请求发出，到第一个输出 Token 生成——教程里叫 TTFT，它包含排队和 Prefill；Prefill 通常是大头，但 TTFT 不等于 Prefill 耗时。第二个：到你屏幕上真正冒出文字——中间还要经过 Detokenizer、网络和 SSE，这是客户端的首段显示时间。两个时刻，不是同一个指标。〔双刻度标尺出现：请求发出 → ①首 Token 生成（Benchmark TTFT） → ②首个可显示文字（客户端显示时间）〕
至于这个分数怎么变成一个确定的 Token——先按下不表，我们在 Decode 那里一起讲。〔悬念转场〕

### 第 6 节 KV Cache（5:40–6:50，约 52 秒旁白 + 18 秒纯动画停顿）

这里必须停下来，讲一个关键角色：KV Cache。〔镜头拉向层平面旁的缓存架〕
想一想：如果每生成一个新字，都要把前面所有字从头重算一遍——第一个字算一遍，第二个字算两遍，第十个字算十遍——越长越亏。〔重复计算的累加演示〕
所以引擎会把每一层算出来的 K 和 V，存进一张缓存。这就是 KV Cache。〔K/V 方块逐层落入缓存架〕
三个要点。第一，只存 K 和 V，不存 Q——Q 只在当前这一步用，历史的 Q，以后用不上。〔Q 用完即散，K/V 落架〕
第二，每一层都有自己的缓存——几十层，就是几十份。〔缓存架按层分格〕
第三，它随生成不断追加。〔新 K/V 亮起落入下一格〕
它有多大？教程里算了笔账：以一个 36 层、8 个 KV 头、每头 128 维、bf16 存储的模型为例——100 个 Token 的缓存，大约 14 兆字节；每多生成一个 Token，大约再增加 0.14 兆。〔徽章：原文估算/理论计算；数字条按 1024 进位〕
听起来不大。但它是每个请求都长期占着显存的历史状态。上下文拉到 32K，再乘上并发，KV Cache 就会超过模型权重本身，成为显存的大头。〔显存蛋糕：KV 占比反转动画〕
后面几集所有的缓存优化——分页、前缀共享——都是从这个"大头"开始的。〔预告角标〕

### 第 7 节 Decode（6:50–8:00，约 55 秒旁白 + 15 秒纯动画停顿）

第一个 Token 出炉之后，就进入 Decode 阶段。〔阶段切换：lane 变色〕
规则变得非常简单：每一步，只把上一步生成的那个 Token，重新送进模型。〔单 token 块进入引擎〕
它先查表变成向量；然后在每一层，算自己的 Q、K、V——K 和 V，追加进缓存；Q，拿去和缓存里所有的历史 K 比对，读出历史的 V 加权求和。〔读缓存箭头从缓存架拉回注意力〕
最后，还是只看这个新位置的 logits。〔新分数条〕
对比一下：Prefill 是很多 Token 并行、一次算完；Decode 是每次只有一个新 Token，串行往前爬。〔双 lane 对比定格〕
它的计算量看起来不大，但每一步，都要执行模型全部几十层的计算：把每一层的权重读进计算单元，让当前 Query 读取该层的历史 K/V，再把新 K/V 写回缓存。对小批量 Decode 来说，这些数据搬运的开销，往往比计算本身还大——所以瓶颈常常不在算力，而在显存带宽。〔逐层小包裹动画：权重进层 → 读历史 K/V → 写回新 K/V〕
这样一步一步，直到满足停止条件：采到结束符 EOS——那个"不代表任何字、只代表到此为止"的特殊 Token；或者，达到 max_tokens 上限；或者，顶到模型的最大上下文。〔三种停止条件开关依次点亮〕

### 第 8 节 采样（8:00–9:00，约 44 秒旁白 + 16 秒纯动画停顿）

现在回答刚才按下不表的问题：logits，是怎么变成一个确定的 Token 的？顺序很重要，一共三步。
第一步：如果设置了温度 T，先把每个分数除以 T——温度低于 1，分数之间的差距被拉大；高于 1，差距被抹平；趋近 0，就退化成贪心——每一步只选第一名。〔旋钮先作用于 logits 分数条〕
第二步：再过 Softmax，把调整后的分数，变成加起来等于 1 的概率。〔概率柱成形〕
第三步：Top-k 和 Top-p 负责砍掉低概率的长尾——Top-k 固定保留前 k 名；Top-p 动态保留累计概率达到 p 的最小集合；然后，在剩下的候选里重新归一化，再抽。〔青色竖线切走尾部，剩余柱重排〕
这套规则，对 Prefill 出的第一个 Token，和 Decode 的每一步，都一样。〔两 lane 上同一旋钮复用〕
把温度和截断讲透，需要一整集——这里先立住结论。〔该集预告角标〕

### 第 9 节 Detokenizer 与流式返回（9:00–10:20，约 58 秒旁白 + 12 秒纯动画停顿）

最后两站，收尾。
Detokenizer，把新 Token 变回文字。但它不能一个 Token 一个 Token 单独解码、再拼起来——一个汉字，通常由三个字节编码；Token 的边界，可能正好落在字节序列的中间。只拿着不完整的字节序列去解码，就会冒出替代字符。〔字节分段示意：一个汉字＝3 个字节，被两个 Token 分段承载〕
所以它给每个请求保留一份解码状态：确认拼完整了，才把新增的那几个字，放行。〔缓冲区攒字→放行动画〕
放行之后呢？前台有一条一直开着的 HTTP 连接，用 SSE，一段一段往浏览器推数据——每一段带着新增文字，浏览器边收边显示。〔SSE 波纹逐段推进，网页逐字浮现〕
这就是"一个字一个字蹦出来"的真相：不是特效——服务端每生成新的 Token，都会先交给 Detokenizer 拼接；一旦拼出安全、可显示的文字，就立刻通过连接推送。一次推送，可能对应一个或多个 Token。〔幕后/幕前对照〕
要说清楚的是：流式返回，改善的是你看到首段内容的等待体验；它不会减少模型的前向计算，也不会让整段回答更快生成。〔结论字幕〕
当你点下"停止"，取消信号也要从浏览器一路传回调度器，把没算完的请求停下、把占用的显存还回去——这叫协作式取消；"点击停止"和"资源回收完成"，是两个时刻。〔取消信号逆流动画〕

### 第 10 节 回顾与预告（10:20–11:10，约 42 秒旁白）

回头看这一路。〔整条流水线从右往左快速重放，每站闪一次结论字〕
你按下回车——前台发号排队——Tokenizer 把文字切成编号——调度器安排它上场——Prefill 把整段问题并行算完，建好 KV Cache，产出第一个 Token——Decode 每次只喂一个新 Token：读缓存、追加缓存，一步一步生成——每一步的 logits，经过温度和截断，变成一次选择——Detokenizer 拼好文字，SSE 一段段送回你的屏幕——直到 EOS，它才真正停笔。〔EOS 紫色方块落下，光标停止〕
每一个字，都是这套系统算出来的——也是抽出来的。
下一集，我们把镜头怼到第一站：Token 到底是怎么切出来的？为什么同一个词，有时是一整块，有时被切成两块？〔凋｜零分裂预告画面〕
我们下期见。

**时长压缩方案（如需 9 分 30 秒版本）**：第 8 节压缩为 25 秒（只保留温度一句+Top-p 一句，详细版移至采样专集）；第 9 节取消部分压缩为 12 秒（只留"两个时刻"结论）；第 6 节纯动画停顿减半。合计可省约 70 秒。**不建议**删减 KV Cache 要点与 Prefill/Decode 对比——这两处是本集区别于参考视频的核心增量。

## 3.4 B. 视频分镜表

> 符号约定：〔旁白触发〕= 该动画由旁白对应句子的起始词触发；舞台坐标以画面百分比描述。全片为连续舞台，除 S30 外无黑场硬切。
> **时间码性质（v2.1）**：下表时间列为 T0 估算基线，G2 配音后按 3.3 头部的"时长口径声明"回填并保证闭合。
> 舞台分区（贯穿全片）：A 区=浏览器窗口（画面上部，模拟用户视角）；B 区=服务端流水线（画面中部主舞台）；C 区=字幕条（底部）；D 区=角标区（右上，放置"教学示意 / 原文出处 / 原文计算 / 参考视频实测 / 本项目实测 / 系列预告"徽章，六类见 3.5.3 S-02）。

### 3.4.1 开场与全景（S01–S05）

| 镜头 | 时间 | 讲解内容（旁白句） | 画面构图 | 动画过程（含触发时机） | 屏幕文字 | 音效 |
|---|---|---|---|---|---|---|
| S01 | 0:00–0:12 | "你有没有留意过……一个字一个字蹦出来的" | A 区全屏：深蓝底聊天窗口，输入框居中偏下，光标闪烁 | 〔旁白"按下回车"〕回车键下沉；〔"蹦出来"〕回答 token 链"KV Cache 是给模型记笔记的本子，" 逐块弹出（7 块，块间距 8px，弹出 80ms/块），光标随行 | 字幕；输入框内 prompt"什么是 KV Cache？"；D 区：**示例回答·教学示意**（非真实模型输出；如需标"本项目实测"须真实运行并留存记录） | 回车"嗒"；token 弹出"嗒嗒嗒" |
| S02 | 0:12–0:24 | "这既不是打字机特效……精确接力" | 同 S01，token 链放大 | 〔"不是打字机特效"〕打字机图标飞入→打红叉碎裂；〔"精确接力"〕镜头拉远，A 区缩小到上方 1/4，B 区流水线骨架（五个站牌虚框）淡入 | "不是特效"（金色大字，0.5s 后淡出） | 打叉 buzz；拉远"唰" |
| S03 | 0:24–0:40 | "今天……到底发生了什么？" | B 区中央 | 〔旁白起〕大标题金属字落下压场 1.5s；〔"中间到底"〕标题上移缩小成左上角系列徽标 | 大标题：《按下回车之后，大模型究竟如何生成一句回答？》；副标：SGLang 视角的第一课 | 标题"咚" |
| S04 | 0:40–1:05 | "先看一张地图……第一站前台……发号" | B 区全景：横向流水线含**五个路径节点**（API Server(入口)→Tokenizer→Scheduler+Engine 双门→Detokenizer→API Server(出口) 回环箭头至 A 区），节点下方挂"四类逻辑组件"分组括线 | 〔"第一站"〕站牌逐个点亮（左→右，每站 600ms）；〔"发一个编号"〕请求卡片从浏览器落入 API Server，弹出"号牌 #42"（uid 视觉隐喻【教学推演】角标注明） | 站牌：前台 / Tokenizer / 调度+引擎 / Detokenizer；分组括线："四类逻辑组件，五个路径节点——不是五个独立程序"；角标：原文 Part Ⅱ 第 2 章 | 点亮"叮"×5；落卡"啪" |
| S05 | 1:05–1:30 | "第三站是核心：调度器决定算谁，引擎负责怎么算……四类组件是分工，不一定对应四个程序" | 同 S04 | 〔"调度器决定"〕调度门与引擎门分色（蓝/金），门间标"同一进程（教学简化）"；〔"一段一段发回"〕Detokenizer 到 A 区之间画出 SSE 波纹通道；〔"跟着这句话"〕prompt 文字球沿流水线上方轨道移动就位 | "调度器决定算谁，引擎负责怎么算"（双色字幕条）；"四个路径节点≠四个进程（API Server 出入口各计一次）"；D 区：教学简化徽章 | 分色"叮咚"；文字球滚动声 |

### 3.4.2 Tokenizer 与调度（S06–S09）

| 镜头 | 时间 | 讲解内容 | 画面构图 | 动画过程 | 屏幕文字 | 音效 |
|---|---|---|---|---|---|---|
| S06 | 1:30–1:50 | "模型不认识文字，只认识数字……先把文本切成小块" | Tokenizer 站放大至 60% 画面 | 〔旁白起〕镜头推近；〔"切成小块"〕"什么是 KV Cache？"七字被下落栅栏线切成 token 块（什么｜是｜ KV｜ Cache｜？——教学示意切分，D 区角标"切分规则因模型而异"）；未切完前块间无缝隙，切完缝隙弹开 6px | "Token＝模型的最小处理单元"；角标：教学示意 | 栅栏"咔"×2；切分"嗒"×7 |
| S07 | 1:50–2:12 | "再按词表，把每个 Token 换成一个编号……不一定是一个字" | 同 S06，右侧滑入词表抽屉 | 〔"换成一个编号"〕每块 token 翻面（Y 轴 180°）露出编号（用教学编号 id₁…id₅，D 区角标"非真实 ID"）；〔"不一定是一个字"〕三组对照小动画并排闪现：整词一块 / 词的一部分一块 / 单个标点一块 | "Token ≠ 汉字"；"词表（vocab）：全部编号的总表"；角标：原文 Part Ⅰ §5.7 | 翻面"唰"×5 |
| S08 | 2:12–2:30 | "Tokenizer 只做映射，不理解语义……变成了一串编号" | 同 S06 | 〔"只做映射"〕词表抽屉与 token 块之间画两条单向箭头后，任何"语义"试图出现即被灰化；〔"变成一串编号"〕token 块整体翻面成数字串 `id₁ id₂ id₃ id₄ id₅`，飞入请求卡片 | "只做映射，不理解语义，不参与计算"（原文结论，白底金框） | 灰化"嗡"；飞入"嗖" |
| S09 | 2:30–3:10 | "第二站，调度器……队列里有请求，不等于这一轮就能算它" | 调度门放大至 60% | 〔旁白起〕三个请求卡片（#41/#42/#43）落入等待队列竖排；〔"先算谁"〕调度器探照灯扫队列；〔"优先处理"〕#42 卡片穿过"Prefill 优先"闸门；〔"检查缓存空间"〕闸门上方出现容量锁图标并亮绿灯；〔"轮到它上场"〕#42 飞入引擎大门 | "队列 ≠ 立即执行"；"Prefill 优先（本教程版本的规则）"；角标：原文 Part Ⅱ 第 6 章 | 扫描"滴"；闸门"咔哒"；锁"叮" |

### 3.4.3 Prefill 三步（S10–S16）

| 镜头 | 时间 | 讲解内容 | 画面构图 | 动画过程 | 屏幕文字 | 音效 |
|---|---|---|---|---|---|---|
| S10 | 3:10–3:30 | "Prefill 是什么？整段输入一次性送进去，并行处理所有 Token" | 引擎门内：Prefill 车间，5 个 token 块横向排开 | 〔旁白起〕大门打开露出车间；〔"一次性送进去"〕5 块同时加速冲入并排；〔"并行"〕5 块同时亮金边脉冲一次（区别于后续 decode 的单块脉冲） | "Prefill：一次完整的前向"；"并行处理所有输入 Token" | 大门"轰"；并行脉冲"嗡" |
| S11 | 3:30–3:55 | "第一步，查表……每个向量只带着'我是谁'" | 嵌入表特写：右侧竖排行号 id₁…id₅，左侧行向量格子（示意 12 列×5 行色块，D 区角标"真实为 1024 维"） | 〔"查表"〕5 根探针同时从 5 个编号射向对应行（并行，一次到位）；〔"取出一行"〕5 行高亮后复制副本沿轨道飞出；〔"还不知道上下文"〕副本间浮现虚线问号弧 | "嵌入表 E：行数＝词表大小"；角标：原文 Part Ⅱ 第 3 章 §2.1；D 区徽章：**参考视频实测**·词表 151,936 行（Qwen3-0.6B；来源：阿栗学长 BV1jdpS6tEMQ） | 探针"叮"×5（快速连音）；复制"唰" |
| S12 | 3:55–4:35 | "进注意力之前先补位置（RoPE）……这叫注意力……Q/K/V……当前以及之前……加权汇总" | 层平面#1 放大：token 向量 5 个横排，上方三张参数矩阵卡 W_Q/W_K/W_V，入口处一只环形刻度盘 | 〔"位置/RoPE"〕5 个向量先过环形刻度盘：Q/K 卡片各被旋转一个与位置对应的角度（V 不转），角标"RoPE：对 Q/K 施加与位置相关的旋转，让点积携带位置信息（原文 Part Ⅱ 第 3 章 §2.2）"；〔"抬头看别人"〕旋转后的 5 个向量头顶各分裂出三张小卡：Q 金、K 蓝、V 绿；〔"当前以及之前的 K 比对"〕最后一块（5 号位）的 Q 发出弧线连到 **1～5 号位（含自身）** 的 K，自身 K 描金边，弧线粗细=匹配度（教学权重示意）；〔"加权汇总"〕各块 V 卡按权重缩放后滑向汇总槽叠加成一个混合色向量 | "RoPE：旋转 Q/K 注入位置信息"；"看别人＝注意力（Attention）"；"Q 我在找什么 / K 我是什么特征 / V 我携带什么"；"可见范围＝当前位置及之前（因果）"；角标：原文 Part Ⅱ 第 3 章 §2.2–2.3 | 刻度盘"咔"；分裂"叮叮叮"；弧线"咻"；汇总"咚" |
| S13 | 4:35–4:55 | "因果掩码……谁也不能偷看后面……残差相加……再归一化、前馈网络、再残差相加" | 同 S12 俯视角：5×5 分数小矩阵 + 层内四步小流水线侧视图（norm→Attn→⊕→norm→FFN→⊕，Pre-Norm 布局） | 〔"因果掩码"〕矩阵右上三角盖红色幕布（幕布上写 −∞；对角线可见=含自身）；〔"残差相加"〕侧视图主路 H 直通⊕节点，注意力输出从支路汇入（⊕亮金）；〔"再归一化、FFN、再残差相加"〕第二组 norm→FFN→⊕ 依次点亮；四步完成后整层平面亮边并向上叠加一层半透明层（暗示堆叠），D 区角标"真实 Block 为 Pre-Norm 结构；本图为教学简化，Residual/Norm 细节在后续集展开" | "Causal Mask：未来位置＝−∞（对角线可见）"；"残差：Output＝Input＋Layer(Input)，给信息留直通车"；"一层＝Norm→Attn→⊕→Norm→FFN→⊕（Pre-Norm）"；角标：原文 Part Ⅱ 第 3 章 §2.4（`h＝x+Attention(RMSNorm(x))`、`out＝h+MLP(RMSNorm(h))`） | 幕布落下"嚓"；⊕"咚"×2；叠层"咔" |
| S14 | 4:55–5:15 | "第三步，只取最后一个位置……先过 Final Norm，再由 LM Head 投影到词表空间——得到 logits" | 层堆顶部：最后位置的向量特写 + Final Norm 小盒 + LM Head 扇形展开 | 〔"只取最后一个位置"〕镜头推到最后一块，前 4 块灰化下沉；〔"Final Norm"〕向量先过 final norm 小盒（角标：原文 Part Ⅱ 第 3 章 §2.4/代码篇 self.norm）；〔"LM Head"〕扇形展开成词表候选条（画 12 根示意柱+省略号，D 区角标"真实 15 万+候选"）；柱高=分数（未归一化，柱顶数字标注 s₁…s₁₂ 教学值） | "只取最后一个位置"（金色大字）；"Final Norm → LM Head：隐藏状态 → 词表分数"；"logits＝未归一化的原始分数"；角标：原文 Part Ⅱ 第 3 章 §2.4–2.5 | 灰化"噗"；norm"叮"；展开"唰" |
| S15 | 5:15–5:28 | "logits 不是概率——还没有归一化" | 同 S14 | 〔旁白起〕"概率"字样从右飞来盖到柱顶→红叉碎裂；柱组上方总和标注闪红"Σ≠1" | "logits ≠ 概率"（红字）；D 区：归一化见第 8 节 | 碎裂 buzz |
| S16 | 5:28–5:40 | "所以 Prefill 干了三件事……两个时刻：TTFT 与客户端首段显示时间" | 车间全景 | 〔旁白起〕三枚勾选徽章依次盖章：①并行处理整段 ②建立 KV Cache ③产出第一个新 Token（第③枚时第 1 个 token 块从 LM Head 弹出滚到车间出口）；〔"两个时刻"〕A 区与车间之间拉出**双刻度标尺**：`请求发出 →① 首 Token 生成（Benchmark TTFT：含排队+Prefill，教程口径） →② 首个可显示文字（客户端首段显示时间：另含 Detokenizer/网络/SSE）`，①②之间留出可见间隔（角标"流程示意，不按耗时比例"） | "Prefill 三件事"（三条清单）；"TTFT（Benchmark 口径）＝请求发出→首个输出 Token 生成"；"客户端首段显示时间＝请求发出→浏览器收到可显示文本"；"两个时刻 ≠ 同一指标"；角标：原文 Part Ⅱ 第 3 章 §4.1 / Part Ⅰ 第 5 章 §3.1 | 盖章"啪"×3；标尺"叮"×2 |

### 3.4.4 KV Cache 专讲（S17–S19）——本集最重镜头，全部细节按任务要求列出

| 镜头 | 时间 | 讲解内容 | 画面构图 | 动画过程（精确到触发词与元素数量） | 屏幕文字 | 音效 |
|---|---|---|---|---|---|---|
| S17 | 5:40–6:05 | "想一想：如果每生成一个新字，都要把前面所有字从头重算一遍……越长越亏" | 分屏左：车间第 1 层平面（固定 5 个 token 块）；分屏右：工作量计数器 | 〔旁白起〕镜头拉出第 1 层平面；〔"从头重算一遍"〕右侧演示三连帧：第 1 步亮 1 块、第 2 步亮 2 块（第 1 块重复亮起并标"又算了一遍"）、第 3 步亮 3 块（前两块重复标"又算了一遍"）；计数器同步跳 1→3→6；第 10 步预测条到达 55 并打红 | "没有缓存：历史 K/V 每一步都重算"；计数：Σ=1+2+3+…；角标：**Σ 仅指"历史 K/V 重复投影"的累计**（原文 Part Ⅱ 第 4 章 §4.1：Σt=O(n²)）——不代表使用 KV Cache 后总计算量降为 O(n)：每步 Query 仍要与历史 K 做 Attention，跨 Decode 步累计的注意力交互仍约 O(n²) | 重复点亮"咚咚"加重；预测条警"叮叮" |
| S18 | 6:05–6:30 | "所以引擎会把每一层算出来的 K 和 V 存进缓存……三个要点" | 车间右侧立起"KV Cache 分层架"：横向 8 个格子（对应 token 位），纵向按层分组（示意 3 层×3 组，D 区角标"真实 28/36 层"）；第一层平面仍可见 | 〔"存进缓存"〕S12 产生过的 K 卡（蓝）与 V 卡（绿）从注意力下方逐个滑出，按 token 位置落入本层对应格子（第 1 层 5 个位置：K₁V₁…K₅V₅ 共 10 张小卡，先 K 后 V 成对落下）；〔要点一〕Q 卡（金）试图跟随落入→被格口弹回→碎裂消失，K/V 落定发光；〔要点二〕镜头纵向拉开：每层都有同款格子组，逐层填入（快速连放）；〔要点三〕第 6 格（新 token 位）虚框等待 | "KV Cache：按层保存历史 K/V"；三要点清单：①只存 K/V，不存 Q ②每层一份 ③随生成追加；角标：原文 Part Ⅰ 第 4 章 §2.1 | 落卡"嗒"×10（成对）；弹回碎裂 buzz；逐层"哒哒哒" |
| S19 | 6:30–6:50 | "它有多大？……大约 14 兆字节，每多生成一个 Token 约再增加 0.14 兆……超过模型权重本身" | 数字面板：左侧显存蛋糕（权重饼+KV 饼），右侧公式条 | 〔旁白起〕公式逐项点亮：`2(K和V) × 36层 × 8 KV头 × 128维 × 2字节 ＝ 147,456 字节/Token`；〔"14 兆字节"〕阶段一计数：100 输入 + 1 输出＝101 Token → 约 **14.20**（数字滚动，单位条同步）；〔"再增加"〕阶段二：100 输入 + 200 输出＝300 Token → 约 **42.19**；〔"超过模型权重"〕蛋糕重切动画：上下文滑块拉到 32K×8 请求，KV 扇区反超权重扇区（约 36G vs 约 7.45G，原文数字）；画面角落常驻限定条：B=1、理论稠密 KV 占用，不含权重/激活/碎片/对齐与引擎额外开销 | 公式：KV_bytes＝B×T×L×2×n_kv×d_h×s；徽章：**原文计算/理论稠密占用**（Qwen3-4B-Thinking-2507，Part Ⅰ 第 4 章 §4）；单位角标："数值按 1024 进位（即 MiB/GiB），教程原文记作 MB/GB"；"长上下文×并发：KV > 权重" | 公式点亮"叮"；盖章"啪"；蛋糕重切"咔——" |

### 3.4.5 Decode 与采样（S20–S24）

| 镜头 | 时间 | 讲解内容 | 画面构图 | 动画过程 | 屏幕文字 | 音效 |
|---|---|---|---|---|---|---|
| S20 | 6:50–7:10 | "第一个 Token 出炉……进入 Decode。每一步，只把上一步生成的那个 Token 重新送进模型" | Decode lane 启动：单 token 块轨道 | 〔旁白起〕lane 染金色变 Decode 色；第 1 个生成 token 块（从 S16 弹出的那块）掉头沿虚线轨道回到引擎入口；〔"重新送进"〕块入场时只有它一个亮，其余全暗（与 S10 的 5 块同亮形成对照闪回 0.5s） | "Decode：一次一个新 Token"；角标：原文 Part Ⅰ 第 2 章 §4.1 | lane 变色"叮"；入场"嗖" |
| S21 | 7:10–7:40 | "它先查表……每一层算自己的 Q、K、V——K 和 V 追加进缓存；Q 拿去和缓存里所有历史 K 比对，读出历史 V 加权求和" | 第 1 层平面+缓存架同框 | 〔"查表"〕单 token 探针取行（1 根，对比 S11 的 5 根）；〔"算自己的 QKV"〕只有 1 组三色卡生成；〔"追加进缓存"〕K/V 卡滑入缓存架第 6 格（S19 的虚框填实）；〔"Q 和所有历史 K 比对"〕金 Q 发出 6 条弧线：5 条射向缓存架已落的 K₁–K₅（带回读取箭头）、1 条射向刚落的 K₆；〔"读出历史 V 加权求和"〕V₁–V₆ 沿读取箭头反向流回汇总槽 | "读缓存 → 追加缓存"（双向箭头图例）；角标：原文 Part Ⅰ 第 4 章 §2.1 公式 o_{t+1}=softmax(qK₁:t+1ᵀ/√d)V₁:t+₁（小字角标） | 追加"嗒"；弧线"咻"×6；读取回流"唰" |
| S22 | 7:40–7:55 | "对比一下：Prefill 并行一次算完；Decode 串行往前爬……瓶颈通常不在算力，而在搬运" | 分屏对比定格 | 〔旁白起〕左：Prefill lane 回放 0.8s（5 块同进）；右：Decode lane（1 块进→出循环 3 次）；〔"搬运开销"〕改为**逐层**演示一帧循环：第 i 层——权重切片包从显存门进入该层计算单元 → 当前 Query 发弧线读取该层历史 K/V（缓存架该层行高亮） → 新 K/V 写回缓存格；快放 3 层后镜头微拉远露出"逐层累加＝一整步"的字幕。**不画**"整个 KV Cache 作为单块包裹整体搬运"。角标："典型小 Batch Decode 情况；每步读全部权重+读历史 KV 的总量口径见原文 Part Ⅰ 第 2 章 §4.3（权重 14GB+KV 1.5GB/步，LLaMA-7B 例）" | "Prefill：计算密集倾向 / Decode：搬运密集倾向（小 Batch 典型）"；"每层：读权重 → 读历史 K/V → 写新 K/V"；角标：原文 Part Ⅰ 第 2 章 §4.3、第 3 章 §4.3 | 循环"咔嗒咔嗒"；红边警报低鸣 |
| S23 | 7:55–8:00 | "直到满足停止条件：采到 EOS……达到生成长度上限……顶到最大上下文" | 三开关面板 | 〔旁白逐条〕紫色 EOS 方块（无字，印"⏹"）落下点亮开关①；计数器爆表点亮②；长度条顶格点亮③ | "停止条件：EOS / 生成长度上限 / 最大上下文长度"；D 区角标："字段名随接口而异：max_new_tokens（概念）/ max_tokens（mini-SGLang SamplingParams 与 OpenAI 兼容请求）——语义均为生成长度上限"；角标：原文 Part Ⅱ 第 3 章 §3.3 | 三连"叮叮叮" |
| S24 | 8:00–9:00 | "顺序很重要，一共三步：先温度除 logits → 再 Softmax → 再 Top-k/Top-p 截断" | 采样车间：logits 分数柱 + 温度旋钮 + 概率柱 + top-p 竖线（左右两栏：左=分数域，右=概率域） | 〔"第一步·温度"〕旋钮作用于**左栏 logits 分数柱**：T<1 分数柱差距拉大、T>1 差距压平、T→0 只剩一根（D 区角标："温度除在 logits 上，先于 Softmax——原文 Part Ⅱ 第 3 章 §3.2.2"）；〔"第二步·Softmax"〕左栏调整后的分数柱流动到右栏并变形为概率柱（总和条填到 100%）——**左栏未调整前右栏不出现概率柱**；〔"第三步·截断"〕青色累计线在右栏从左往右爬（0.52→0.80 处停），尾部概率柱被斩落并淡出，剩余柱重新归一化（高度重排动画）；〔"都一样"〕Prefill lane 与 Decode lane 各弹出同款三步小图标 | "温度 T：先除 logits（＜1 拉大差距 / ＞1 抹平 / →0 贪心）"；"Softmax：分数 → 加和为 1 的概率"；"Top-k 固定前 k 名 / Top-p 动态累计到 p"；角标：原文 Part Ⅱ 第 3 章 §3.2；D 区：采样专集预告 | 分数变形"唰"；流动"哗"；斩落"嚓"；归一"叮" |

### 3.4.6 Detokenizer、流式与收尾（S25–S30）

| 镜头 | 时间 | 讲解内容 | 画面构图 | 动画过程 | 屏幕文字 | 音效 |
|---|---|---|---|---|---|---|
| S25 | 9:00–9:25 | "Detokenizer 把新 Token 变回文字……Token 边界可能落在 UTF-8 字节序列中间" | Detokenizer 站放大 | 〔旁白起〕token 块送入站内漏斗；〔"字节序列中间"〕特写：汉字"缓"＝3 个 UTF-8 字节（示意字节格 E1/E7/BC），被两个 Token 分段承载——Token A 携带前 2 个字节格、Token B 携带末 1 个字节格；单独解码 Token A → 弹出替代字符"�"（红色抖动，标注"字节序列不完整→替代字符 U+FFFD"）；〔"保留一份解码状态"〕站内为请求 #42 弹出"解码缓冲"小黑板，不完整字节块被黑板吸住不外发，**不画**"汉字从中间物理切开/半个字" | "字节没拼齐，先不发"；"Token 边界 ∝ UTF-8 字节序列边界"；角标：原文 Part Ⅱ 第 2 章 §2.4；D 区：字节切分为教学示意 | 漏斗"唰"；乱码 buzz 抖动 |
| S26 | 9:25–9:45 | "确认拼完整了，才把新增的那几个字放行" | 同 S25 | 〔旁白起〕下一 token 块落入黑板补全（乱码块与它拼合闪金光→变成完整"缓存"二字）；〔"放行"〕完整文字块沿轨道滑出站点 | "拼完整 → 才放行" | 拼合"叮"金光；放行"嗖" |
| S27 | 9:45–10:05 | "前台有一条一直开着的 HTTP 连接，用 SSE 一段一段往浏览器推……拼出安全文字就立刻推送" | A 区浏览器与 B 区同框，SSE 波纹通道激活 | 〔旁白起〕幕后接力环：引擎产出 Token ID → Detokenizer 拼接缓冲 → 拼出安全文本 → 推包（一次推送可含 1..n 个 Token，也可能暂时无新增可显示文本——用"缓冲区攒了 2 块才推 1 包"的演示帧体现）；波纹通道上数据包逐段推进（每包标签 `data: {"delta":"缓存"}` 教学简化包体）；〔"边收边显示"〕A 区聊天窗口文字逐段浮现，与 B 区推包严格同拍 | "SSE：服务器持续推送增量"；"一次推送 ≠ 一个 Token"；"不是特效，是真实到达"；角标：原文 Part Ⅱ 第 5 章 §6；D 区：`[DONE]` 为接口约定非 SSE 标准；"包体为教学简化，非完整 OpenAI Chat Completions 线上结构" | 波纹"哗"×n；显示"嗒"同步 |
| S28 | 10:05–10:20 | "流式返回改善的是首段等待体验；不会减少模型前向计算……点下'停止'……协作式取消……两个时刻" | 同 S27 | 〔旁白起〕两枚天平砝码落下：左"你的等待↓"、右"计算量＝"（不变）；〔"点下停止"〕A 区停止按钮按下→红色取消信号沿通道逆流回调度器（途经 Tokenizer 变形为 AbortBackendMsg 标签【教学简化，角标注明】）；〔"两个时刻"〕时间轴上两个刻度点"你点击"与"资源回收完成"依次亮起并保持距离 | "流式≠更快生成"；"取消：从浏览器传回调度器"；"点击 ≠ 回收完成"；角标：原文 Part Ⅱ 第 5 章 §5.2 | 砝码"咚"；信号逆流"咻——"；刻度"叮"×2 |
| S29 | 10:20–10:55 | "回头看这一路……直到 EOS 它才真正停笔" | 全流水线全景（快速重放） | 〔旁白与站点结论同步〕镜头从右向左快速横移，每到一站闪现结论大字（前台·发号 / Tokenizer·切与编号 / 调度·算谁 / Prefill·并行+建缓存+首 token / KV Cache·按层追加 / Decode·单步循环 / 采样·抽签 / Detokenizer·拼完整 / SSE·逐段到达），每字停留 0.8s；〔"真正停笔"〕EOS 紫方块落到光标处，光标熄灭，全场景灯光转暗一档 | 九个结论词（金色大字依次）；角标：对应章节汇总 | 横移"唰"；每站"叮"；熄灭"噗" |
| S30 | 10:55–11:10 | "每一个字，都是算出来的——也是抽出来的。下一集：Token 是怎么切出来的？" | 中央收束 | 〔旁白起〕"算出来的"金字+齿轮图标、"抽出来的"金字+签筒图标并列；〔"下一集"〕画面缩为圆窗，圆窗内"凋｜零"块裂开成两半的预告动画（呼应参考系列钩子，D 区角标"预告画面为演示"）；黑场出片尾 | "算出来的，也是抽出来的"；"下一集：Token 到底怎么切？" | 双金"咚咚"；裂开"咔"；黑场淡出 | 

## 3.5 C. 视觉素材清单

### 3.5.1 架构图类（静态底图，可复用）

| 编号 | 名称 | 内容 | 出处/依据 | 使用镜头 |
|---|---|---|---|---|
| V01 | 四类组件/五节点流水线总图 | API Server(入口)→Tokenizer→Scheduler+Engine→Detokenizer→API Server(出口)→SSE 回环；控制/数据双路径双色；组件分组括线 | 教程 Part Ⅱ 第 2 章图 2-1（重绘，非截图） | S04–S05、S29 |
| V02 | 请求状态条 | WAITING→PREFILL→DECODE→FINISHED 简化状态条 | 教程 Part Ⅱ 第 1 章图 1.4 | S09/S16/S20 角标 |
| V03 | KV Cache 分层架 | 按层分组的 token 格位（K 蓝/V 绿成对） | Part Ⅰ 第 4 章图 4-1/4-2 意象重绘 | S18–S19、S21 |
| V04 | 显存蛋糕模板 | 权重/KV/其他三扇区，可滑动比例 | Part Ⅰ 第 4 章 §4 数字 | S19 |
| V05 | 采样控件包 | 温度旋钮（低/高弧）、top-k/top-p 开关、累计竖线 | Part Ⅱ 第 3 章 §3.2 | S24 |
| V06 | 词表抽屉+嵌入表 | 行号列+向量色块行 | Part Ⅱ 第 3 章图 3-1 | S07/S11 |
| V07 | 因果掩码 5×5 矩阵 | 右上三角幕布 −∞ | Part Ⅱ 第 3 章图 3-5 | S13 |

### 3.5.2 动态图形类（需逐帧制作）

| 编号 | 名称 | 规格 | 使用镜头 |
|---|---|---|---|
| A01 | token 块（三态） | 文字态/编号态/向量态，统一圆角方块语言，金色下划线 | 全片 |
| A02 | Q/K/V 三色卡 | 金/蓝/绿，尺寸 72px，带投影 | S12/S21 |
| A03 | 弧线箭头系统 | 三种粗细=三种权重；读取/写回双向箭头 | S12/S21/S27 |
| A04 | 层平面堆叠体 | 半透明层片×N 堆叠动画 | S13/S14 |
| A05 | 取号牌/请求卡片 | 带 uid 角标 | S04/S09 |
| A06 | SSE 波纹通道 | 数据包沿线推进 | S05/S27 |
| A07 | 重复计数器 | Σ=1+2+3+… 累加 | S17 |
| A08 | EOS 紫方块 | 无字+"⏹"印记 | S23/S29 |
| A09 | 大字标题系统 | 金属字落下/结论字闪现两套 | S03/S15/S29 |
| A10 | 吉祥物（可选） | 参考作者风格；如启用需自绘原创形象避免侵权【合规提醒】 | 开场/收尾点缀 |

### 3.5.3 公式与字幕

| 编号 | 内容 | 呈现方式 | 使用镜头 |
|---|---|---|---|
| F01 | KV_bytes＝B×T×L×2×n_kv×d_h×s | 公式条逐项点亮+代入数值 | S19 |
| F02 | o_{t+1}＝softmax(q_{t+1}K₁:t+₁ᵀ/√d_h)V₁:t+₁ | 小字角标（不朗读） | S21 |
| F03 | p_i＝exp(logit_i/T)/Σexp(logit_j/T) | **温度先作用于 logits、再 Softmax**（顺序与旁白/S24 动画一致；采样专集再全展开，本集只示意 T 的作用） | S24 |
| F04 | Softmax 定义（无温度） | 柱形变形动画 | S24 |
| S-01 | 底部解说字幕 | 白色中文，与配音逐句对齐，样式同参考视频 | 全片 |
| S-02 | 角标系统 | 六类徽章：教学示意 / 原文出处（Part X 第 Y 章）/ **原文计算**（公式推算值，非实测）/ **参考视频实测** / **本项目实测**（仅真实运行后使用）/ 系列预告 | 全片 |

### 3.5.4 其他素材

- **背景**：深蓝点阵网格+透视地板（矢量，4K 源）；A 区浏览器窗口皮肤（模拟 UI，避免使用真实产品截图）。
- **音乐**：低调氛围电子底 1 条（约 11 分钟循环）；章节变奏 4 段。
- **音效包**：嗒/叮/咚/唰/咻/嚓/咔哒/buzz/警鸣 共 9 类，复用参考视频的音效语义映射。
- **配音**：中文男声或女声（TTS 候选：需支持 SSML 分句控制；人工配音则按解说稿逐节录制）。
- **需要特别注明的素材红线**：不使用教程仓库 images/ 目录下的任何截图直接入片（CC BY-NC-SA 4.0 允许署名共享改编，但为了画面统一全部重绘）；不使用参考视频的画面元素（只借鉴方法论与色彩语义）。

## 3.6 D. 动画实现建议

### 3.6.1 工具选型总判断（v2：单一主管线，避免多工具并行过重）

**主管线：Remotion（React）为主时间轴与合成工具**，其余工具只做受限补充。第一支样片不让四套工具共同承担基础画面。

| 环节 | 工具 | 职责 |
|---|---|---|
| 主时间轴 / 合成 / 渲染 | **Remotion** | 全片镜头序列、音频/字幕/音效同步点、转场、批量渲染出 MP4；参数化改数字零成本、可版本管理 |
| 图形组件 | SVG / Canvas（React 组件） | token 块、缓存格、请求卡、弧线箭头、柱状图、显存蛋糕、旋钮、SSE 波纹、大字系统 |
| 数学特写（**按需可选**） | Manim | 仅当确实需要精确几何/LaTeX 的片段：F01 公式条、S11 探针取行、S13 5×5 掩码。输出透明底视频，由 Remotion 合成 |
| 手工精修（**后期可选**） | After Effects / DaVinci Resolve | 样片验证通过后，若个别镜头需要更高级的质感与光效，再单镜头引入；不进入基础管线 |

选择理由：本片动画主体是**数据驱动**的（柱形、蛋糕比例、累计线、缓存格位），Remotion 中"旋钮→柱形"由同一份数据函数驱动，天然满足音画同步与参数一致性；Remotion 的时间轴代码化也便于逐句对齐解说时间戳。AE 的优势（手绘质感、摄影机抖动）在样片阶段不是必需项。

**⚠️ Remotion 授权核查（进入 G1 前完成）**：Remotion 是 **source-available 软件，不是 OSI 开源软件**。当前官方条款：个人或 **3 人及以下**的团队符合条件时可使用免费许可（Remotion Free License）；更大团队/公司通常需要购买 Company License。制作启动前须登记：①Remotion 版本；②制作团队人数与组织性质；③是否符合 Free License 条件；④如不符合——改用 Company License 或更换主管线（备选：纯开源组合 = Manim + MoviePy/ffmpeg 合成，或 Motion Canvas）。⑤许可证确认日期。官方条款：remotion.dev/docs/terms、remotion.dev/docs/license/pricing。

**视觉空间定义（v2.1）：第一版为 2.5D 连续动画舞台**——SVG/Canvas 图形 + CSS 透视 + 摄影机缩放平移（Remotion transform）实现空间纵深与"连续舞台"感；**不默认引入真实 3D 场景**。SVG 的镜头缩放是 2D 变换，不等同于带视差的 3D 摄影机；如某集确需真实 3D（如 GPU 芯片解剖），单独评估引入 Three.js/React Three Fiber 或后期合成，并在该集策划书中注明。

### 3.6.2 分场景实现建议（v2：按 Remotion 主管线改写）

| 场景 | 实现要点 |
|---|---|
| S01–S03 开场 | Remotion：token 弹出用 spring()（overshoot）；打字机碎裂用 mask 序列帧；大标题金属字用 SVG 贴图+落定震屏 1 帧 |
| S04–S05 全景地图 | Remotion：五站牌为 React 组件单场景；"文字球"沿贝塞尔路径 motion path；SSE 波纹用 SVG stroke-dashoffset |
| S06–S08 Tokenizer | Remotion：栅栏线 SVG 滑入；翻面用 CSS 3D transform；"教学示意"角标为统一组件，入点=镜头推近帧 |
| S09 调度 | Remotion：探照灯用径向渐变遮罩；容量锁绿灯两态组件 |
| S11 查表 | **Manim 片段（可选）**：`Table`/`Matrix` mobject + `FocusOn`、探针 `line` 逐根出现；导出透明底视频由 Remotion 合成。若用 Remotion 直做：SVG 行列网格 + staggered 探针动画亦可满足 |
| S12–S13 注意力/掩码 | Remotion+SVG：弧线用 quadratic path（stroke-width/不透明度映射权重）；掩码幕布用半透明矩形+`−∞` 文本；FFN 盒为组件 |
| S14–S15 logits | Remotion：分数→概率柱补间写成数据函数（输入数组即出动画），本集与采样专集复用同一组件 |
| S17–S19 KV Cache | Remotion：重复计数器（useCurrentFrame 驱动）、分层架落卡（按层数组循环渲染）、显存蛋糕（SVG 扇区按比例数据驱动）；公式 F01 若需 LaTeX 用 Manim 渲染单条，否则用 KaTeX-in-Remotion |
| S20–S22 Decode | Remotion：单 token 循环轨道（frame 取模）；双 lane 对比闪回复用已渲染序列（interpolate 时间重映射）；S22 逐层读写循环按 3.4 表新描述实现 |
| S24 采样 | Remotion：**温度 t∈(0,2] 直接驱动 logits 柱高（先除 T），Softmax 函数再生成概率柱，top-p 累计线由同一概率数组计算**——数据函数实现顺序＝旁白顺序＝原文顺序，保证"旋钮动、柱形必变"且次序正确 |
| S25–S28 流式与取消 | Remotion：波纹包 delta 文字交换；取消逆流 path 动画；"两个时刻"刻度轴组件 |
| S29–S30 收尾 | Remotion：横移重放=已渲染镜头的低帧率代理按 0.6s/站串接；圆窗遮罩转场 |

### 3.6.3 制作流水线与同步规范

1. **先锁解说音频**：TTS/配音定稿 → 导出逐句时间戳 → 分镜表 S 序列按时间戳回填，保证"旁白触发动画"逐句对齐（参考视频的核心做法）。
2. **组件先行**：先做 A01–A05 五个母组件并评审，再铺量。
3. **一致性检查脚本化**：颜色语义（金=动作、蓝=数据、紫=EOS、红=风险、青=top-p）写进样式表；渲染前跑脚本检查误用色。
4. **输出**：1080p25 master；字幕烧录版+外挂 SRT 双版本；音效与 BGM 分轨存档，便于平台再剪辑。
5. **采样数据函数的顺序约束**：柱形动画的实现顺序必须＝旁白顺序＝原文顺序——先温度缩放 logits → 再 Softmax 归一化 → 再 Top-k/Top-p 截断与重归一化 → 再抽样。代码评审时按此顺序逐行核对。

## 3.7 E. 质量审核（逐项检查结论）

| 审核项 | 结论 | 证据/说明 |
|---|---|---|
| ① 是否覆盖计划全部知识点 | ✅ 覆盖任务要求的 11 项教学内容 | Tokenizer(S06–S08)、Token ID 与 Embedding(S07/S11)、Transformer 结构(S12–S14)、QKV/Attention(S12)、Prefill(S10/S16)、KV Cache 生成与保存(S18/S19)、Decode(S20–S21)、自回归(S20 轨道+循环表述)、Logits/采样/Temperature/Top-K/Top-P(S14–S15/S24)、Detokenizer(S25–S26)、Streaming(S27)；用户提交 Prompt(S01/S04)、请求调度(S09)作为任务"注意"条款亦覆盖 |
| ② 是否与 Datawhale 原文一致 | ✅ 逐条对齐 | 见 3.2 对照表 K1–K17，每条标注原文章节；数字（14.06MB/0.141MB/36 层/8 KV 头/128 维/停止条件三分法/温度与截断定义/半个字节问题/流式不减少计算量）均出自原文对应章节 |
| ③ 是否有知识性错误 | 未发现；三处需成片时执行的保护措施 | a) S11 词表行数 151,936 来自参考视频对 Qwen3-0.6B 的实测，非教程数字，角标已注明来源【网络补充】；b) S12"弧线粗细=匹配度"为教学示意，角标注明真实为逐维点积；c) S06 token 切分为示意切分，角标注明真实切分取决于分词器 |
| ④ 是否有不严谨类比 | 两处，均已加角标或改为直述 | a) uid="取号牌"【教学推演】（原文只说"分配内部唯一标识"）；b) Decode"串行往前爬"为口语化描述，画面仍以真实机制（读缓存+追加）为准，不用拟人替代机制 |
| ⑤ 动画是否真正解释原理（而非装饰） | ✅ | KV Cache 镜头（S17–S19）完成了"为什么缓存（重复计算演示）→缓存什么（K/V 落架、Q 弹回）→多大（公式代入）→怎么用（S21 读缓存+追加双向箭头）"完整因果链；采样镜头旋钮与柱形由同一数据函数驱动，杜绝"画面与参数脱节" |
| ⑥ 是否存在画面与旁白不匹配风险 | 低；已内置同步机制 | 解说稿每节标注触发词，分镜表每格含〔旁白触发〕列；制作流水线第 1 步先锁音频时间戳 |
| ⑦ 是否存在为视觉效果牺牲准确性 | 已规避三处常见陷阱 | a) 不把"逐字出现"画成打字机特效——S02 显式打叉、S27 给出真相；b) 不把 Prefill 与 Decode 画成"两个不同模型/两台不同机器"——S22 明确同一引擎两个阶段（PD 分离是后续集内容，本集不引入）；c) 不把 KV Cache 画成"一块连续大内存"——分层架按层分组，第 19 集再引出分页 |
| ⑧ Prefill/Decode/模型计算/调度/流式是否严格区分 | ✅ | 五者各有独立镜头与颜色（调度蓝门/引擎金门/流式波纹），S04/S05 明确"分工≠进程"，S09 明确"队列≠立即执行"，S27–S28 明确流式的收益边界与取消的两时刻 |
| ⑨ 术语首次定义是否完整 | ✅ | Token/Token ID/Embedding/注意力/因果掩码/FFN/LM Head/logits/Prefill/Decode/KV Cache/EOS/max_tokens/TTFT/温度/Top-k/Top-p/SSE——首次出现均有大字或角标定义 |
| ⑩ 时长与信息密度 | 10:50±（压缩方案 9:30） | 解说 2530 字≈10:00 朗读+动画停顿；密度对齐参考视频（536 秒讲 4 站，本集 650 秒讲 4 站+缓存专讲，增量合理） |

---

## 附录 A 教程文件清单（机器可核验，共 37 个中文 markdown，下载自 raw.githubusercontent.com/datawhalechina/zero-to-sglang/main，访问日期 2026-11-15）

| # | 路径（course-material/ch/ 或根目录） | 部分 | 类型 | 映射视频集 |
|---|---|---|---|---|
| 1 | README_zh.md | 总览 | 说明 | 全系列定位 |
| 2 | part0/Part0-编码伦理与开源精神.md | Part 0 | 概念 | 番外/第 22 集 |
| 3 | part0/Part0-部署你的第一个SGLang服务.md | Part 0 | 动手 | 番外 |
| 4 | part1/第1章_LLM入门.md | Part Ⅰ | 概念 | 第 1/2 集 |
| 5 | part1/第2章_推理入门.md | Part Ⅰ | 概念 | 第 1/3/4 集 |
| 6 | part1/第3章_GPU入门.md | Part Ⅰ | 概念 | 第 5 集 |
| 7 | part1/第4章_推理的核心数据结构入门.md | Part Ⅰ | 概念 | 第 1/8 集 |
| 8 | part1/第5章_Benchmark入门.md | Part Ⅰ | 概念 | 第 20 集 |
| 9 | part2/第1章_mini-sglang：推理引擎长什么样.md | Part Ⅱ | 概念 | 第 6 集 |
| 10 | part2/第2章_一个请求的旅程.md | Part Ⅱ | 概念 | 第 1/7 集 |
| 11 | part2/第2章_一个请求的旅程_代码.md | Part Ⅱ | 代码走读 | 第 7 集 |
| 12 | part2/第3章_前向与生成.md | Part Ⅱ | 概念 | 第 1/2 集 |
| 13 | part2/第3章_前向与生成_代码.md | Part Ⅱ | 代码走读 | 第 2/7 集 |
| 14 | part2/第4章_KVCache优化.md | Part Ⅱ | 概念 | 第 1/8 集 |
| 15 | part2/第4章_KVCache优化_代码.md | Part Ⅱ | 代码走读 | 第 8 集 |
| 16 | part2/第5章_HTTP服务与并发.md | Part Ⅱ | 概念 | 第 1/9 集 |
| 17 | part2/第5章_HTTP服务与并发_代码.md | Part Ⅱ | 代码走读 | 第 9 集 |
| 18 | part2/第6章_ContinuousBatching与调度.md | Part Ⅱ | 概念 | 第 10 集 |
| 19 | part2/第6章_ContinuousBatching与调度_代码.md | Part Ⅱ | 代码走读 | 第 10 集 |
| 20 | part2/第7章_PagedKVCache与显存管理.md | Part Ⅱ | 概念 | 第 11 集 |
| 21 | part2/第8章_RadixAttention与前缀缓存.md | Part Ⅱ | 概念 | 第 12 集 |
| 22 | part2/第8章_RadixAttention与前缀缓存_代码.md | Part Ⅱ | 代码走读 | 第 12 集 |
| 23 | part2/第9章_多进程与张量并行.md | Part Ⅱ | 概念 | 第 13 集 |
| 24 | part2/第9章_多进程与张量并行_代码.md | Part Ⅱ | 代码走读 | 第 13 集 |
| 25 | part2/第10章_投机解码.md | Part Ⅱ | 概念 | 第 14 集 |
| 26 | part2/第10章_投机解码_代码.md | Part Ⅱ | 代码走读 | 第 14 集 |
| 27 | part3/第1章_注意力后端.md | Part Ⅲ | 概念 | 第 15 集 |
| 28 | part3/第2章_量化与低精度推理.md | Part Ⅲ | 概念 | 第 16 集 |
| 29 | part3/第3章_分层缓存.md | Part Ⅲ | 概念 | 第 17 集 |
| 30 | part3/第4章_横向扩展.md | Part Ⅲ | 概念 | 第 18 集 |
| 31 | part3/第5章_PrefillDecode分离.md | Part Ⅲ | 概念 | 第 19 集 |
| 32 | part4/第1章_用Cookbook部署SGLang.md | Part Ⅳ | 动手 | 第 20 集 |
| 33 | part4/第2章_Profiling与Trace分析.md | Part Ⅳ | 概念+实例 | 第 21 集 |
| 34 | part4/第3章_SGLangPR工作流.md | Part Ⅳ | 概念 | 第 22 集 |
| 35 | community/ch/index.md | Community | 说明 | 第 22 集延伸 |
| 36 | community/ch/PR_requirement.md | Community | 规范 | 第 22 集延伸 |
| 37 | community/ch/notes/llm-engine-viz/原理可视化.md | Community | 笔记 | 相关集延伸阅读 |

> 未下载：course-material/ch/WRITING_TEMPLATE.md（写作规范，非教学内容）、eng/ 英文版（内容与中文版对应）、images/（图片资源，成片全部重绘）。

## 附录 B 关键外部来源（网络补充类，与原文区分）

| 来源 | 用途 | 日期/版本 |
|---|---|---|
| B 站 BV1jdpS6tEMQ（阿栗学长）实测画面：Qwen3-0.6B 词表 151,936、概率表、"凋零"切分钩子 | 样片词表数字、系列钩子衔接 | 实测于视频发布版本（B 站，观看日期 2026-11-15） |
| B 站 BV1UV4e61ESj / BV1MjYD6gEJe（同作者） | 风格体系确认（实测数字、论文引用卡、免责角标做法） | 2026-11-15 观看 |
| SGLang 官方文档（docs.sglang.io：install / cookbook / attention_backend / quantization / pd_disaggregation / hicache / benchmark） | 教程引用的官方链接汇总 | 以教程各章参考资料栏为准 |
| 论文：Attention Is All You Need（1706.03762）；FlashAttention（2205.14135）；PagedAttention（2309.06180）；Online Softmax（1805.02867）；Speculative Decoding（2211.17192）；SGLang（2312.07104）；AWQ（2306.00978）；GPTQ（2210.17323） | 延伸阅读清单 | 教程原文已列，此处汇总 |

**引用台账（正式制作前必须锁定 commit，避免上游更新导致引用漂移）：**

| 引用对象 | 版本/Commit | 用途 | 访问/锁定日期 |
|---|---|---|---|
| datawhalechina/zero-to-sglang | main 分支（访问日期 2026-11-15；**制作启动时执行 `git rev-parse HEAD` 锁定并记录于此**） | 全部【原文】引用的来源基线 | 2026-11-15（待锁 commit） |
| sgl-project/mini-sglang | `9a91cfafe754aa85daee49998176275667eb58f2`（2026-09-16，教程 Part Ⅱ 声明的参考基线） | 函数名/代码结构引用（forward_batch、store_kv、match_prefix 等） | 教程原文声明 |
| sgl-project/sglang | `1291f3175d3b7614821884e9653985fcd3c3dae8`（Part Ⅱ 第 10 章代码篇）；`1d019ce1c85e11ff2f425344e66f446b106f6074`（Part Ⅲ 第 1 章）；`75878f27c88c2ffd296425d94226d0dc9bd42518`（Part Ⅲ 第 2 章）；`0c2e211930c10117beb4c52591bffe237525afdf`（Part Ⅲ 第 5 章）；`bfcf15a129fd4e26ef8d4af140f30cbcf02a6696`（Part Ⅳ 第 2 章 / PR #13587 合并提交） | Part Ⅲ/Ⅳ 相关集的源码引用锚点（完整 40 位 SHA，仓库：github.com/sgl-project/sglang） | 教程原文声明 |
| B 站 BV1jdpS6tEMQ / BV1UV4e61ESj / BV1MjYD6gEJe | 发布版本（下载日期 2026-11-15） | 风格研究 + 【参考视频实测】数字 | 2026-11-15 |

台账维护规则：每个结论/数字/画面入片前，登记"原始论据 → URL/commit → 版本日期 → 访问日期 → 是否教学推演"五元组；凡无法登记到台账的表述，一律降级为【教学推演】并加角标。

## 附录 C 本方案未做的事项（明确边界）

- ❌ 未设计交互式实验室或 HTML 网站；
- ❌ 未将课程压缩为若干概念模块（仍按教程 33 个教学文件的完整映射推进，见 2.26 分集表）；
- ❌ 未批量生成第 2 集及以后的分镜/解说（仅完成第 1 集样片全套材料）；
- ❌ 未编造任何源码函数、执行步骤或性能数据——涉及 mini-sglang 的函数名（`forward_batch`、`store_kv`、`match_prefix` 等）均转录自教程代码走读篇原文；样片正片不直接展示源码，涉及处一律以"原文出处"角标承载。

---

## 附录 D 修订记录（v2：吸收外部审校意见）

| 审校条目 | 核实结论 | 修订位置 |
|---|---|---|
| P0-1 温度/Softmax 顺序写反 | **成立**。教程 Part Ⅱ 第 3 章 §3.2.2："温度 T 会先除到 logits 上，再做 Softmax"；参考视频同样"旋钮装在换算之前" | 解说稿第 8 节改为三步顺序；S24 分镜改为左右两栏（分数域/概率域）、旋钮先作用于 logits；F03 加顺序说明；3.6.3 新增第 5 条顺序约束 |
| P0-2 KV Cache 数字/单位 | **成立**。147,456 B/token；14.20 对应 101 token 而非 200；42.19 对应 300 token；原文按 1024 进位但记作 MB | S19 分镜重写（101→14.20、300→42.19 两阶段计数）；K10 补每 token 字节数；新增"原文计算"徽章类型与 1024 进位角标、B=1 与理论稠密占用限定条 |
| P0-3 "每算出一个 Token 就发回一段"与 Detokenizer 矛盾 | **成立**。与 S25 半字节问题冲突，且教程明确"一个事件不一定对应一个 token" | 解说稿第 9 节改为"生成→Detokenizer 拼接→拼出安全文本立即推送，一次推送可含 1..n 个 Token"；S27 增加接力环与"教学简化包体"角标 |
| P0-4 TTFT 等同于 Prefill 耗时 | **成立**。教程 Part Ⅰ 第 5 章："包含排队与 Prefill" | 解说稿第 5 节改为分段表述；S16 改分段标尺（排队/分词/Prefill/首 Token 采样/首段返回）+"TTFT ≠ Prefill 耗时"字幕 |
| P0-5 Decode 搬运动画过于绝对 | **部分成立**。"每步读全部权重+历史 KV"有原文支撑（Part Ⅰ 第 2 章 §4.3 逐项列出），但"整段缓存作为单块包裹整体搬运"的画面会误导（实际为逐层 kernel 读取） | 解说稿第 7 节保留原文口径但改逐层表述；S22 改逐层循环动画（读权重→读历史 K/V→写新 K/V）+小 Batch 限定角标 |
| P1-6 信息密度与后续集重复 | 部分采纳。第 1 集深度是任务书的明确要求，不削减；处理方式＝明确"第 1 集＝全景导入集"定位 + 在 2.26 增加与第 2/3/4/5/6/8/9 集的防重复分工说明 | 2.26 分工说明段落；分集表第 1 行加定位标注 |
| P1-7 风格原创边界 | 成立 | 1.5 新增第 8 条"原创性边界"；只借方法论与动画语法，资产全原创 |
| P1-8 章节计数不一致 | 成立。正确计数：33 教学（2+5+18+5+3）+3 社区 +1 README＝37；Part Ⅱ 为 18 个文件（第 7 章无独立代码篇） | 2.26 校验段修正；附录 A 重写为逐文件机器可核验表格 |
| P1-9 "原文实测"徽章误用 | 成立。Qwen3-4B 数字是配置+公式推算 | 新增【原文计算】/【参考视频实测】徽章类型（§0 表、1.5 第 3 条、S-02）；S11/S19 徽章改标；解说稿第 6 节徽章措辞修正 |
| P1-10 引用需可追溯 | 成立 | 附录 B 新增引用台账（zero-to-sglang 待锁 commit、mini-sglang 9a91cfa、SGLang 五个 commit）+ 五元组登记规则 |
| P1-11 CC BY-NC-SA 商业边界 | 成立。"非商业"是假设前提，不是已确认授权 | §0 第 5 条改为条件式表述，明确商业用途需先取得权利方确认，重绘不豁免改编许可 |
| 工程建议：多工具并行过重 | 采纳 | 3.6.1/3.6.2 重写为 Remotion 主管线 + SVG/Canvas 组件 + Manim 按需片段 + AE 仅后期精修 |
| 生产关卡 | 采纳 | 见下方"样片生产关卡"，替换原"下一步建议" |

### v2.1 修订行（吸收第二轮复审）

| 复审条目 | 核实结论 | 修订位置 |
|---|---|---|
| P0-1 K12 采样顺序仍写反 | 成立（讲稿已改，映射表漏改） | §3.2 K12 改为六步全链顺序；已全文检索 Softmax/温度相关表述确认一致 |
| P0-2 缺 RoPE/Residual/Norm | 成立。教程 Part Ⅱ 第 3 章 §2.2/§2.4 明确覆盖 | 讲稿第 5 节插入 RoPE 一句 + Block 四步小流水线（norm→Attn→⊕→norm→FFN→⊕，约 20 秒）；S12 增加刻度盘与 RoPE 步；S13 改为掩码+残差+Pre-Norm 四步侧视图 |
| P0-3 S12 Query 可见范围 | 成立。因果注意力可见范围含当前位置自身 | S12 与讲稿均改为"当前位置以及之前（含自身）"，自身 K 描金边 |
| P0-4 "半个字节"术语 | 成立。应为"部分 UTF-8 字节序列" | S25 重写为"缓＝3 字节被两 Token 分段承载"示意；讲稿第 9 节同步；K14 更新；全文档已无"半个字节/左半字节"表述（第 2.3.2 单元保留教程原文措辞"部分字节"） |
| P0-5 TTFT 两口径 | 成立 | 讲稿第 5 节改为双时刻；S16 改双刻度标尺；K8 更新 |
| P0-6 三套时长对不上 | 成立（约 100 秒未闭合） | 不再以字数锁码：3.3 头部新增"时长口径声明"，全部时间码降级为 T0 估算基线；G2 后按真实音频回填并要求闭合；3.4 头部同步标注 |
| P1-1 四站/五站不统一 | 成立 | 讲稿第 2 节、S04/S05、V01 统一为"四类逻辑组件、五个路径节点" |
| P1-2 max_tokens 命名 | 成立 | K13 与 S23 增加字段名口径注释 |
| P1-3 S17 O(n²) 范围 | 成立 | S17 角标限定"仅指历史 K/V 重复投影累计；注意力交互累计仍约 O(n²)" |
| P1-4 S01 示例回答 | 成立 | S01 增加"示例回答·教学示意"角标 |
| P1-5 知识点级覆盖 | 成立 | 新增附录 E（Part Ⅰ 第 1 章 + Part Ⅱ 第 3 章逐知识点映射）；并规定每集开拍前必须产出该集的知识点级覆盖表作为 G1 前置门 |
| P1-6 参考视频日期 | 成立（原写 2025-10 为猜测） | K4 改为 B 站 pubdate 时间戳 1791766543（≈2026-10）+ 观看日期；删除猜测性发布表述 |
| P1-7 Remotion 授权 | 成立（source-available，非 OSI） | 3.6.1 新增授权核查清单（团队人数/Free License 条件/备选开源管线）与"2.5D 连续舞台"空间定义 |
| P1-8 commit 省略号 | 成立 | 附录 B 五个 SGLang commit 补齐 40 位完整 SHA |

---

## 附录 E 知识点级覆盖表（v2.1 新增；其余章节在各自集开拍前照此格式产出）

> 覆盖深度定义：**完整**=该集主讲并配专镜；**概览**=一句话+一个镜头立结论；**预告**=角标提示后续集展开；**待排**=当前 2.26 未安排专集，排期评审时决定归属。验收门槛：原文章节的每个小节至少落入一行，且不允许出现"没有任何集讲"的悬空知识点。

### E.1 Part Ⅰ 第 1 章《LLM 入门》逐知识点

| 原文小节 | 知识点 | 计划集 | 深度 | 讲稿/镜头锚点 |
|---|---|---|---|---|
| §2 | LLM 定义（预测下一个 token 的概率模型） | 第 1 集 | 概览 | 第 3 节"模型不认识文字，只认识数字"承接；完整讲解在第 3 集 |
| §2 | 发展脉络 2017→2022（Transformer→BERT/GPT→GPT-3→开源浪潮） | 第 3 集 | 完整 | 第 3 集时间轴动画；第 1 集不涉及 |
| §2 | 能力边界（幻觉/知识截止/上下文有限） | 第 3 集 | 完整 | 第 3 集；第 1 集回顾不提 |
| §3.1–3.4 | 应用与推理成本/延迟动机（KV 显存、TTFT/TPOT 约束） | 第 3/4 集 | 完整 | 第 3 集成本结构、第 4 集两阶段；第 1 集 S19 的显存蛋糕为其前置概览 |
| §4 | Transformer 起源与 Encoder-Decoder 结构 | 第 1 集 | 概览 | S12/S13"一层内部"；完整结构对比在结构专集（待排：并入第 2 集扩展或新增集） |
| §4.1 | 正余弦位置编码（公式+多尺度） | 第 2 集（分词/表示专集） | 完整 | 第 2 集公式推导；第 1 集 S12 只讲 RoPE 直觉（概览，注明原始论文用正余弦） |
| §4.2 | MHA：Q/K/V、缩放点积、多头切分/拼接/输出投影、原论文参数（512/8/64） | 第 1 集+结构专集 | 第 1 集概览（Q/K/V+单头直觉，S12） | 多头拆分/拼接完整动画归结构专集（待排） |
| §4.3 | LayerNorm（公式、γ/β）与残差（F(x)=H(x)−x）；Post-Norm vs Pre-Norm | 第 1 集 | 概览（S13 直觉化："稳定尺度/直通车"） | 公式级讲解归结构专集（待排） |
| §4.4 | FFN 结构与 ReLU；激活函数三特质 | 第 1 集 | 概览（S13"自己消化"） | FFN 内部（扩展比/门控/SwiGLU）归结构专集（待排） |
| §4.5 | SOTA 演进对照表（MoE/GQA-MLA/RMSNorm/RoPE/量化精度等 10 条线） | 各专集 | 预告 | 第 1 集 S29 角标一闪；逐条在各专集展开 |
| §5.1–5.6 | Prompt/上下文工程/Skill/RAG/Agent/Harness | 第 3 集（为什么需要推理优化） | 概览 | 第 3 集应用图；第 1 集不涉及 |
| §5.7 | Token（≈1.6–1.7 汉字/DeepSeek）；Embedding；参数；自回归生成；上下文窗口（1M≈1.5–2 本红楼梦） | 第 1 集+第 2 集 | Token/Embedding/自回归=第 1 集完整（第 3 节、S20–S21 循环）；参数/上下文窗口=第 8/11 集概览 | — |
| §6 | 总结与测试题 | 不入正片 | — | 课程网页承载；视频不放测试题 |

### E.2 Part Ⅱ 第 3 章《前向与生成》（概念篇+代码篇）逐知识点

| 原文小节 | 知识点 | 计划集 | 深度 | 讲稿/镜头锚点 |
|---|---|---|---|---|
| 概念 §2.1 | tokenize 三步与 BPE/WordPiece/SentencePiece 列表；"只做映射" | 第 2 集 | 完整 | 第 1 集 S06–S08 为概览（只立结论） |
| 概念 §2.1 | embedding 查找表 [vocab_size, d_model]；[B,T,d_model] | 第 1 集 | 完整 | S11 |
| 概念 §2.2 | RoPE（对 Q/K 旋转，点积携带位置） | 第 1 集 | 概览+直觉 | S12 刻度盘（v2.1 新增）；公式级在第 2 集 |
| 概念 §2.3 | Q/K/V 语义；多头拆分；Causal Mask 负无穷 | 第 1 集 | Q/K/V 与 Mask=完整（S12/S13）；多头拆分=概览（角标） | — |
| 概念 §2.4 | Block：Attention 跨 token + MLP 单 token；Pre-Norm；残差；N 层+final norm | 第 1 集 | 概览（S13 四步小流水线，v2.1 新增） | 结构公式级归结构专集（待排） |
| 概念 §2.5 | LM Head；logits 未归一化；只取最后位置 | 第 1 集 | 完整 | S14/S15 |
| 概念 §2.6 | 结构变体表（MoE/MQA-GQA/RMSNorm/FlashAttention） | 各专集 | 预告 | S13/S29 角标一闪 |
| 概念 §3.1–3.2 | 采样动机；Greedy；温度公式与五档；Top-k/Top-p 定义与对照表 | 第 1 集 | 概览（S24 三步立结论） | 完整展开=采样专集 |
| 概念 §3.3 | 自回归循环；停止条件三条 | 第 1 集 | 完整 | S20 轨道+S23 |
| 概念 §4 | Prefill 三件事/Decode 六步/两阶段区别/TTFT-TPOT | 第 1 集+第 4 集 | 第 1 集完整定性（S10/S16/S20–S22）；定量（Roofline/TPOT）=第 4 集 | — |
| 概念 §5 | 概念总结表 | — | — | 作为字幕/角标素材库，不单独成段 |
| 代码篇 §2 | 源码定位表（models/qwen3.py 等） | 第 2/7 集 | 完整 | 第 7 集请求链路代码走读 |
| 代码篇 §3.1–3.2 | Qwen3Model.forward；打包布局 [N,C] vs [B,T,C]；RMSNormFused 残差合并 | 第 7/2 集 | 完整（第 2 集表示布局、第 7 集代码） | 第 1 集不涉及（S13 直觉化已注明教学简化） |
| 代码篇 §3.3 | RopeAttn：qkv_proj 合并投影、Q/K 归一化+RoPE、V 不旋转；GQA repeat_interleave 教学展开 | 第 2 集 | 完整 | 第 1 集 S12 只取"V 不旋转/RoPE 只转 QK"两点做直觉 |
| 代码篇 §3.4 | GatedMLP：gate_up_proj→silu(gate)*up→down_proj | 结构专集（待排） | 完整 | 第 1 集 S13 不展开 |
| 代码篇 §4.1–4.3 | LM Head get_last_indices；generate 教学循环；Sampler 两条路径（argmax/温度+top_k+top_p） | 第 1 集+第 7 集 | 第 1 集"只取最后位置"=完整（S14）；采样两路径=概览（S24） | 代码画面归第 7 集 |
| 代码篇 §5 | 形状自查表与因果性三检查 | 第 2 集 | 完整 | — |

> 排期评审决议项（记录在此，不悬空）：①"Transformer 结构专集"是否新增或并入第 2 集（含 MHA 多头/LayerNorm 公式/FFN 内部/正余弦编码）——**建议新增为第 2.5 集**，理由：第 2 集分词与本结构内容体量均足一集；②第 3 集标题从"训练 vs 推理"扩展为"LLM 是什么、为什么会算"，以承接 Part Ⅰ 第 1 章 §2–3/§5.1–5.6 的应用与边界内容。

---

## 附录 F G1 交付记录（v3.1）

| 项 | 记录 |
|---|---|
| 交付物 | `sglang-video/g1/frame1-opening.svg / frame2-stage.svg / frame3-kvcache.svg`（源文件）＋ `sglang-video/g1/rendered/G1-frame1-opening.png / G1-frame2-stage.png / G1-frame3-kvcache.png`（1920×1080 实渲染）＋ `sglang-video/README.md`（字体/工具/分辨率/文件映射） |
| 评审结论 | 第一轮评审：整体设计方向保留；按评审意见完成三项小修——①Frame 1 问题 Token 链补齐 KV/Cache 并加"教学切分示意 · 非真实 Qwen3 Tokenizer 输出"角标；②Frame 3 分层架改为每层 8 个 Token 位置（16 个 K/V 数据格）；③三层统一缓存长度（各 4 个已缓存 Token，第 5 位跨层高亮"待追加"，标注"同一逻辑位置，逐层分别写入"）；④讲稿 LM Head 前补 Final Norm（第 5 节/S14/K7） |
| 验收状态 | **G1 已完成**（三帧均经浏览器 1920×1080 实渲染核验：字体/下标/中文字符/溢出/安全边距通过） |
| 剩余问题 | 无阻塞项。已知说明：Token 切分为教学示意（画面已标注）；Frame 3"写入层 N"标签与位置 6 格左缘间距较紧，G2 精修时顺带调整 |
| G2 状态 | **已提交，未验收**。交付物（`sglang-video/g2/`，详见 `sglang-video/README.md`）：成片 `out/G2-animatic-30s.mp4`（1080p25 H.264+AAC，30.04s，约 8.3MB）＋工程源码 `src/index.html`（确定性 setT 时间轴：连续舞台、摄影机推拉横移、逐步构建、数据沿路径运动）＋配音 `audio/seg*.wav`（macOS say·Tingting 占位音）＋字幕 `out/G2-subtitles.srt`＋混音母带 `out/vo30.wav`＋验收检查帧 `out/check_t5/t17/t27.png`。节拍合约与再生成命令见 README §4–5。范围：0–8s 回车+Token；8–20s API Server/Tokenizer/Scheduler；20–30s Transformer（Embedding/QKV/Attention，Q 可见范围含自身）。管线说明：本次样条采用"确定性 HTML+浏览器渲染+ffmpeg"纯开源管线（规避 Remotion 授权待核查项）；Remotion 主管线决策仍按 §3.6.1 执行 |

## 样片生产关卡（v2，不直接渲染 11 分钟）

| 关卡 | 交付物 | 验收标准 |
|---|---|---|
| G1 风格帧 | 3 张 16:9 关键帧（开场 / 连续舞台 / KV Cache 特写） | 美术体系成立；颜色语义、token 形状、徽章样式定稿；与参考视频无资产混淆 |
| G2 30 秒动态样条 | 开场→全景→Tokenizer 三镜头连续片段 | 镜头推拉流畅；对象逐步构建；字幕/音效/解说逐句同步 |
| G3 60–90 秒技术证明片段 | KV Cache 连续动画：重复计算 → K/V 落架 → Decode 读缓存+追加（S17–S19/S21） | 因果链完整；数字与单位正确（101→14.20、300→42.19）；徽章类型正确；无"整体搬运"误导画面 |
| G4 整集 Animatic | 简化图形+临时配音+占位音效跑通全时长 | 节奏成立；无与后续集重复的展开段；旁白每个技术断言逐条核对（对照 3.2 表） |
| G5 正片 | 锁稿→配音→精动画→混音→字幕→导出 | 逐项验收：断言正确、画面解释断言、单位一致、原文/补充区分、徽章无"实测"误标、 TTFT/采样顺序/Token-SSE 关系三处红线复查 |

---

**方案结束（v2）。** 本文档可作为第一轮导演/编剧稿基础；下一步是按 G1–G5 关卡推进样片，而不是扩展交互功能或批量生成后续章节。
