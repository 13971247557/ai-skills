---
name: video-command-center
description: AI 视频/短剧制作的总控与调度中枢（指挥官层）。任何 AI 视频/短剧任务进入时，先读本技能，按"十阶段流水线"路由到对应板块技能/文件，每阶段过门禁才进下一环。统筹板块：故事方法论（storytelling/story-cases）、剧本、分镜（shot-vocabulary/storyboard）、资产（character-refs/shot-asset-manager/consistency-lock/character-consistency）、平台路由（platform-rules/no-watermark-channels）、生成（doubao-creative-video/drama/ark/xyq）、质检（qc-checklist）、音频（配音/BGM/环境音/口型/卡点）、剪辑合成（video-edit-pipeline/video-use/scene-cut/mediakit）、发布（publish-pack/平台规则）。触发：用户要做任何 AI 视频/短剧/宣传片/参赛片，或新智能体接手视频项目时。禁止：本技能不代替各板块技能执行细节，只负责路由、顺序、门禁和指挥原则。
---

# AI 视频制作指挥中心（总控）

> 一句话定位：**每个板块技能是"车间"，本技能是"调度台"**。拿到需求 → 按十阶段流水线逐环推进，每环产出物被下一环消费，环环过门禁。

## 一、总流水线（十环，顺序不可跳）

