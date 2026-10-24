---
name: sfx-layer
description: "音效与环境声层：为 AI 短视频/短剧成片垫一层氛围环境声（雨声/心跳/风声/城市底噪/低频嗡鸣）并按场景放置音效点（转场whoosh/重音boom/情绪sting），与配音、BGM 构成三层混音（配音最前、BGM 中、环境音垫底）。全部用 ffmpeg 本地合成，无需素材库、免费。触发场景：用户说“加音效、环境声、氛围声、转场音效、背景雨声/心跳/风声、给片子垫一层底噪”；配合 video-edit-pipeline（合成，支持 --sfx 第三轨）、bgm-selection（BGM 层）使用。"
---

# 音效与环境声层（sfx-layer）

## 定位

成片声音的三层结构：**配音（最前）→ BGM（中间）→ 环境音/音效（垫底与点缀）**。本技能负责最底层与点缀层：

- **环境声（bed）**：持续铺底、建立空间感与情绪基调（雨声/心跳/风声/城市底噪/低频嗡鸣）；
- **音效点（sfx hit）**：在关键时刻放置（转场 whoosh、剧情重音 boom、情绪 sting、动作拟音）。

全部用 `scripts/sfx_generate.py` 本地 ffmpeg 合成，免费、无需外部素材库。

## 流程

| 步骤 | 做什么 | 产出 |
|---|---|---|
| 1 | **读场景定音效**：从分镜/画面判定场景类型 → `references/design.md` 映射表取环境声 + 音效点方案 | 音效方案 |
| 2 | **生成环境声轨**：`scripts/sfx_generate.py --scene <类型> --dur <时长>` | sfx_bed.wav |
| 3 | **生成音效点**：按剧情节点放置（`--hits "时间:类型:强度"`） | 含音效点的完整 sfx 轨 |
| 4 | **三层混音**：配音 + BGM + sfx 合成（`build_video.py --voice --bgm --sfx`） | 成片 |
| 5 | **听感检查**：环境音不抢戏、音效点到位、无爆音 | 验收 |

## 命令速查

```bash
# 生成雨夜环境声 10s（雨声+低频垫）
python3 scripts/sfx_generate.py --scene rain --dur 10 --out sfx_rain.wav

# 带音效点：3.5s 处转场 whoosh、6s 处重音 boom
python3 scripts/sfx_generate.py --scene rain --dur 10 --hits "3.5:whoosh:med,6.0:boom:high" --out sfx_full.wav

# 合成成片（三层：人声 1.0 / BGM 0.4 / SFX 0.18，自动混音）
python3 ../video-edit-pipeline/scripts/build_video.py --clips ./clips \
  --voice voice.mp3 --bgm bgm.mp3 --sfx sfx_full.wav --subtitle sub.srt --out final.mp4 --ratio 9:16
```

## 混音分层（硬规则）

| 层 | 相对音量 | 说明 |
|---|---|---|
| 配音 | 1.0（基准） | 永不盖过 |
| BGM | 0.40（≈-8dB） | 人声下 6~10dB（bgm-selection 规则） |
| 环境音/音效 | 0.18（≈-15dB） | 垫底不抢戏；音效点可单独 0.5~0.8 峰值 |

- 环境音**始终低于 BGM**；音效点在关键时刻可短暂突出但不破音；
- 无配音段（纯氛围）环境音可提到 0.3（-10dB）；
- **--sfx 对接口径**：build_video.py --sfx 把 sfx_generate.py 产出的**整条 sfx 轨（bed+音效点已混在一起）**再乘全局音量 0.18（有配音）/0.30（无配音）；design.md 的 low/med/high=0.30/0.50/0.80 是这条轨内部 bed↔音效点的相对强弱，不是最终混音电平——音效点想更突出就把档位调到 high，不要调大 0.18（会连环境音一起顶上去）；
- 成片响度统一 loudnorm -14 LUFS（build_video.py 已内置）；
- 悬疑/惊悚场景低音为主，燃向场景音效点密度高，情感场景留白多（见 design.md）。

## 硬规则

- **环境音不抢戏**：垫底音量默认 0.18，宁可小不可大；
- **音效点按剧情放**：只有关键时刻才放，滥用音效 = 廉价感（用户硬要求：出片质量第一）；
- **分层清晰**：配音/BGM/环境音三轨独立生成再混，不把环境音塞进 BGM 轨；
- **downlifter 本机不合成**：`sfx_generate.py` 支持 whoosh/boom/sting/riser/tick，**downlifter 需外部素材**，不硬凑；
- **音效点不与 BGM 鼓点打架**：boom/sting 与底鼓错开 ≥0.1s；
- **交付前听感检查**：环境音无突兀、音效点无爆音、整体无刺耳。

## Resources

| 文件 | 内容 | 何时读 |
|---|---|---|
| `references/design.md` | 场景→环境声/音效点映射表 + 音效点强度档 | 步骤 1 |
| `scripts/sfx_generate.py` | ffmpeg 合成环境声 + 音效点（纯本地、免费） | 步骤 2/3 |

## 引用导航

| 任务/场景 | 查阅文件 | 何时读 |
|---|---|---|
| 需要脚本执行（环境声bed+音效点hits） | `scripts/sfx_generate.py` | 步骤2/3 |
| 场景→环境声映射、音效点强度档(low/med/high) | `references/design.md` | 步骤1 |
| 配音/人声轨判定与音色 | `../video-edit-pipeline/references/audio.md` | 配音最前层 |
| BGM中层配比(压人声6~10dB) | `../bgm-selection/references/mixing.md` | 混音分层 |
| 三层混音合成(--sfx第三轨) | `../video-edit-pipeline/scripts/build_video.py` | 步骤4 |

**UPSTREAM:** 由 `ai-video-director`/`video-edit-pipeline` 在「声音三层混音、垫环境氛围」阶段调用。
