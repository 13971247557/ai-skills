---
name: video-edit-pipeline
description: "成片合成器：把 AI 视频镜头素材（智谱清影/豆包等生成片段）+ 配音 + BGM + 字幕自动剪辑合成一条流畅、无水印、可发布的成片。用 ffmpeg 完成拼接、转场、混音、字幕烧录与导出；用 edge-tts 免费生成中文配音（多音色，无需 Key）；内置 BGM 音量平衡（人声下压 6~10dB）、流畅度质检清单与无水印策略。触发场景：用户说'合成成片/剪辑出片/配字幕/配音/配BGM/去水印/把素材剪成一条视频/出一条能发的片子'时使用；配合 ai-short-video（内容与分镜）、ark-video-pipeline（素材生成）组成完整出片链。"
---

# 成片合成器（video-edit-pipeline）

## 最高原则：出片质量第一位（用户硬要求，压过一切其他考量）

**质量 > 成本 > 速度。** 任何环节在质量与省钱/省时冲突时，**无条件选择质量**：

1. **素材不合格就重做**：镜头有水印、崩坏（手部/多指/乱码）、画质差、人物不一致 → 重生成或换渠道，不为省钱保留低质素材；
2. **配音不自然就重配**：朗读腔、节奏平、情绪不对 → 调参数/换音色/改文本重配，不将就；
3. **剪辑不流畅就重剪**：转场生硬、卡点不对、节奏拖沓 → 重剪，不硬拼；
4. **逐关卡质控**：每一阶段过质检清单（`references/quality.md`）才进入下一阶段，不合格不交付；
5. **交付前必须检查完整（用户硬要求）**：每次交付成片前强制过「交付前完整性检查」三项——①字幕无重叠（脚本自动 clamp + 日志"残余重叠 0" + **抽帧目检：抽 3~5 帧看字幕无重叠、无截断、不越底部安全区**）；②成片完整可播（ffprobe + 播放验证）；③画面与字幕对齐（安全区/不遮挡/无截断）。缺一不可，见 `references/quality.md`；
6. **成本策略从属**：免费渠道优先，但质量不达标时允许用付费/高成本方案（去水印、高质量TTS、更稳的生成渠道），以"质量达标下的最低成本"为准，而非绝对免费；
7. **如实上报**：无法达到用户质量要求的项，明确说明差距与可选项，不假装成功。

## 概述

输入：镜头视频素材目录 + 剧本/配音文本 + BGM（可选）→ 输出：一条带字幕、配音、BGM、转场流畅的成片。

**本机已验证工具链（v2 升级）**：ffmpeg 4.4（剪辑/xfade 转场/字幕/混音/loudnorm）、edge-tts（免费中文配音，多音色+音量+逐行标记）、系统含 Noto Serif CJK 中文字体。xfade 转场链与导出一键参数已按调研落地（字幕安全区 20%H、fps 30、High@4.1、closed GOP、BT.709、loudnorm 双遍 -14 LUFS）。

> **转场/插帧（2026-10 调研，详见 `references/quality.md`）**：xfade 4.4 **原生支持 `fadewhite`/`fadeblack`/`fadegrays`/`dissolve`/`hblur`**（白闪直接用 fadewhite，不必 fade 近似；但它是扁平白场，非体积雾）；叠化时长按叙事分档（短切 0.3–0.5s / 情绪 0.8–1.2s / 回忆 1.5–2s）；`--interpolate` 用 minterpolate 运动补偿插帧到目标 fps（参数 mci:aobmc:bidir:vsbmc=1:scd=fdiff，已实测可播）。

> **大片级升级要点（2026-10 二轮调研，详见 `references/editing-mastery.md`）**：
> - **xfade 工程纪律**：两输入须同分辨率/同帧率/同时基/CFR，必须重编码（不能 -c copy）；`offset = 镜头1长度 − duration`，多镜链式 offset 递推、镜头多时用 ffprobe 自动算；音频同步叠化用 `acrossfade`；
> - **节奏锚点**：动作段 2–4s/镜、剧情段 10s+/镜、短视频 <0.5s/镜；释放点放慢动作（`setpts`+`minterpolate`）或静帧（`loop`）；
> - **声音三层**：人声 `acompressor`+`highpass` 前置 → BGM 走 `sidechaincompress` 自动避让（threshold 0.02/ratio 4/attack 200/release 1000，人声 asplit=2 一路驱动一路混回）→ 音效按点 `adelay` 对齐 → 母带 `loudnorm` 两遍法 linear=true（YouTube −14 LUFS、TP≤−1.5）；混响 `aecho`/`afir`；
> - **色彩**：Log 素材先 Log→Rec.709 LUT 再风格 LUT；统一 `lut3d`（必加 `-pix_fmt yuv420p`）保跨镜头一致；ffmpeg 无 AI 自动色彩匹配（C 档）；
> - **花字包装分工**：逐词高亮对白走 ASS `\k`（`ass` 滤镜）；静态文字走 drawtext；**复杂花字/片头片尾/卡点文字卡片→走 HyperFrames 通道**（输出 mp4 再 overlay 回本链）。