| 环 | 阶段 | 干什么 | 调用的板块（技能/文件） | 产出物 | 门禁（不过不进下一环） |
|---|---|---|---|---|---|
| 1 | 选题与故事 | 定讲什么故事：前提、主题、赌注、选题查重 | `ai-short-drama-pipeline/references/storytelling.md`（前提工作台+结构库）、`story-cases.md`（爆款对照）、`topic-market-map.md`（11 类题材饱和度）、`creator-deepdive.md`（获奖/爆款博主深拆） | logline+主控思想+反命题+结构选择 | 一句话 logline 讲得清；结构已选定；非 and-then 平铺 |
| 2 | 剧本与台词 | 写剧本/台词，卡字数密度 | `doubao-creative-drama`（短剧）、`doubao-novel-writing`、`ai-short-drama-pipeline/scripts/dialog_density.py`；打磨/自检：`script-drafting-sop.md`（SOP）、`script-attack-checklist.md`（编剧攻击）、`audience-attack-checklist.md`（51 类刁钻观众）、`multi-dim-check.md`（14 维细节）、`multi-direction-check.md`（8 方向质检）、`character-naming.md`（取名）、`time-space-transitions-open-source.md`（时间流逝/转场）、`era-detail-research.md`（年代制度）、`voice-speed-plan.md`（语速密度） | 剧本全文+分集台词 | 抒情台词≤4字/s（快节奏≤5）；金句可传播；每集情绪闭环；全维自检 PASS（含刁钻观众攻击） |
| 3 | 分镜 | 分镜表：镜头/时长/节拍/转场 | `ai-short-drama-pipeline/references/storyboard-contract.md`（分镜合同）、`shot-vocabulary.md`（镜头词汇）、`prompt-template.md`、`scene-chaining.md`（跨镜衔接/转场提示词）、`camera-movement-open-source.md`（机位/运镜参数）、`director-skill-ecosystem.md`（导演 skill 生态）、`prompt-craft-mindset.md`（手搓提示词工匠：景别→主体前15词→环境→光线句尾15-20字/6节拍动作链/风格块逐字复用）、`ai-short-drama-pipeline/scripts/budget.py`（积分预算） | 完整分镜表（镜号/时间/动作/镜头语言/声音/尾态）+ 积分账 | 时间连续无空档；总时长整数秒；积分够跑（预算≤可用积分）；每镜有承接锚点；跨 SEG 边界有 Last-Frame Seed 衔接 |
| 4 | 资产 | 生成/锁定角色、场景、道具锚点 | `doubao-creative-design`（生成图）、`character-refs`（角色参考图）、`shot-asset-manager`（镜头资产包）、`consistency-lock`（跨镜一致）、`ai-short-drama-pipeline/references/character-consistency.md`（人脸防线）、`asset-system-open-source.md`（资产七类/命名/版本）、`3d-skill-ecosystem.md`（角色资产 3D 方向/MiniMax 角色卡规格） | 角色大头照+全身照、场景锚点、道具素面图、关键帧 | 每镜需要的锚点全部就位；人脸用"大头照+全身照"非多视图；品牌铁律（素面无字）；场景含连续性地标 |
| 5 | 平台路由 | 选生成平台/模型/渠道 | `ai-short-drama-pipeline/references/platform-rules.md`（H3 八段式 vs Seedance 五段式 vs 免费渠道）、`no-watermark-channels`（免费额度监控）、`seedance-prompt.md`、`canvas-open-source-map.md`（画布平台/开源提示词库/seedance-prompt-skill）、`video-skill-map.md`（180 视频 skill 全景/底座选型/模型九强） | 平台+模型+分辨率+积分单价确认 | 免费/积分能跑完；模型支持参考图张数；禁混剪平台合规 |
| 6 | 生成 | 逐镜图生视频/文生视频 | `doubao-creative-video`（Seedance）、`doubao-creative-drama`、`ark-video-pipeline`（API）、`xyq-toolkit`、`image_to_video/text_to_video`；**生成前必过 `skill-supervisor`（含 D2 转场预埋/D3 提示词 7 要素评分卡）**；失败返工走 `prompt-optimizer-models.md`（返工诊断闭环：失败模式→归因→一次只改1-2变量→复评） | 每镜成片素材 | 每镜过五检（见质检环）；失败一次即复盘，禁重复烧额度 |
| 7 | 质检 | 逐镜对照提示词验收 | `ai-short-drama-pipeline/references/qc-checklist.md`（素材定稿质检）、`ai-video-footage-diagnosis`（抽帧/转写/能量分析）、`artifact-preview`（渲染检查） | 质检报告+返工清单 | 品牌无穿帮、人脸无漂移、语速正常、声画同步、镜头不丢 |
| 8 | 音频 | 配音/BGM/环境音/口型/卡点 | 配音：`text_to_audio_plus/audio_to_audio_plus`；BGM：`bgm-selection`/`bgm-remaster`；环境音：`sfx-layer`；口型：`lip-sync`；卡点：`beat-sync`；参考：`ai-short-drama-pipeline/references/bgm-sound-design.md` | 配音轨+BGM轨+环境音轨 | 关键节点进出留白（非全程铺满）；情绪与画面匹配；口型对齐 |
| 9 | 剪辑合成 | 拼接/转场/调色/字幕 | `video-edit-pipeline`（成片合成）、`video-use`（逐字稿剪辑）、`scene-cut`（自动切分）、`byted-mediakit-editing`、`video-localization`（跨语言）、`video-redaction`（敏感词消音）、`ai-short-drama-pipeline/references/scene-chaining.md`（动作重叠剪点/接点留半/Last-Frame Seed 拼接） | 成片（含字幕） | 总时长达标；首尾完整；无未消敏感词 |
| 10 | 发布（暂缓） | 平台适配/投稿/声明 | `publish-pack`（标题/话题/AI声明）、赛事规则（`跳跃视界`等平台文档）、`lark-*`（如需文档交付） | 发布材料+投稿动作 | 用户暂不需要，本环停用；启用时补 publish-automation/post-publish-review |

## 二、指挥原则（贯穿全片，违反即返工）

