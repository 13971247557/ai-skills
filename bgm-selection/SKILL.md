---
name: bgm-selection
description: "BGM 选配：为 AI 短视频/短剧成片挑选并配置背景音乐。按成片场景/情绪匹配 BGM 风格（悬疑/燃向/甜宠/口播/情感/史诗），从免费无版权曲库选曲，用脚本分析 BPM/时长/响度验证适配性，按卡点设计落鼓点，并输出与配音平衡的混音参数（BGM 压人声下 6~10dB、淡入淡出）。触发场景：用户说“选BGM、配背景音乐、配乐、卡点音乐、BGM风格怎么定、给片子配首歌”；与 video-edit-pipeline（合成）、bgm-remaster（编曲重塑）配合。"
---

# BGM 选配（bgm-selection）

## 定位

成片链中的"配乐"环节：**把合适的背景音乐选出来、验证适配、给出混音参数**。只管"选与配"；"编曲重塑"（歌词好但伴奏难听）走 `bgm-remaster`；"混音合成"在 `video-edit-pipeline` 的 `build_video.py --bgm`。

## 流程

| 步骤 | 做什么 | 产出 |
|---|---|---|
| 1 | **读成片场景**：从分镜/剧本/成片抽帧判断情绪基调（悬疑/燃/甜/emo/口播/史诗…） | 场景情绪标签 |
| 2 | **匹配风格**：按 `references/selection.md` 情绪→风格/乐器/速度映射表定 BGM 要求 | 风格要求（类型+速度档+BPM区间） |
| 3 | **选曲**：从免费无版权曲库找候选（见 selection.md 曲库清单）；用户提供本地 BGM 文件则直接用 | 候选曲 1~3 首 |
| 4 | **分析验证**：`scripts/analyze_bgm.py` 测 BPM/时长/响度，对照成片时长与卡点需求（3秒/15秒节奏节点） | 适配结论（选哪首/是否可卡点） |
| 5 | **卡点设计**：按节奏节点定鼓点落点与转场处音乐（卡点档位见 mixing.md） | 卡点方案 |
| 6 | **给混音参数**：音量（有人声时 BGM 压至人声下 6~10dB，脚本默认 volume 0.40≈-8dB）、淡入0.5s/淡出1.5s | 混音参数（直接喂 build_video.py） |
| 7 | **合成验证**：`build_video.py --bgm` 合成后听感检查（BGM 不盖人声、情绪匹配） | 成片 |

## 命令速查

```bash
# 分析候选 BGM（BPM/时长/响度/可卡点判断）
python3 scripts/analyze_bgm.py --audio bgm_candidate.mp3

# 合成成片带 BGM（BGM 自动压至人声下约 8dB + 淡入淡出）
python3 ../video-edit-pipeline/scripts/build_video.py --clips ./clips \
  --voice voice.mp3 --bgm bgm.mp3 --subtitle sub.srt --out final.mp4 --ratio 9:16
```

## 硬规则

- **BGM 永不盖过人声**：有配音时 BGM 压至人声下 6~10dB（本地默认 volume 0.40≈-8dB）；业界参考上限为人-BGM 差≥15dB（详见 mixing.md 两级标准）；只有 BGM 无配音时正常音量；
- **情绪匹配优先**：先定情绪再选曲，不因"好听"选与场景冲突的曲子（燃向成片配抒情曲=翻车）；
- **免费优先**：优先无版权/商用量曲库；用户提供本地文件直接使用（标注来源）；
- **卡点真实可达**：卡点方案必须与 BPM 计算对齐（analyze_bgm.py 输出），不能凭感觉报卡点；
- **质量第一**：BGM 与成片节奏/情绪不匹配时换曲重选，不将就。

## 与其它技能分工

| 技能 | 负责 |
|---|---|
| `bgm-selection`（本技能） | 选曲、分析验证、卡点、混音参数 |
| `bgm-remaster` | 歌词好但伴奏差 → 重新编曲 |
| `video-edit-pipeline` | 混音合成（--bgm 参数） |

## Resources

| 文件 | 内容 | 何时读 |
|---|---|---|
| `references/selection.md` | 情绪→风格映射表、免费无版权曲库清单、选曲检查 | 步骤 2/3 |
| `references/mixing.md` | 音量平衡、淡入淡出、卡点档位、听感检查 | 步骤 5/6 |
| `scripts/analyze_bgm.py` | BPM/时长/响度分析（librosa 优先，纯标准库兜底） | 步骤 4 |

## 引用导航

| 任务/场景 | 查阅文件 | 何时读 |
|---|---|---|
| 需要脚本执行（BPM/时长/响度分析） | `scripts/analyze_bgm.py` | 步骤4 |
| 情绪→风格映射、免费无版权曲库清单 | `references/selection.md` | 步骤2/3 |
| 音量平衡/淡入淡出/卡点档位/听感检查 | `references/mixing.md` | 步骤5/6 |
| 拍点检测（卡点用） | `../beat-sync/scripts/beat_detect.py` | 卡点设计 |
| 合成成片带BGM | `../video-edit-pipeline/scripts/build_video.py --bgm` | 步骤7 |

**UPSTREAM:** 由 `ai-video-director` 在「BGM选配」阶段调用，配合 `video-edit-pipeline` 合成。