## 合成工程铁律（蒸馏自开源 video-use / ffmpeg-skill，合成前必过）

1. **字幕最后加**：字幕滤镜永远放在 filter 链末尾、所有 overlay 之后——否则被其他覆盖层盖住（静默失败）；
2. **纯拼接走无损 concat**：无转场/无字幕/无滤镜的纯拼接用分段 `-c copy` 无损 concat，不进单次 filtergraph（否则整段重编码、质量损失+耗时）；需要转场/字幕/滤镜时才进重编码链（xfade 必须重编码）；
3. **段边界 30ms 音频淡入淡出**：每个拼接边界加 `afade=t=in:st=0:d=0.03` 级短淡变，防爆音/咔哒声；
4. **固定四步工作流**：探测（ffprobe 核时长/帧率/分辨率/音频流）→ 编辑 → 检查（look 联络表抽帧目视，不只信命令成功）→ 验证（ffprobe+播放+字幕安全区），缺一步不交付。

## 工作流

| 步骤 | 做什么 | 工具/命令 |
|---|---|---|
| 0 | 素材检查：镜头视频 01.mp4、02.mp4…（同分辨率/帧率），ffprobe 核对 | `ffprobe` |
| 1 | 配音：**先按 `references/audio.md`「场景→配音程度判定」选音色/语速/音量**（悬疑=云健降速轻声、情感=晓晓轻诉、口播=云扬加速；全角色音色 ID/参数/来源见 `references/voice-library.md`），再生成配音轨（多角色逐行换音色，自动防字幕重叠）。**pitch 硬纪律**：女声角色禁用 pitch 负数（实测会把女声压进男声区，F0<140Hz 一律不合格）；老奶奶角色 pitch +2/+3；交付前每段配音过 `scripts/voice_check.py`（女声 F0≥150Hz）；完整规则见 `references/audio.md`「F0 性别质检」 | `scripts/tts_voice.py` / `scripts/voice_check.py` |
| 2 | 字幕：配音自动输出配套 SRT（句级时间戳，防重叠 clamp，含停顿偏移） | `scripts/tts_voice.py --srt-out` |
| 3 | 拼接+转场：默认 xfade 转场链（fade 0.5s；`--cut hard` 回退硬切；高潮可 hlslice 卡点切片）。**转场/卡点/节奏手法选型先查 references/editing-mastery.md（决策树+卡点SOP+节奏模板）；工具分工/AI工具链/大师手法先查 references/editing-tools.md** | `scripts/build_video.py --cut xfade` |
| 4 | 混音：人声 + BGM（BGM 音量压至人声下 6~10dB）+ 淡入淡出 + loudnorm 双遍 -14 LUFS。**响度标准/平台规范/loudnorm 实操见 references/audio.md「响度标准与平台规范」** | 同上 |
| 5 | 字幕烧录：SRT → drawtext 时间窗口烧进底部安全区（规避 ffmpeg 4.4 libass bug；自动防重叠）。**字幕安全区/样式/SRT 格式/平台差异先查 references/subtitle-safety.md / subtitle-style.md / srt-timing.md / platform-rendering.md** | 同上 |
| 6 | 水印策略：见「无水印策略」 | — |
| 7 | 质检：见 `references/quality.md` 清单（含交付前完整性检查三项） | `ffprobe` + 抽帧 |
| 8 | 导出：竖屏 1080×1920 或横屏 1920×1080，H.264 High@4.1 + AAC 192k/48k，BT.709，音频截断对齐视频时长 | 同上 |

## 剪辑质量自检清单（交付前强制，详见 references/editing-mastery.md 第十章）

四项硬关卡，交付前必过：