1. **故事先行**：第 1-2 环没定稿不进第 3 环；细节不替代更强的故事。
2. **资产先行**：任何生成前，该镜需要的角色/场景/道具锚点必须就位（缺锚点=一致性失控）。
3. **一致性优先**：角色用"大头照+全身照"组合（禁多视图做视频参考）；同场景锚点唯一；先改图→再改词→最后才拆镜头。
4. **品牌铁律**：只允许确认过的品牌名，禁一切他牌字样/乱码/水印/袋身文字。
5. **积分纪律**：预算≤可用积分；失败 2h 返还后再补；一次生成即可，禁重复浪费额度（零现金）。
6. **动作桥**：跨镜动作写"前镜尾态→切镜理由→承接锚点→后镜起态"，禁瞬移。
7. **环境音切换**：闪回/现实等时空切换时环境音必须变化，切回时恢复。
8. **BGM 非全程铺满**：关键节点进出留白，重场段落静默只留环境音更真实。
9. **门禁即真相**：每环产出未过门禁，绝不口头宣称完成。

## 三、常见坑速查（来自实战经验）

| 坑 | 对策 |
|---|---|
| 人脸漂移/换脸 | 大头照报白+提示词锁脸型骨相+一镜一动+4-6秒窗口（见 character-consistency） |
| 品牌穿帮（美团/乱码） | 生成阶段就禁品牌词；袋身/车身素面；定稿后抽帧 OCR 复查 |
| 模型不听复杂提示词 | 按平台分支写法（H3 八段式/Seedance 五段式），参考图全名做锚点，动词具体、情绪写微动作 |
| 时长不达标（参赛≥60s） | 第 3 环就用预算脚本算总时长整数秒；超时先补镜头再生成 |
| 语速太快/镜头烂 | 台词密度脚本卡字数；镜头切换写"何时切/因何切/切到哪/切后看到什么" |
| 平台积分不够 | 第 5 环先核免费额度/积分单价；失败返还规则记牢；邀请好友补积分 |
| 竞品混剪被拒 | 完整作品必须单一合规平台生成，保留工作流记录 |

## 〇、操作前技能门禁（最高硬规则：每次实质动作前必跑，不过不动手）

**任何生产动作（写提示词/生成图/生成视频/剪辑/改文档/设计/建模）前，必须先完成 5 步，缺一不可：**

1. **[识别需求→匹配技能]** 用下方《需求→技能匹配表》判定本任务意图类型，列出**可能用到的技能清单**（精准命中，禁止瞎读全部技能）；
2. **[定位环节]** 说出当前在第几环/哪个环节（如"4b 资产-场景"）；
3. **[读路由表]** Read `ai-short-drama-pipeline/references/knowledge-route-table.md` 中该环节行的"必读知识点"，逐条确认；
4. **[读技能]** 第 1 步命中的技能/知识点（及路由表该环节必读项）的 SKILL.md 或 reference 必须**本轮 Read 过**（压缩/换会话后必须重读，禁止凭记忆调用）；
5. **[跑 checklist]** 按该环节"核心铁律 checklist"逐条打勾，全过才动手；任何一条没过→停下来补。

**判定标准**：调用 image_gen/image_edit/text_to_video/image_to_video/mediakit-cli/design-cli/剪辑脚本等任何工具前，若说不出"本任务命中了哪些技能、读了哪份文件、checklist 哪几条已过"，即视为未过门禁，禁止调用。工具失败≠没读过技能，读技能是本步唯一动作。

## 〇·补、需求→技能匹配表（第 1 步识别用，意图优先不按工具名）

