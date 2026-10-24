---
name: beat-sync
description: "卡点剪辑：把 BGM 的打击点/重拍检测出来，让镜头转场对齐到节拍上，实现拍上切换的高级节奏感。提供纯标准库拍点检测脚本（无 librosa 依赖），并让合成器 build_video --beat 自动把 xfade 切点吸附到最近拍点。触发场景：用户说“卡点、卡点剪辑、踩点、切在拍上、BGM节奏对齐、拍上切换、转场踩在鼓点上”；与 bgm-selection（选曲+BPM验证）、video-edit-pipeline（合成）配合使用。"
---

# 卡点剪辑（beat-sync）

## 定位

让画面切换与音乐节拍同步：**切在拍上 = 顺，切在拍间 = 拖沓**。本技能负责拍点检测与切点对齐，是"高级剪辑"的节奏层。

## 流程

| 步骤 | 做什么 | 产出 |
|---|---|---|
| 1 | **选曲验证**：BGM 先过 `bgm-selection/scripts/analyze_bgm.py` 确认 BPM 稳定、与镜头时长可整除 | 可卡点结论 |
| 2 | **拍点检测**：`scripts/beat_detect.py --audio bgm.mp3 --out beats.txt` 得到打击点时间戳；精度档：默认脚本/内置 librosa 够用（Acc2≈75%），强节奏/舞蹈 BGM 要精度上 BeatNet/Essentia（见 align.md） | beats.txt |
| 3 | **切点对齐**：合成时 `build_video.py --beat` 自动把 xfade 转场 offset 吸附到最近拍点（容差 0.35s） | 对齐日志 |
| 4 | **验证**：抽帧/看时间轴确认切换落在拍上；对不上的切点保留原值（不强行吸附） | 验收 |

## 命令速查

```bash
# 拍点检测（BPM + 拍长 + 拍点时间戳 + 3s/15s 对齐建议）
python3 scripts/beat_detect.py --audio bgm.mp3

# 输出拍点文件（每行秒，供规划/合成）
python3 scripts/beat_detect.py --audio bgm.mp3 --out beats.txt

# 合成时自动卡点（--beat 需与 --bgm 一起用）
python3 ../video-edit-pipeline/scripts/build_video.py --clips ./clips \
  --voice voice.mp3 --bgm bgm.mp3 --subtitle sub.srt --out final.mp4 \
  --ratio 9:16 --cut xfade --beat
```

## 硬规则

- **切点落在拍点**：偏差 ≤0.35s 才吸附；不满足的切点保留原值并提示，不强行吸附造成跳帧；
- **BPM 可靠才卡点**：无稳定节拍时 `--beat` 回退不卡点并提示（纯钢琴/纯人声场景）；
- **剧情重音 > 音乐重音**：故事关键切换优先对齐剧情节点（3s/15s/45s），再对齐拍点，冲突时剧情优先；
- **转场时长独立**：卡点只管切点位置，转场时长按节奏档位另定（见 bgm-selection/mixing.md 三档）。

## Resources

| 文件 | 内容 | 何时读 |
|---|---|---|
| `scripts/beat_detect.py` | 纯标准库拍点检测（能量包络峰值→时间戳+BPM） | 步骤 2 |
| `references/align.md` | 卡点对齐规则、容差、使用方式、翻车修复 | 步骤 3/4 |

## 引用导航

| 任务/场景 | 查阅文件 | 何时读 |
|---|---|---|
| 需要脚本执行（拍点检测/BPM/时间戳） | `scripts/beat_detect.py` | 步骤2 |
| 切点对齐规则/0.35s容差/精度档/翻车修复 | `references/align.md` | 步骤3/4 |
| 选曲+BPM可整除验证 | `../bgm-selection/scripts/analyze_bgm.py` | 步骤1 |
| 情绪→风格选曲 | `../bgm-selection/references/selection.md` | 步骤1 |
| 混音/转场时长档位 | `../bgm-selection/references/mixing.md` | 转场时长另定 |
| 合成时自动卡点吸附 | `../video-edit-pipeline/scripts/build_video.py --beat` | 步骤3 |

**UPSTREAM:** 由 `ai-video-director`/`video-edit-pipeline` 在「BGM卡点、切点对齐节拍」阶段调用。