1. **转场合理性**：转场类型与情绪/节奏匹配；软转场（叠化等）≤总切点 30%。
2. **节奏一致性**：切点密度匹配节奏设计模板；无 3s 以上无刺激段；爆点位于 75%–90%。
3. **卡点对齐误差**：切点与拍点偏差 ≤0.25s 精确 / ≤0.4s 可接受 / >0.4s 已补救（微调切点→微调变速→放弃卡点）。
4. **连续性**：同主体相邻镜机位差 ≥30°；无越轴错误；动作匹配切点在峰值前（动作启动后 30%–60%）；视线方向一致。

## 命令速查

```bash
# 配音（单段，同时输出句级字幕，全局音量）
python3 scripts/tts_voice.py --text "雨夜的街头，霓虹灯在闪烁。" --voice zh-CN-XiaoxiaoNeural --out voice.mp3 --srt-out sub.srt --volume -20%

# 配音（多段：逐行合成，行内标记 [暂停0.5秒] [rate:-10%] [volume:-30%] [voice:Yunjian]）
python3 scripts/tts_voice.py --script lines.txt --voice zh-CN-XiaoxiaoNeural --out voice_full.mp3 --srt-out sub.srt

# 脚本示例（多角色 + 重音降速 + 轻声 + 悬念停顿）
# 旁白(云健)：雨夜的街头，霓虹灯在闪烁。
# [暂停0.8秒][voice:Yunxi]男主(云希)：你怎么才来。
# [voice:Xiaoxiao][volume:-30%][rate:-10%]女主(晓晓)轻声：我以为你不会来了。
# 合成成片（clips 目录内 01.mp4、02.mp4... 按文件名序；默认 xfade 转场链）
python3 scripts/build_video.py --clips ./clips --voice voice_full.mp3 --bgm bgm.mp3 --subtitle sub.srt --out final.mp4 --ratio 9:16 --cut xfade --xfade fade --xfade-dur 0.5

# 白闪转场（穿白场，原生 fadewhite）/ 60fps 运动补偿插帧（--interpolate）
python3 scripts/build_video.py --clips ./clips --out flash.mp4 --ratio 9:16 --cut xfade --xfade fadewhite --xfade-dur 0.4
python3 scripts/build_video.py --clips ./clips --out smooth60.mp4 --ratio 9:16 --fps 60 --interpolate

# 仅拼接不加音轨/字幕
python3 scripts/build_video.py --clips ./clips --out silent_edit.mp4 --ratio 9:16 --cut hard

# 高级手法（2026-10-13 落地，默认全关）：爆点速度斜坡 + 环境床 + L-cut 声桥 + 颗粒 + 青橙调色 + 浅景深
python3 scripts/build_video.py --clips ./clips --voice v.mp3 --bgm b.mp3 --ambience amb.wav:0.015 \
  --speed-ramp --jl-cut 0.3 --grain normal --grade teal --dof soft --out pro.mp4
```

## 无水印策略（用户硬要求：成片不带水印）

- **素材端优先**：清影默认带「清影·AI生成」角标，网页端开启「去水印」参数需额外积分或 VIP；豆包网页版生成同样带平台标识。
- **免费无水印素材渠道**：Luma（5 条/天）、Google Veo 3.1（AI Studio，每天 5~10 次，需 Google 访问）、可灵付费档；生成前先选渠道。
- **合成端兜底**：角标位于固定角落时，用 ffmpeg crop 裁掉安全区（损失少量边缘画面）；不要在成片上叠加任何自有 logo 水印。
- **质检含「无水印」项**：逐镜抽帧检查四角无平台角标，有则裁切或重生成。

## Resources

