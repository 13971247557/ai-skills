---
name: hyperframes-html-video
description: "HTML 出片通道：用 HyperFrames 把 HTML/CSS/GSAP 动画确定性渲染成无水印 MP4，专治 ffmpeg 做不好的花字字幕、动态片头片尾、卡点文字卡片、口播提词卡、数据滚动与模板批量渲染。写一个带 data-* 时间轴的 index.html + 一条可 seek 的 GSAP 时间线，headless Chromium 逐帧截图后由 ffmpeg image2pipe 合帧，同输入必出同帧、零预算本地渲染、成片不带水印。与 video-edit-pipeline（拼镜头素材）互补——它管实拍/AI 镜头剪辑，本技能管『代码画出来的动画层/独立动画段』。触发场景：用户说‘做个花字字幕/动态片头片尾/文字卡点卡片/口播提词卡/HTML 出片/同款文字模板批量渲染/把这段 SRT 做成动效字幕/数据数字滚动动画’时使用。"
---

# HTML 出片通道（hyperframes-html-video）

## Overview

把「一段代码写的动画」变成「一条能发的 MP4」：组合就是一个 `index.html`，DOM 上用 `data-*` 属性声明时间轴，动画用一条**可 seek（可暂停定位）**的 GSAP 时间线实现；渲染器在 headless Chromium 里**不播放、只按帧号逐帧 seek 再截图**，截图直接经 stdin 喂给本地 ffmpeg 合帧。

它补的是现有成片链的空白：`video-edit-pipeline` 用 ffmpeg 拼实拍/AI 镜头素材、烧死板字幕；而**花字字幕、动态片头片尾、文字卡点卡片、口播提词、数据滚动、同款模板批量出片**这类「设计驱动的动效段」，ffmpeg 做不出质感，正是本通道的主场。

**本机已验证**：Node v22 + ffmpeg 4.4 + Chromium，`npm run render` 出 1920×1080/30fps/h264/无水印 MP4。免费、无水印、同输入同输出。

## 何时使用

- 需要**动态/花字字幕**（不是 ffmpeg 死板烧一行 SRT），或把 SRT 做成逐词高亮、放大、弹跳的动效字幕层；
- 需要**片头/片尾**：标题淡入、logo 入场、片尾字幕条；
- 需要**文字卡点卡片**：跟着 BGM 拍点跳出的数字/关键词/报价卡；
- 需要**口播提词卡 / 数据数字滚动 / 图表动画**；
- 需要**同款模板批量渲染**：一个 HTML 模板 × N 组文案 JSON = N 支定制片。

> 不适用：拼接实拍/AI 镜头、配音混音、BGM 压条、死板字幕烧录——这些仍走 `video-edit-pipeline`。本通道产出「纯动画段/字幕层 mp4」，再交回 ffmpeg 合成链。

## 大片包装应用场景（与 ffmpeg 字幕通道的分工）

> **结论：纳入大片包装流程。** HyperFrames 正是 ffmpeg/ASS 弱项的补位——复杂动效文字层走本通道，简单字幕走 ffmpeg。

| 包装需求 | 推荐通道 | 理由 |
|---|---|---|
| 逐词高亮对白字幕（TikTok 黄词效果） | ffmpeg ASS（`\k` 标签） | libass 原生 karaoke，Whisper 词时间戳驱动，轻量成熟 |
| 普通烧录字幕/对白条 | ffmpeg ASS/subtitles | 成熟、省一次渲染 |
| 静态 logo、水印、下三分之一静态条 | ffmpeg drawtext | 单滤镜即可 |
| **花字（逐词弹大/旋转/描边阴影跳）** | **本通道（HyperFrames）** | ASS `\k` 只能变色不能缩放弹动，GSAP 质感 |
| **动态片头/片尾（多元素编排动画）** | **本通道** | ffmpeg 做不出设计驱动动画 |
| **卡点文字卡片（随拍点跳出数字/报价）** | **本通道** | GSAP 时间轴对齐拍点，可批量 |
| **口播提词卡 / 数字滚动 / 图表动画** | **本通道** | DOM+CSS 动画天然擅长 |
| **同款模板批量出片** | **本通道** | 模板×JSON 批量渲染 |