| 用户需求意图（关键词/示例） | 主技能 | 辅助技能/知识点 |
|---|---|---|
| 设计一个XX（产品/包装/UI/角色/空间/品牌/视频片头） | design-on-demand（按需设计工坊） | ai-short-drama-pipeline/references/design-methodology-open-source.md、scene-space-design-open-source.md、asset-system-open-source.md |
| 出视频/短剧/宣传片/AI短片 | video-command-center（总控） | doubao-creative-video、ai-video-director、ai-short-video |
| 写剧本/改剧本/台词 | doubao-creative-drama/ai-short-drama-pipeline 剧本SOP | script-drafting-sop、multi-dim-check、audience-attack-checklist、character-naming 等（路由表 2 环） |
| 分镜/镜头/机位/衔接 | ai-short-drama-pipeline 分镜体系 | storyboard-contract、shot-vocabulary、camera-movement-open-source、scene-chaining、director-craft（路由表 3 环） |
| 角色图/场景图/道具图/资产 | doubao-creative-design | character-refs、shot-asset-manager、consistency-lock、scene-space-design-open-source（路由表 4a/4b/4c） |
| 配音/BGM/音效/口型/卡点 | text_to_audio_plus、bgm-selection、bgm-remaster、sfx-layer、lip-sync、beat-sync | ai-short-drama-pipeline/references/bgm-sound-design.md |
| 剪辑/拼接/转场/字幕/调色 | video-edit-pipeline、video-use、scene-cut、byted-mediakit-editing | video-localization、video-redaction（敏感词） |
| 出表格/分析数据 | sheet | doubao-data-analysis |
| 写文档/Word/飞书 | word、lark-doc | lark-markdown |
| 做 PPT | ppt | — |
| 调研/搜索/学习/蒸馏 | general_search、image_search、video_search | skill-distiller（蒸馏成技能）、doubao-sentiment-tracker（舆情） |
| 文件/目录/备份/迁移 | PowerShell/FileBatchUpload | asset-hostrename-copymove 经验 |
| 查时间/日期/定时任务 | calculator、get_current_time、doubao-cron-scheduler | — |
| 清理电脑/磁盘 | doubao-pc-optimizer | — |

匹配逻辑：意图命中多行时全列（如"做AI短剧"命中出视频+剧本+分镜+资产+音频+剪辑），但**只读本次动作实际用到的**（如本次只出角色图→只读 doubao-creative-design+character-refs+consistency-lock，不读音频/剪辑技能）。

## 四、指挥顺序（新任务进入时）

1. 读本 SKILL.md → 确定任务类型（短剧/宣传片/参赛片/短视频）；
2. 每次实质动作先跑【〇 操作前技能门禁】4 步；
3. 按十环逐环推进，每环先读该环板块技能（见上表"调用的板块"）；
4. 每环产出后按"门禁"自查，未过门禁不进入下一环；
5. 九环完成（发布环暂缓） → 交付成片；如需发布再启用第 10 环。

## 五、板块技能索引（本机路径）

| 板块 | 技能目录 |
|---|---|
| 主流程方法论 | `.user_skills/ai-short-drama-pipeline/`（66 篇 references 全量，按任务取用：storyboard-contract/prompt-template/qc-checklist/character-consistency/long-form-pipeline/storytelling/story-cases/shot-vocabulary/shot-design-open-source/storyboard-open-source/scene-chaining/camera-movement-open-source/director-skill-ecosystem/bgm-sound-design/platform-rules/seedance-prompt/canvas-open-source-map/video-skill-map/3d-skill-ecosystem/asset-system-open-source/audience-attack-checklist/multi-dim-check/multi-direction-check/script-drafting-sop/script-attack-checklist/character-naming/time-space-transitions-open-source/era-detail-research/voice-speed-plan/topic-market-map/creator-deepdive/aigc-bloggers/cross-domain-tips/model-prompt-language/cinematic-look-prompt/douyin-prompt-cases/agent-skill-design-pattern 等 + `ai-short-drama-pipeline/scripts/budget.py`/`dialog_density.py`/`script_qc.py`/`script_full_check.py`） |
| 故事/剧本 | `.skills/doubao-creative-drama/`、`.skills/doubao-novel-writing/`、`.user_skills/novel-characters/` |
| 角色资产 | `.user_skills/character-refs/`、`.user_skills/shot-asset-manager/`、`.user_skills/consistency-lock/` |
| 图片生成 | `.skills/doubao-creative-design/` |
| 豆包出片 | `.skills/doubao-creative-video/`（seedance-prompt 五段式）；分镜合同见 `ai-short-drama-pipeline/references/storyboard-contract.md` |
| 免费渠道 | `.user_skills/no-watermark-channels/` |
| 音频 | `.user_skills/bgm-selection/`、`.user_skills/bgm-remaster/`、`.user_skills/sfx-layer/`、`.user_skills/lip-sync/`、`.user_skills/beat-sync/` |
| 剪辑 | `.user_skills/video-edit-pipeline/`、`.user_skills/video-use/`、`.user_skills/scene-cut/`、`.user_skills/video-localization/`、`.skills/video-redaction/`、`.skills/byted-mediakit-editing/` |
| 发布 | `.user_skills/publish-pack/`、`.skills/doubao-newmedia-writing/`（分镜/文案） |
| 孵化与门禁 | `.user_skills/skill-distiller/`（外部技能孵化）、`.user_skills/skill-evolution/`（自检门禁） |
| 选题验证（前置） | `.user_skills/topic-validation/`（选题市场数据验证→选题评分） |
| 系列连续性（多集） | `.user_skills/series-continuity/`（系列档案/伏笔台账/跨集设定追踪） |
| 分镜批量生成 | `.user_skills/storyboard-to-assets/`（分镜表→批量分镜图+视频提示词初稿，`storyboard-to-assets/scripts/storyboard2prompt.py` 已实测） |
| 发布动作（暂缓） | `.user_skills/publish-automation/`（发布材料→实际发片→参赛投稿→工作流证据留存） |
| 发布后复盘（暂缓） | `.user_skills/post-publish-review/`（数据→流失段→迭代项，闭环回流选题） |

