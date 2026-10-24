---
name: video-localization
description: 视频本地化管线：把一段视频转写→翻译→（可选配音）→烧字幕/最终合成，一条龙出可发布成片。四步编排器：①转写出源语 SRT；②翻译并按句界重切分成目标语 SRT 或双语 SRT；③按需用 TTS 配时间对齐的配音；④一次 ffmpeg 编码里烧字幕+混音（保留原声作低声床）。中文走火山 ASR/TTS，其他语言走 Whisper/edge-tts。本技能已孵化出可直接跑的脚本 scripts/render.py（烧字幕+混音，已本机实测产出 burned.mp4）；转写与配音步骤的引擎路由见 references/pipeline.md，需对应 API Key。触发：视频本地化、转成中文字幕配音、翻译字幕、烧硬字幕、横转竖后发抖音视频号、dub this video、burn subtitles。
license: MIT
metadata:
  source: https://github.com/jianshuo/claude-skills
  installed_from: jianshuo/claude-skills (MIT, 王建硕, 2026-10 蒸馏四步管线)
---

# video-localization

> 蒸馏自开源仓库 jianshuo/claude-skills（MIT，王建硕）。原仓库把本地化拆成四个独立子技能；本技能把它们的**管线编排逻辑**和**最终合成脚本**孵化成一个可执行技能。转写/翻译/配音的引擎路由方法论见 `references/pipeline.md`。

## 何时用

- 用户要"把这段外文视频做成中文字幕 + 中文配音"，或"帮我把这段视频本地化"。
- 只要单步：只转写、只翻译字幕、只烧硬字幕——直接跳到对应步骤，边界更干净。
- 本组已有 `lip-sync`（对口型）、`video-use`（逐字稿剪辑）；本技能补的是**跨语言本地化这条链**（源语 SRT → 目标语 SRT → 配音 → 一次合成）。

## 已实测可跑的命令（2026-10-11 本机验证）

最终合成这一步已跑通（本机 ffmpeg 4.4 自带 libass）：

```bash
# 只烧字幕：原音轨直通
python3 scripts/render.py --video in.mp4 --srt in.zh-CN.srt --out burned.mp4

# 只换/混配音：保留画面，替换或混音音轨
python3 scripts/render.py --video in.mp4 --dub dub.mp3 --out dubbed.mp4

# 完整本地化版：一次编码里既烧字幕又混配音，默认把原声压到 0.18 倍作低声床
python3 scripts/render.py --video in.mp4 --srt in.zh-CN.srt --dub dub.mp3 --out localized.mp4

# 不要原声床：加 --no-original-audio
# 字幕样式可调：--font "PingFang SC" --fontsize 12 --margin-v 40 --style outline
```

**暂不可跑（如实标注）**：第①步转写、第③步配音需要对应引擎的 API Key（火山 ASR/TTS 或 OpenAI Whisper）。本机无这些 Key，未实跑；拿到 Key 后按 `references/pipeline.md` 的路由表执行，产出的 SRT/配音音轨直接喂给上面已跑通的 `render.py`。

## 四步编排（详见 references/pipeline.md）

| 步 | 做什么 | 输入 → 输出 | 引擎路由 |
|---|---|---|---|
| 1 转写 | 出源语时间戳 SRT | 音视频 + 源语种 → 源语 SRT | 中文默认火山（豆包）ASR；其他语言 Whisper API 词级时间戳 |
| 2 翻译 | 翻译并按句界重切 | 源语 SRT + 目标语 → 目标语 SRT / 双语 SRT | 文本翻译，zh-CN 与 en 为一等目标 |
| 3 配音（可选） | 时间对齐 TTS 配音 | 视频 + 目标语 SRT + 音色 → `*_<lang>_dub.mp4` | 中文火山 TTS；其他语言 edge-tts；默认单说话人 |
| 4 合成（已跑通） | 烧字幕 + 混音，一次编码 | 视频 + SRT +（可选配音）→ 发布 MP4 | `scripts/render.py`，libass 烧字幕 |

## 硬规则

- **默认单说话人**：独白/口播/录课类一律单音色；只有用户明说"两个人/访谈/对话/分角色"才做多说话人。
- **一次编码出成片**：字幕 + 配音混音在同一条 ffmpeg 命令里完成，不要多级串接重编码。
- **转写停在源语 SRT**，翻译停在目标语 SRT，配音停在配音轨——每步边界清晰，最后交给 `render.py` 合成。
- 中文字幕/配音优先火山引擎，不要默认 Whisper（中文准确率更低）。

## 目录

```
video-localization/
├── SKILL.md
├── references/pipeline.md   # 四步方法论 + 引擎路由表 + SRT 重切规则
└── scripts/render.py         # 最终合成（烧字幕+混音，已实测）
```