**决策边界**：
- 需要"设计感"的动效文字（花字弹动、片头编排、卡点卡片）→ 走本通道；
- 只需要"能看清"的对白字幕 → 走 ffmpeg ASS；
- 逐词变色（黄词）→ ffmpeg ASS `\k`；逐词弹大（放大弹跳）→ 本通道。

产物形态：本通道输出纯动画段/字幕层 mp4，再交回 `video-edit-pipeline/build_video.py` 当一路 clip，或 `overlay` 叠到口播实拍上。

来源：调研文档 `research/04-editing-mastery.md` §5（花字包装可行性评估）

## 工作流

| 步骤 | 做什么 | 命令 |
|---|---|---|
| 1 建工程 | 本地脚手架（**不写全局配置**） | `npx --yes hyperframes@0.8.114 init <工程名>` |
| 2 写模板 | 从 `templates/` 拷一个改，或按 `references/html-rendering.md` 契约手写 | 编辑 `index.html` |
| 3 动效字幕(可选) | SRT → HTML 字幕动画片段 | `python3 scripts/srt_to_html_captions.py sub.srt -o captions.html` |
| 4 巡检 | 静态 + 浏览器跑一遍（改完就跑） | `npm run check` |
| 5 出片 | 渲染 mp4 到 `renders/` | `npm run render`（批量：`npx --yes hyperframes@0.8.114 render --batch rows.json --output "renders/{name}.mp4"`） |
| 6 验收 | 校验时长/分辨率/可播 | `ffprobe -v error -show_entries stream=codec_name,width,height,r_frame_rate renders/*.mp4` |
| 7 接入合成链 | 把产出 mp4 当一路 clip，或 overlay 到口播实拍 | 交回 `video-edit-pipeline/scripts/build_video.py` |

## 命令速查

```bash
npx --yes hyperframes@0.8.114 init my-title      # 脚手架（含 index.html/hyperframes.json/package.json）
npx --yes hyperframes@0.8.114 doctor --json     # 检查 Node/ffmpeg/Chromium 是否就绪
npx --yes hyperframes@0.8.114 check             # 静态+浏览器巡检（lint 内含）
npx --yes hyperframes@0.8.114 render            # 出 mp4（= npm run render）
npx --yes hyperframes@0.8.114 render --batch rows.json --output "renders/{name}.mp4"  # 模板批量
ffprobe renders/*.mp4                            # 验收
```

> 本机默认 `captureMode=screenshot`（软件 GL）：避免重型 blur/backdrop-filter/WebGL，轻量文字动画无压力；静态元素尽量少动，引擎会自动复用未变的帧（实测省约一半渲染时间）。

## 引用导航

| 任务 / 场景 | 查阅文件 | 何时读 |
|---|---|---|
| 写/改 index.html 的 data-* 时间轴、变量、批量 | `references/html-rendering.md` | 每次动手写 HTML 前必读 |
| 片头/标题动画起手模板 | `templates/title-fade/`（index.html + hyperframes.json + package.json） | 要做片头片尾时直接拷改 |
| 花字字幕/底部字幕条起手模板 | `templates/caption-rail/` | 要做动效字幕层时直接拷改 |
| 把现成 SRT 转成字幕动画 HTML | `scripts/srt_to_html_captions.py` | 已有精修好的 SRT（srt_refine 产物）时用 |
| 与 ffmpeg 合成链衔接、overlay 回成片 | `video-edit-pipeline/SKILL.md` + `video-edit-pipeline/scripts/build_video.py` | 出片后要混入镜头/配音/BGM 时 |

**UPSTREAM:** 由 ai-short-video 阶段6、video-edit-pipeline 在需要 HTML 出片素材（字幕动画/片头片尾/文字卡片）时调用，产出 mp4 后进入 ffmpeg 合成链。