## 六、技能选择方法论（怎么选技能最好）

**读 `references/skill-routing.md`**（蒸馏自开源 skill-orchestrator/agentic-skill-router/multi-agent-orchestration/using-bee + Zylos/QVeris 生产架构）。

五条核心原则（详见该文件）：
1. **意图优先**：按任务意图路由，不按技能名/工具名；
2. **两阶段路由**：先检索候选（查 references 目录/关键词）→ 再加载执行；
3. **成本/延迟/权限三因素**：草稿走免费层、定稿才动积分；缺 Key 如实标注不硬调；
4. **链优先于图**：默认线性链，只有"分支需汇合"才用图；
5. **主备 fallback 表**：每类任务列首选→备选→切换条件（如 H3 失败→Seedance/免费渠道），切换靠明确触发条件。

**优秀实战参照**：读 `references/routing-cases.md`（9 个公开案例）——Showspring 12 服务跨 6 厂商分工、《蜀锦一梦》双平台分流+分层提示词模板库、AniShort 双轨并行资产先行、海螺vs可灵同任务质量差决定路由、Google 7 步流水线每步互检门禁、Bark×AWS 三层分离、Fastio 导演 Agent 计划先行、Alibaba 门禁两段式。

## 七、导演手艺（镜头是故事决策）

**读 `references/director-craft.md`**（蒸馏自开源 DirectorSKILL v2.1）。写分镜/提示词前必读。扩展生态查 `ai-short-drama-pipeline/references/director-skill-ecosystem.md`（Flova 三套 skill/SCAI 电影感/电影导演 20 套风格/LibTV + video-shotcraft 124 张镜头配方卡/OpenMontage 七阶段管线）。核心：
1. 镜头功能先行（建立/关系/特写/插入/反应/转场/余波/主观/揭示），说不出功能=删；
2. 景别 6 级阶梯，切镜跳两档，AI 可靠带 MS-CU（EWS/ECU 用静止帧策略）；
3. 机位先定高度再定角度，AI 不认数字——写可见后果（"机位很低，桌边横切画面底部"）；
4. 走位先于取景：人际距离写数字（`2.4m→0.6m over 4s`），权力几何选一件；
5. 提示词转换五步：抽象→一个身体部位+动词+方向+量级→删抽象；
6. 失败不重roll：60 秒分流树定编码（两码→拆镜，三码=三个镜头），成本阶梯 L1-L7 从最便宜修起，三击回上游；
7. 类型 playbook：戏剧引擎+观众契约+七工艺旋钮，偏离>3 旋钮=换类型。