| 文件 | 内容 | 何时读 |
|---|---|---|
| `scripts/tts_voice.py` | edge-tts 配音生成（单段/多段脚本模式，含句级 SRT 输出） | 需要配音时 |
| `scripts/build_video.py` | 拼接/混音/字幕烧录(drawtext)/导出 主合成脚本 | 每次合成时 |
| `references/audio.md` | 配音音色表、**场景→配音程度判定表**（音色/语速/音量/停顿/情绪浓度三档）、BGM 选择与音量平衡规则、**响度标准与平台规范（-14 LUFS/EBU R128/loudnorm 单双遍/交付规格）** | 配音/配乐/响度归一前必读 |
| `references/voice-library.md` | 中文全角色音色库（全渠道音色 ID/角色/参数/来源，含 F0 纪律） | 配音选音色时必读 |
| `references/quality.md` | 流畅度质检清单、参数规范（帧率/码率/转场节奏/字幕安全区）、无水印检查 | 质检前必读 |
| `references/editing-mastery.md` | 剪辑手法精通手册（转场体系/**xfade offset 链式算法/acrossfade/custom:expr/光转场** /节奏情绪/**ASL 锚点/静帧 loop/时间重映射** /卡点SOP/视觉连续性/声音设计/**sidechain 避让/loudnorm 两遍法/aecho 混响** /风格化/**lut3d/Log 顺序/跨镜头一致** /剪辑决策树/节奏设计模板/剪辑质量自检清单/**花字包装 ASS\k↔HyperFrames 分工**） | 规划转场与卡点、剪辑手法选型时必读 |
| `references/editing-tools.md` | 剪辑工具能力对照（剪映/Pr/达芬奇）、AI 工具链分工（OpusClip/Descript/Runway/Pika）、Murch/Schoonmaker/Frame.io 大师要点、图形匹配/smash cut/软切/乐句卡点、与脚本结合点 | 决定「用哪款软件做、哪步进本地管线」时读 |
| `references/subtitle-safety.md` | 字幕安全区数值表（竖屏底 23-25%H/上 228px/左右 60px）+ if-then 决策 | 定字幕位置时必读 |
| `references/subtitle-style.md` | 字幕样式规范（无衬线黑体/字号 7-8%/黑描边/每行≤16字/防遮挡清单）+ 脚本待改进点 | 定字体字号描边时必读 |
| `references/srt-timing.md` | SRT 格式硬规则（逗号时间码/UTF-8 无 BOM）+ 时长阅读速度 + Ofcom 同步要求 | 写/校对 SRT 时读 |
| `references/platform-rendering.md` | 软硬字幕对照 + 平台渲染差异（抖音必烧录/B站仅.ass/Apple HLS WebVTT） | 判断烧录还是挂软字幕时读 |
| `references/transitions-toolkit.md` | **高级转场工具包（2026-10-13 蒸馏）**：自动 xfade 转场链生成器 auto_xfade.py（零依赖，本机实测出片）+ RIFE 插帧两路线（rife-ncnn-vulkan CPU 可跑/-g -1、Practical-RIFE 需 GPU）+ xfade 43 类型速查表（本机 ffmpeg -h 实测）+ 决策分支 | 要自动转场链/插帧补帧/选 xfade 类型时必读 |
| `references/smart-subtitle.md` | **智能字幕工具包（2026-10-13 蒸馏）**：whisper/faster-whisper 转写决策树与模型选型表 + stable-ts 断句优化与 ASS 卡拉OK 花字直出（{\kXX}）+ ffmpeg ass 烧录命令 + 双语翻译两路线 + 一条照跑全链示例（本机实测） | 从音频出字幕/断句优化/逐词花字/翻译对齐时必读 |

## 引用导航

| 任务/场景 | 查阅文件 | 何时读 |
|---|---|---|
| 需要脚本执行（拼接/转场/混音/字幕烧录/导出） | `scripts/build_video.py` | 每次合成 |
| 需要脚本执行（edge-tts配音+句级SRT） | `scripts/tts_voice.py` | 步骤1/2 |
| 女声F0性别质检 | `scripts/voice_check.py` | 交付前 |
| 火山豆包配音（真角色音/方言） | `scripts/volc_synth.py` | 配音补充 |
| 场景→配音程度判定/BGM平衡/F0质检/响度标准 | `references/audio.md` | 配音配乐前 |
| 全角色音色ID库与参数 | `references/voice-library.md` | 选音色 |
| 交付前完整性检查三项/参数规范/无水印 | `references/quality.md` | 质检前 |
| 转场/卡点/节奏决策树/节奏模板 | `references/editing-mastery.md` | 合成前规划 |
| 工具能力对照/AI工具链分工/大师手法 | `references/editing-tools.md` | 选工具/分工时 |
| 字幕安全区数值与 if-then 决策 | `references/subtitle-safety.md` | 定字幕位置 |
| 字幕字体/字号/描边/字数规范 | `references/subtitle-style.md` | 定字幕样式 |
| SRT 格式与时长规则 | `references/srt-timing.md` | 写/校对 SRT |
| 软硬字幕与平台渲染差异 | `references/platform-rendering.md` | 判断烧录/软字幕 |
| 自动转场链/插帧补帧/43 种 xfade 选型 | `references/transitions-toolkit.md` | 规划转场与补帧时 |
| 音频转字幕/断句优化/ASS 卡拉OK花字/翻译对齐 | `references/smart-subtitle.md` | 字幕生成与美化时 |

**UPSTREAM:** 由 `ai-video-director` 在「配音字幕、合成成片、交付前完整性检查」阶段调用。
