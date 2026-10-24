---
name: scene-cut
description: 自动场景切分与智能粗剪：读入一段视频，用内容变化检测自动找出硬切镜头边界，输出每个场景的时间区间 JSON，可选直接按场景切成独立片段。基于开源 PySceneDetect（pip 可装，本地运行，不调云端、不耗生成额度）。已本机实测：对带 2.0s 硬切的合成素材精确检出两段并切成 scene_001/scene_002。典型用途：长素材先自动粗切成镜头清单、多机位/混剪前先切分、粗剪交给后续 EDL 或人工精修。与 video-use（逐字稿 EDL 精剪）互补——本技能负责"找到切在哪"，video-use 负责"按语义决定留哪段"。触发：自动切分镜头、场景检测、粗剪、找出视频里的镜头切换点、scene detection、split into scenes。
license: BSD-3-Clause
metadata:
  source: https://github.com/Breakthrough/PySceneDetect
  installed_from: Breakthrough/PySceneDetect (BSD-3, 2026-10 蒸馏为薄封装)
---

# scene-cut

> 蒸馏自开源库 Breakthrough/PySceneDetect（BSD-3）。原库是成熟的命令行场景切分工具；本技能把它封装成一个可直接跑的 `scripts/scene_cut.py`，输出结构化场景区间 JSON，方便接进本组后续剪辑链。方法论与参数调优见 `references/scene-cut-method.md`。

## 何时用

- 拿到一段长素材，想先自动知道"里面有几个镜头、切在哪一秒"。
- 混剪/多机位前，先把成片粗切成独立镜头片段。
- 粗切之后再交给人工或 `video-use` 的语义精剪。
- 本组 `video-use` 做的是"按逐字稿决定留哪段"；本技能做的是"纯视觉上找到硬切边界"，两者上下游互补。

## 已实测可跑的命令（2026-10-13 本机验证）

前置：`pip install scenedetect`（本机已装 0.7.2）、`ffmpeg` 在 PATH。

```bash
# 只检测，输出场景区间 JSON
python3 scripts/scene_cut.py input.mp4 --out-json scenes.json

# 阈值更灵敏（硬切明显时降到 20-25）
python3 scripts/scene_cut.py input.mp4 --threshold 25

# 同时按场景切成独立片段到 clips/
python3 scripts/scene_cut.py input.mp4 --split --out-dir clips
```

实测记录：对"2s testsrc + 2s smptebars 硬切"的合成素材，`--threshold 25` 精确检出 场景1(0.00–2.00s) 与 场景2(2.00–4.00s)，并 `--split` 产出 scene_001.mp4 / scene_002.mp4。

参数：`--threshold`（内容变化阈值，默认27，越小越灵敏）、`--min-scene`（最短场景秒，默认0.6，过滤闪切）、`--out-json`（JSON 路径）、`--split`（切片段开关）、`--out-dir`（片段目录）。

## 边界

- 本技能只认**硬切**（画面突变）；渐变转场、叠化、淡入淡出靠 `--threshold` 调低勉强捕捉但不保证。
- 不做语义判断（不区分"这一段该不该留"），那是 `video-use` 的事。
- 静音口癖切除不是本技能范围，见 `references/scene-cut-method.md` 的 ffmpeg silencedetect 备选思路。

## 目录

```
scene-cut/
├── SKILL.md
├── references/scene-cut-method.md   # 阈值调优/与粗剪链衔接/静音切分备选
└── scripts/scene_cut.py            # 场景检测+切片段（已实测）
```
