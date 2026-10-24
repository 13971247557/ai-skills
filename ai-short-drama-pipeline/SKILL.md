---
name: ai-short-drama-pipeline
description: AI 短剧/短视频成片出片全流程技能（含图生视频平台）。覆盖：剧本创作与台词优化、分镜规划、角色/场景/道具资产锚点体系、每镜图生视频提示词编写（全局设定/参考图用途/第0秒锁定/画面时间线/配音/环境音/BGM 八段式）、积分预算核算、素材一致性质检（OCR品牌穿帮/背景/脸/服装/道具）、BGM 非全程设计、成片后抽帧质检。触发场景：用户要做 AI 短剧、AI 短视频、微短剧、视频脚本分镜、图生视频提示词、跳跃视界/MiniMax H3 出片、雨夜外卖/亲情短片等任何 AI 视频创作项目；或需要"提示词怎么写给视频模型听懂、资产怎么锚定人物一致、积分怎么规划够用、成品怎么质检"。不用于：纯文本文案写作、长视频后期剪辑工程（剪映/CapCut 操作）、真人实拍拍摄。
---

# AI 短剧出片全流程（视频skill）

通用方法库：适用于任何 AI 短剧 / AI 短视频成片项目（跳跃视界、可灵、即梦等图生视频平台）。以"做什么→按什么规则→用什么模板→怎么验证→交付什么"组织，所有项目可复用。

## 0.0 操作前技能门禁（硬性SOP，不可越过——用户2026-10-18铁律）

**任何实质动作（调工具/写prompt/改文件/渲染/生成/搜索/修复）前，必须先走 `references/mandatory-skill-gate.md` 门禁五步：识别任务→查`skill-routing-quick-ref.md`命中技能→本轮Read命中技能→对路由表该环节→跑checklist全过才动手。** 说不出"任务类型/命中哪份技能/本轮Read了哪份/checklist过了哪几条"=未过门禁=禁止调用工具。本SOP不可越过，违者即使结果正确也算流程违规。

## 0. 项目初始化（每项目必做）

1. 建项目根目录，固定四个资产子目录：`关键帧/`、`角色/`、`场景/`、`道具/`；另建 `剧本-<片名>.md`、`分镜资产适配方案.html`（或 md）。
2. 先定三个全局事实并写入剧本头部，全程不许改：
   - **剧情护城河**：悬念点（谁知道/何时揭晓）、时间线（现在 vs 闪回）、情感转折顺序——写下来，任何分镜/提示词改动不得破坏它。
   - **品牌铁律**：允许出现的品牌名（如"美兔"）与**禁止清单**（真实平台名"美团"等、水印、乱码、字幕、任何袋身/车身/服装上未经授权的文字图案）。
   - **总积分预算**（如 585）与总时长目标（按故事实际，不硬压 30s）。
3. 平台选型：图生视频平台（如跳跃视界 MiniMax H3 Max）选**直达"视频生成"模式，不选 Agent/剧集工具**（省资产生成积分）。

## 1. 剧本与台词规则

- 台词写完后逐句核：**密度 ≤5 字/秒**（含停顿），超了就删字或加时长；用 `scripts/dialog_density.py` 检查。
- 台词按人声特性写：年龄/嗓音/情绪/语速/停顿都写进配音段（例："45岁中年女声，沙哑带哭腔，约4.2字/秒，越说越颤"）。
- 用**并行**塞时长：台词与动作并行（"他偏头不看她，咬肌绷紧"与母亲台词同时），比串行省秒数。
- 关键动作、微表情、手的拿法、视线方向、走路/骑车姿态，全部写成可观察细节（"左手搭上扶手再抓牢""视线在屏幕和门牌间来回两次"）。视频模型"笨"，不给细节它就自由发挥。
- 时长规划：按段落估算每镜秒数→整镜取整数（平台输入无小数点）→ 每段允许 ±20% 浮动写进提示词（"不赶拍"）。

## 2. 资产锚点体系（一致性根基）

每个"会反复出现"的对象都要有**锚点图**，生成视频时作为参考图上传：

| 对象 | 锚点图要求 | 用途 |
|---|---|---|
| 角色 | 四视图 + 表情图 + 全身形象图（或合并"全集合成图"） | 人物形象锁定 |
| 角色分时期 | 闪回（年轻/旧装）与现在（制服/新装）**必须分开建锚点**，各配一张 | 防止时代错乱 |
| 场景 | 每个场景一张"干净空镜锚点"，跨镜复用 | 背景一致性唯一基准 |
| 道具 | 素面图（袋身/车身禁任何文字） | 道具款式锁定 |
| 关键帧 | 每镜首帧（构图/机位/光线）+ 需要的尾帧 | 首尾帧衔接 |

**硬规则**：
- 同场景多张关键帧背景不一致时，用**单一已验证场景锚点图作唯一参考**重绘各镜（统一机位/灯光/道具连续），不要各镜各画。
- **尾帧 = 下一镜首帧**（同图或同场景构图），两镜衔接零跳变；平台支持首尾帧输入时，尾帧位填衔接图。
- 角色锚点跨镜必须同一张脸：生成新视角图时以**已确认的角色关键帧为唯一身份母版**（只传母版一张参考），不要混第二张"构图参考"，否则模型会画成两张脸。
- **人脸固定三层防线**（防卡脸/漂移）：① 每个反复出现的人物先出**清晰正面大头照**上传资产库审核报白，固定人脸特征；② 提示词每镜写死"人物身份以<图片N>大头照为准，锁定脸型/骨相/五官比例/发际线/肤色，不得美化替换"；③ 视频生成人物参考只用**大头照+全身照组合**，**不用多视图/四视图/表情拼图**作视频参考（多视图仅用于图生图重绘造型/服装）。分时期角色各建一套。详见 `references/character-consistency.md`。
- 人脸排查补充（调研版）：先分清段内扭曲/跨镜漂移/换脸闪烁三种失败；出问题**先改图→再改词→最后拆镜头**；身份描述当常量逐字复用、换装限 3 个风格关键词；每镜一镜一动、慢运镜、4-6 秒窗口；平台功能优先级＝参考图固定 > 首尾帧 > 分镜继承 > 纯提示词。
- 新增资产命名 `编号-角色/场景/道具-描述.jpg`，落到对应文件夹，生成即入画布留档。

## 3. 每镜提示词八段式模板

每镜提示词固定八段（顺序不许乱），整段可复制粘贴给平台：

1. **【全局设定（全片通用）】**——画幅/色调体系/运镜总则/对白语言/品牌铁律（禁什么字样）
2. **【参考图用途】**——每张上传图"干什么"逐张写明；**只作动作参考的图必须写明"背景一律以XX锚点为准"**
3. **【第0秒锁定】**——首帧画面完全复刻哪张图，仅让动作从那帧开始
4. **【画面时间线】**——分段落：每段（时长，镜头）＋主体动作＋微表情＋道具细节
5. **【光线与画面质感】**——每场景色调/光源/景深规则/焦点跟随原则
6. **【配音】**——每句台词：人声属性＋时长＋语速＋情绪＋原文
7. **【环境音】**——每段的环境音层（雨声/脚步声/门声/提示音），**闪回进出必须写明环境音切换**
8. **【BGM】**——按情绪弧线设计进出点（见 §7），总规则：BGM音量<环境音<对白

完整模板与示例见 `references/prompt-template.md`。

## 4. 每镜参考图上传规则

- 参考图 ≤ 平台上限（H3 为 9 张/镜），留 1 张余量。
- 上传顺序 = 使用优先级：**首帧图第 1 位**，随后是场景锚点→动作参考→角色→道具。
- 跨镜复用锚点（角色、场景、关键道具）必须出现在每一镜，锁死跨镜一致性。
- 尾帧位：平台支持就填衔接帧，不支持则在提示词【第0秒锁定/时间线】写明"画面停在此帧=下镜首帧"。

## 5. 积分预算与可行性核验

- 参考单价（H3：480P≈7分/秒、768P≈15、1080P≈30）；**以平台"预计消耗"实时为准**，首镜发出后核对再定分辨率组合。
- 用 `scripts/budget.py` 核算各镜"秒:分辨率"合计 ≤ 总预算，留 10-15% 兜底。
- 生成失败被扣积分通常 X 小时内自动返还；每日登录赠分仅当日有效——写进方案备忘。
- **一次机会原则**：免费/限量额度不重复生成。首镜用较低分辨率试发，确认提示词有效再依次生成。

## 6. 素材质检清单（出片前必跑）

出片前逐项核对，见 `references/qc-checklist.md`，关键点：
- 关键帧 OCR 查品牌/乱码/水印（袋身/车身/胸前只允许规定字样）
- 背景是否与场景锚点一致（同楼道/同门/同福字位置）
- 角色脸/服装/年龄跨镜一致（尤其闪回 vs 现在分时期）
- 道具款式跨镜一致（袋/车/保温箱）
- 有问题的图：先完成交付闭环留档，再基于"唯一锚点+明确改哪"重绘，禁止静默丢弃

## 7. BGM 与环境音设计

- **非全程铺满**：关键节点进出＋重场段落留白（争吵/紧张段静默只留环境音更真实）。
- 情绪弧线→BGM 动机：同一 2-3 音钢琴动机贯穿，转暖时转大调；台词时 BGM ducking 退让。
- 环境音是"空间感开关"：闪回进=环境音切换（雨声敞亮→隔窗闷响+老钟滴答），切回现实=恢复（雨声变大+风声电机声）。
- 详见 `references/bgm-sound-design.md`。

## 8. 成片质检（生成后）

- 抽帧 + OCR：查品牌穿帮/乱码/字幕；音频转写核台词是否与剧本一致、语速是否超。
- 声画对齐：台词时长 vs 镜头时长；环境音切换点是否符合提示词。
- 片尾字幕等后期文字，不在提示词里要（图内文字不可控），由剪映等后期加。
- 首镜出来后先质检再继续下一镜，不要一次生成全片才发现方向错。

## 9. 交付规范

- 主交付物：方案 HTML/md（每镜：参考资产清单＋完整可复制提示词＋上传顺序＋时长＋分辨率建议＋预算核算＋平台操作路径）。
- 每次更新提示词后，把最新全文贴给用户方便复制（用户偏好）。
- 生成类图片/视频产物：画布持久化＋present_files 双通道，不静默丢弃失败稿。

## 学习沉淀自动流（所有学习步骤的统一机制）

> **每次学到任何新知识/技能，都必须按此流程沉淀成"可加载运行的技能"，放入本技能库相应文件夹，禁止学完就丢。** 这是"学习→加载→运行"的自动流 SOP，与 knowledge-route-table（环节→知识点自动路由）配套。

1. **学习**：收集来源（网页/视频/教材/实操踩坑），提取可追溯结论。
2. **沉淀**：跑 `python scripts/learning_deposit_sop.py <技能名> --new "主题一句话" --route <环节>` 生成技能文件骨架到 `references/`，再编辑填充【来源/结论/铁律/落地/引用导航】五段（参考 `scene-3d-pano-modeling-open-source.md`）。
3. **加载**：在 `references/knowledge-route-table.md` 对应环节行的【必读知识点】列加入该技能名，使进入环节时自动 Read 加载。
4. **运行**：在 SKILL.md 资源索引表加一行（何时读该文件），并登记到 scripts 引用区。
5. **校验**：重跑 `python scripts/learning_deposit_sop.py <技能名>`，四项 PASS 才算沉淀完成；缺一项按指引补齐。

## 资源索引

| 文件 | 何时读 |
|---|---|
| `references/storyboard-contract.md` | 写/改完整分镜表时（表头/规则/客观校验） |
| `references/prompt-template.md` | 写任何一镜提示词时 |
| `references/qc-checklist.md` | 素材定稿后、出片前质检时 |
| `references/character-consistency.md` | 处理角色人脸一致性/卡脸/漂移、生成角色锚点图时 |
| `references/character-naming.md` | 给角色定名/改名/校验名字合理性时：五条铁律（主题装置/反讽命名/时代真实/值得被记住/避梗保区分度）+命名四步法+时代命名速查表+《她的名字》应用诊断 |
| `references/storytelling-open-source.md` | 剧本层开源补强：9 转折点标准/3 秒钩子三类型/微短剧三幕适配/门框蒙太奇/故事圣经（screenwriting-skills+北电+澎湃） |
| `references/vfx-music-competition-open-source.md` | 特效转场/AI配乐/比赛投递时：首尾帧桥接+look bible+配乐工作流+评审评分权重（Pika/ai-music-street/爱奇艺/联想AI电影季） |
| `references/storyboard-art-qc-open-source.md` | 出分镜图/逐镜验收时：镜头描述句式+3×3网格+拼帧法+6轴rubric+帧差探测（Astorie/storyboard-generator/review-video/ContinuityGuard） |
| `references/color-voice-open-source.md` | 设计色彩体系/写配音指令时：色温情绪+teal-orange+肤色保护+配音表演清单（呼吸/省略号/质感词） |
| `references/audience-montage-open-source.md` | 设计钩子/伏笔节奏/跨镜叙事时：好奇心缺口+蔡格尼克+蒙太奇语法（时间压缩/平行/隐喻） |
| `references/asset-system-open-source.md` | 搭资产库/定资产清单时：命名法+资产七类表+角色包标准+状态变体拆分+动作光线资产+Brand Bible（Seedance官方+Neolemon+asset-passport） |
| `references/asset-anchor-frames-open-source.md` | 做资产清单/锚点图/定帧/动作提示词前：资产分层三步定案法+头部创作者角色包标准（6-12张+首尾帧）+GitHub开源一致性方案（MoCha/BindWeave/ContextAnyone/Ovi/LTX）+必做4类帧+动作锚点两级（参考视频/提示词）+镜头语言速查（dolly vs zoom/景别情绪/构图） |
| `references/camera-movement-open-source.md` | 设计对话戏机位/定运镜时：180°轴线+三角形三机位/运镜情绪语言（push-in=重要）/H3运镜词义精确定义/运镜连接目的 |
| `references/asset-design-open-source.md` | 做场景/道具/元素锚点图前：establishing shot锁场景/一墙静帧/道具六阶段+Seedance资产表/双光色/粒子元素 |
| `references/performance-lipsync-open-source.md` | 写有对白镜头时：口型铁律（中近景/正面/单句≤4秒/逐句同步）+Seedance口型两种写法+音素库 |
| `references/editing-sound-model-open-source.md` | 转场节奏/声音三层+留白+静音/模型选型参数：J-cut/L-cut/叠化分档/动作峰值剪切/Seedance 2.5 多参考 |
| `references/auto-editing-open-source.md` | 剪辑/合成环节选方案时：豆包↔剪映自动剪辑工作流（剪映editor skill）+GitHub开源工程（CapCutAPI/CapCut Mate/pyJianYingDraft/jianying-headless/vectcut-api）+卡点/字幕/转场多技巧+成片流水线 |
| `references/time-space-transitions-open-source.md` | 写剧本/分镜需体现"时间流逝/切换场景时间人物"时：七类时间证据（光/声/物理/环境累积/循环变化/蒙太奇/标题卡）+转场库（非技巧：出入画/视线/声音/相似体/闪回；技巧：叠化/淡入淡出/匹配剪辑/虚化）+连续性铁律（180度/30度/视线/动作匹配）+ASL节奏+对《她的名字》每场转场落地表（来源 Fiveable/北大影视剪辑/今敏千年女优/ScreenWeaver/ScreenwritingCommunity） |
| `references/shot-design-open-source.md` | 写每镜提示词锁高级观感：视觉语言六要素/焦段情绪/机位权力/景深引导/电影质感参数/获奖案例（nolanx+CinePrompt+Cinemastery+狂飙+断鞘） |
| `references/storyboard-open-source.md` | 写分镜表/每镜提示词前：nolanx 镜头规则+清华栏目+提示词长度规范+Higgsfield 七段骨架+构图八法+镜头时长原则 |
| `references/storyboard-feasibility-open-source.md` | 分镜表可行性检验时：一镜一信息点+三秒镜头测试+30度角反向用（AI对话拼接）+连续性压力点+文本验收五问+验收七维+图内文字统一处理+节奏基准+获奖案例（种子/Meridian/最后一班/金鱼） |
| `references/multi-direction-check.md` | 交付前 8 方向全量质检（逻辑/情绪弧线/三幕/节奏/留存/技术/合规/观众），来源 CloudKOL+Umbrella+Hooked+BFA+arXiv |
| `references/multi-dim-check.md` | 任何交付前按 14 维自检（编剧逻辑/叙事连续/空间技术/穿帮扫雷四层），来源 No Film School+arXiv EvalVerse+穿帮案例 |
| `references/era-detail-research.md` | 剧本进视频前查时代真实性：档案法规（本人不得查阅/涂改自己档案→违规重量）/登记表真实版式/80-90年代档案室场景一手素材 |
| `references/scene-consistency-error-driven.md` | 场景多机位/多镜头反复不一致、机位派生失败、AI 乱码时：先调研开源提示词+实操案例再修复（已实测结论表/每镜 prompt 链模板/改视角句式/去文字句式/逐镜验证闭环/Skywork loop），禁止无脑重试 |
| `references/prop-continuity-registry.md` | **道具/场景可操作性（道具连续性注册表）**：每个关键道具=ID+物理描述+位置+镜号+禁用项+状态版本（Frank Houbre）；固定/可动分离+光照状态（AeliaVision）；状态变化只在段内（SkillMP）；道具按功能/结构/材质/尺寸/使用方式/状态版本标准化（窦继雯）；剪辑台逐镜KO检查——**资产是分镜的可操作舞台，不是摆设** |
| `references/greybox-whitebox-advanced-sop.md` | **白模/灰模高级技巧（做灰模/白模走位预演/灰盒验收前）**：粗/细颗粒度白模双模式+Seedance提示词模板（映射块承重墙）、灰盒验收6问（SEELE）、DESIGN-FOR-AI硬规则（首帧全元素/一镜一动作/双guard射线检测）、Blender→AI三步流水线（Flick：blocking→首帧渲染→运动参考）、白模走位编排（updream/TapNow实测：白模锁运镜构图）、DZS空间预演协议（三状态表+冲突仲裁+7预演技术）、AI+实体混合（锁定底板库）——**让AI负责完成，不是负责猜** |
| `references/lighting-asset-execution-sop.md` | **光线资产执行包（出档案室5档光L1-L5前）**：每档光定义（清晨斜光/日间冷白/午后移光/黄昏/深夜台灯单光源）→i2i只换光几何不动→每档光提示词槽Copy-ready→阴影落向核验→L5台灯位置阴影侧整段逐字固定（身份串）→cinematic五问+九项门禁；用户验收后才算定稿 |
| `references/scene-geometry-gate-sop.md` | **任何场景资产生成前后**：先读平面布局参数表+机位空间核对清单+下载灰模对应机位图（三读三对），灰模=几何权威（按平面图生成）、AI 图必须继承灰模几何只换质感；生成后并排核验桌数/柜列/窗数/门位/道具位/时间光，任一不符失败即停→调研→蒸馏→修复 |
| `references/camera-style-prompt-library.md` | **分镜→提示词映射库（写分镜/出片提示词前）**：14种镜头风格模板（景别×角度×运镜×光影×氛围，直接填槽可用）+光影风格库6种+《她的名字》代表性镜号映射示例+应用方法（每镜查表→填槽→验证） |
| `references/error-prevention-checklist.md` | **操作前先读（任何场景/道具/建模/渲染/生图动作）**：错误预防库13条历史错误（柜列数/桌位/暖水瓶位置材质/桌面道具缺失/柜色穿帮/相机遮挡/门外无光/否定词/固有认知/计数/灰模不按平面图）→命中即停→按修复动作；操作前三问（对照参数表逐项/视线遮挡/否犯过） |
| `references/performance-acting-prompt-library.md` | **写人物表演类提示词前**：三层表演法（行动逻辑→表演节拍→面部动作时序）+微表情防塑料写法（毛孔/泪沟/纹理）+走位文字确认→锁提示词；《她的名字》4处应用示例 |
| `references/action-asset-execution-sop.md` | **出7项关键动作资产（开锁拂灰/倒水递回/落笔盖章/蘸印泥/小周写名/同事B接递/握笔迟疑）前**：姿势帧拆解（起点→过程→结果）+像素变化语言（禁抽象词）+道具状态段内连贯+手部服装锚点=角色母版+与穿插对应表逐镜绑定；帧间承接锚点防跨镜瞬移 |
| `references/shot-prompt-compiler-sop.md` | **84镜提示词编译执行包（出片前批量编译84镜提示词）**：分镜表+shot-ref-map.csv→每镜完整五段式（参考声明/对应映射/剧情详述/场景处理/整体收束），`临时工具\build_shot_prompts.py`自动编译+人工核验差异槽；白模8镜自动拼映射块；剧情详述逐字来自分镜；身份块/收束块全片逐字复用防漂移；负向句尾必挂；参考图URL必须来自已验收资产 |
| `references/first-frame-composition-sop.md` | **每镜首帧合成执行包（图生视频前）**：首帧=场景空镜+角色（位置=参数表/朝向=分镜契约）+道具（初始状态）同框；白模8镜走Blender首帧→i2i质感化（构图姿态100%不变）；一镜一帧禁塞整场；先样板3镜验收→再批量；i2i低strength保几何只换质感；零文字 |
| `references/sound-asset-execution-table.md` | **声音资产批量生成执行表（音频环前）**：清单v7第6节16项声音（钥匙/纸张/灯管/铃/脚步/心跳/盖章/倒水等）逐项生成入库（sfx-layer/seed-audio）→按镜号挂载（对照分镜声音列）；操作音与动作同步/环境音闪回切换/前7场零配乐/重场留白/静音段标注；资产复用禁逐镜重生成 |
| `references/camera-position-derive-sop.md` | **建灰模/定机位前**：从分镜逐镜归纳机位（禁凭空定数），场景每镜号必归某机位，全景/中近景/特写三层覆盖（调研蒸馏：blocking先于镜头/三层覆盖系统/shot list结构/180°轴线）；《她的名字》档案室13机位全镜号分配表（v4含方向维+入画出画硬门禁） |
| `references/camera-lens-execution-boundary.md` | **定机位/写镜头提示词前**：焦段选择表（广角=空间/长焦=压缩凝视）+机位高度↔权力语法+光位-机位联动（伦勃朗/顶光=体制压迫）+AI镜头执行边界（一镜一运镜/5-8s分块/≤2-3主体/MS-CU可靠区间/锁定机位显式声明）；《她的名字》13机位焦段/高度/光位锁定表 |
| `references/ai-action-execution-boundary.md` | **含人物动作/道具操作的镜头提示词前**：AI做不好的动作风险表+starts/ends姿态模板（拿放/倒水/落笔直接抄用）+空间锚定（静止参照物防背景融化）+物理约束（恒存/重量/液体）+失败模式标注法（一次只标一个主导模式）；《她的名字》操作镜落地模板 |
| `references/era-image-texture.md` | **年代戏/空镜提示词质感后缀**：写具体胶片名禁vague vintage（35mm grain/褪色化学/失对比/1980s variation/Kodachrome 80s）；《她的名字》四段（档案室/1986/1990-2000/现代）质感块锁定+逐字重复工程 |
| `references/cinematic-quality-gate.md` | **AI图/空镜/镜头交付前审美验收层**：电影感五问（story/emotion/angle/size/lens）+写实九项（光方向/影/肤色/眼神光/手/背景/色板/景深）+构图三维度（景别/主体/空间分层）+可信度两杠杆（单光源定律/受控不完美）；《她的名字》空镜验收三层（几何/质感/审美） |
| `references/execution-photoreal-i2i-previz.md` | **执行层（灰模→照片级AI图+白模走位预演→AI视频）**：i2i strength档位表（低0.25-0.35保几何换质感/3D=真相源AI只加材质光）；Seedance 2.5白模引导工作流（Blender人偶blocking→白模视频作参考→提示词不写镜头语言）；《她的名字》8个白模预演优先镜 |
| `references/voiceover-prompt-library.md` | **角色配音/旁白生成前**：配音提示词五要素（声线/情绪/节奏/场景/语气动作）+平实指令式写法（slower/warmer/holding back tears）；《她的名字》五角色声线锁定表（秀兰60沙哑克制/小周清亮/领导威严） |
| `references/title-sequence-design.md` | **设计视频片头标题卡**：类型→字体→动画→质感表（年代=粗衬线盖章体+慢速+印刷肌理）；设计原则（字体讲故事/色彩单点/质感统一）；《她的名字》片头方案（登记表盖章体红印色） |
| `references/batch-pipeline-orchestration.md` | **大规模批量生成编排（84镜/13机位/批量配音）**：批次安全模式（先锚点审批→再批量/一次性拒绝技术失败/漂移标记重试）+队列五态+重试退避（3次5/15/45s/fallback/2h返还）+成本manifest；《她的名字》按场分批10批+跨场首尾帧衔接 |
| `references/ai-clip-editing-execution.md` | **AI片段剪辑拼接执行（84镜拼成片）**：转场优先级（硬切→匹配→桥接→声音→短叠化）+匹配剪辑AI制作法（形状匹配首选/动作匹配慎用）+pre-roll手柄+trim减速尾+速度匹配叠化（快3-5帧/慢8-12帧）+J/L-Cuts/cutaway+节奏呼吸；《她的名字》SH083形状回收/SH061闪回三连匹配蒙太奇 |
| `references/era-sound-design.md` | **年代环境音设计执行**：声音三层（底噪/操作音/氛围）+档案室声音清单（日光灯嗡鸣/翻纸/盖章/锁鼻/电话铃逐镜对应）+闪回环境音突变（0.5s渐弱）+免费素材来源；配合bgm-sound-design+sfx-layer |
| `references/color-grading-execution.md` | **成片调色执行**：correct先于grade（曝光/白平衡/对比度统一→创意look）+锚点基准法（每场景2-3锚点镜按组拉向目标/禁全片一键look）+质感块映射表（档案室冷白/1986暖黄/1990暖绿/2000白炽/片尾clean）+闪回色调硬跳变+do-not-change几何锁死；配合era-image-texture |
| `references/flashback-scene-construction-sop.md` | **闪回场景构建SOP（1986/1990/2000三场景）**：三重锁定（置景时间证据+色调质感+环境音）+画幅技巧（4:3=怀旧信号）+转场（动作匹配/声音先入/色调硬跳）+减龄独立派生（禁链式递减）；落地三场景物件清单；配合era-detail/era-image-texture/age-transformation |
| `references/subtitle-caption-execution.md` | **字幕设计执行**：三类字幕清单（年代字幕SH028/040/042/红字关键帧SH026/031/033-035/049/片尾SH084）+样式统一/位置/动效/质感+0文字铁律（AI图内禁文字，关键字样全走后期叠加）；执行=byted-mediakit-editing |
| `references/skill-routing-quick-ref.md` | **任务类型→技能自动调用速查表（任何动作前第一件事）**：识别任务→命中技能→Read→checklist→才动手；15类任务精匹配（场景空镜/机位/平面图/3D灰模/道具/角色/一致性/prompt/视频/质检/调研/设计/文档/失败即停/交接）；**由skill-command-center总控调用** |
| `references/realism-global-principle.md` | **全局最高前提（任何环节先读）**：真实感四支柱（材质/光/相机/运动）+AI听得懂的翻译规则+各环节落地+AI感负面清单；产出先问"像人做的/人拍的吗" |
| `references/prompt-commander-total-control.md` | **提示词总控（写任何生成prompt前）**：需求→识别生成类型→查提示词资产库→路由调取模板→填槽→验证；模板库双轴分类+通用结构兜底+沉淀门禁 |
| `references/prompt-language-open-source.md` | 写生图/生视频提示词前：AI 听得懂的提示词语言（CLIP 否定词陷阱→正向替代/词序权重/抽象词防歪/约束断路器/Lexicon Lock-in 词汇锁定/主体定义句式/失败归因表+正向替代词表/本项目落地标签），解决"写了无编号反而画编号、暖水瓶画成灭火器" |
| `references/knowledge-route-table.md` | **总控自动路由表（进入任何环节前必读）**：环节→必读知识点→铁律 checklist，十个环节全覆盖（选题/剧本/分镜/资产-角色/资产-场景/资产-道具/平台/生成/质检/音频/剪辑），禁止凭记忆动手，逐条打勾后才执行 |
| `references/mandatory-skill-gate.md` | **操作前技能门禁·硬性SOP（任何实质动作前必读，不可越过）**：五步（停识别→读skill-command-center总控定技能链→查skill-routing命中→本轮Read→对路由表→跑checklist全过才动手）；判定标准=当场说不出任务类型/命中技能/本轮已读/checklist即未过门禁 |
| `references/skill-command-center.md` | **技能总控中心v2（唯一协调入口·决策层）**：三层分工（决策层本文件+门禁/调度层速查表+路由表/执行层技能·确定性边界）+任务→技能链总表（12类带**数据流产出→消费+失败路由+成本分层**）+防错双检（漏检/同名能力决策表/成本铁律）+总控门禁；全局状态=镜头生产追踪表 |
| `references/long-form-pipeline.md` | 做 5 分钟以上长片（参赛片/短剧）时：分段锚定工程全流程 SOP |
| `references/case-studies.md` | 做片前对照真实获奖/爆款案例（合龙/纸手机/Kite/凌霜/被裁掉女孩）的团队、成本、工具链、创作方法 |
| `references/age-transformation-open-source.md` | 做年代版/闪回减龄加龄资产前：分阶段老化（4-6阶段/20年跳可行50年失败）+数字年龄+共享锚点逐项+禁主观形容词+皮肤纹理+减龄/加龄提示词模板+失败检查清单（arXiv TimeWeaver/Astorie/Runway/Selfielab） |
| `references/creator-workflow-cases.md` | 做资产/场景/道具前对照头部创作者实操：两层锁（外观只定义一次+参数隔离）/三面板角色表/keyframe-first地点关键帧/道具块逐字复用/预演法/场记状态库（掘金两层锁/Higgsfield/卢浮宫小猫/Frank Houbre/VideoClaw/Oakgen） |
| `references/shot-vocabulary.md` | 写分镜提示词时查运镜/转场/节奏/开场/片尾动效词汇（蒸馏自开源 video-shotcraft） |
| `references/storytelling.md` | 写剧本/选故事前：前提工作台（讲什么故事）→结构工程（怎么讲）→形式视角（用什么方式讲）→AI短剧5步落地→多元结构库（Save the Cat/英雄之旅/Freytag/Fichtean/情绪曲线/冲突×2反转）+节拍设计启发式（Hook/Middle/End 三区选项清单）+决策点写作法+讲故事深度补丁（前提可证伪测试/主题=道德论点/象征网络/角色三层/Show vs Tell/钩子五特征/情绪契约/渐进输出） |
| `references/script-drafting-sop.md` | 剧本创作流水线主入口：7 步硬门禁 SOP（选题调研→前提工作台→Story Spine→节拍设计→主角引擎→场景写作→字数核算→质量诊断→可行性三关→模拟观众→交付前攻击自检）+伏笔账本门禁，每步产出物+门禁，配合 scripts/script_qc.py |
| `references/script-attack-checklist.md` | 剧本交付前攻击自检（55 问 10 类）：像苛刻观众/编辑逐问攻击，全过才交付 |
| `references/story-masters-review.md` | 剧本定稿前用故事大师框架专业审读：麦基（每场翻转/Gap/危机/主动主角/一句话意义）+菲尔德（三幕/前10%建立/需求驱动）+斯奈德（15节拍表含百分比定位）——回答"好不好/能不能讲/听得懂/bug"四问 |
| `references/script-vibe-polish.md` | 逻辑全对但"感觉怪"时：观感层打磨 16 大奇怪源+三层定位（文本/潜台词/语境）+双职责测试 |
| `references/script-story-weirdness.md` | 故事本身"感觉不对"时：剧本层 16 大奇怪源（智商波动/行为跳人设/巧合堆叠/时间线/动机空转/世界规则/失忆藏信息等）——与观感层互补 |
| `references/script-impact-analysis.md` | 交付前爆点与留存分析：爆点位置/配重/类型/前置 + 冲突六项（对手贯穿/升级阶梯/代价可见/张力曲线/主动出手/假冲突）+ 留存六项（四段诊断/逐段扫描/3秒测试/45秒法则/最陡定位/留存理由表） |
| `references/script-to-storyboard.md` | 剧本定稿后转分镜时：字数→秒数换算、节拍→镜头数分配、台词→镜头分配、AI 生成可执行性（对白≤2句/镜、画外音优先）、一致性锚点前置 |
| `references/script-feasibility-check.md` | 剧本进分镜前强制三关校验：逻辑性（因果断层/人物目标/时间线）、真实性（职业/法律/常识查证留痕）、题材合规（红线/AI生成特限/赛事规则），过不了回改剧本 |
| `references/script-templates.md` | 剧本各步填空模板库：logline/主控思想/Story Spine/节拍表/主角档案/场景表/台词稿/伏笔账本 + 对话打磨清单 + 金句设计，抄走即用 |
| `scripts/script_qc.py` | 剧本定稿后跑质量自检（台词字数密度/and-then 平铺/被动主角/决策点/节拍三区覆盖），配合 script-drafting-sop |
| `references/story-cases.md` | 写短剧剧本前对照爆款实操拆解：疯美人（打脸爽点）/咬清梨（情感锚点）/好一个乖乖女（高密度反转+金句）/情绪曲线法/冲突×2反转公式 |
| `references/case-by-step.md` | SOP 每一步的实操案例映射：纸手机（高概念道具/破亿）/苏太太（爆发点密集/10亿）/婢女（三秒共情）/盛夏芬德拉（生活流反套路）/AI豆豆（公共议题）/牌子（寓言符号）等，卡在哪步查哪步 |
| `references/platform-rules.md` | 首次接入平台/核算积分时 |
| `references/bgm-sound-design.md` | 设计 BGM/环境音时 |
| `references/jianlai-benchmark.md` | 目标"剑来级质量"（水墨国风/意境化打斗/审美上限）时：质量七维拆解+AI可达性+提示词模板 |
| `references/douyin-prompt-cases.md` | 写提示词/学优秀创作者时：抖音爆款提示词案例库（四段式/微表情/分段风格词/反堆砌） |
| `references/french-nouvelle-vague-style.md` | 出"70年代法国老电影质感"画面时：新浪潮视觉语法7特征（手持/自然光/跳切/现场声/实景）+六槽位填空模板+文生/图生完整可粘贴示例+灵魂词与禁词+seedance实测规则（来源@旭子锅锅+seeddance2+seedancetool+Sider+新浪潮调研） |
| `references/film-opening-hooks-open-source.md` | 剧本转视频时查"怎么开场/怎么自然切入/怎么不弃片"：5种电影开场类型（事中切入/冷开场/建立镜头/主角首见动作/静物细节）+钩子4属性+AI视频前3秒实操（首帧三要素/3个信息点/3秒分段法）+连续引擎（晚进早出/每场留问/钩子桥）+《她的名字》落地表（来源Raindance+Creative Screenwriting+CloudKOL+Hailuo+PixMind） |
| `references/character-relationship-design-open-source.md` | 设计/审计人物关系与出镜决策时：四铁律（主角太阳/配角行星/先功能后出场/关系=测试）+副线纠合规则+出镜vs画外决策（缺席即装置/观众10秒定位门禁/同脸跨年代锁锚）+人物七要素+《她的名字》全角色审计表（来源麦基《故事》+北电编剧研究院+ScriptForge+TCK） |
| `references/name-introduction-natural-open-source.md` | 写/查对白怎么自然引出人名时：核心原则=名字揭示必须有动机（叫名四动机）+No Film School 7法+名字呈现通道分级（视觉物件>呼叫>转述>自我介绍>互叫禁用）+隐藏名字规矩（须剧情理由）+《她的名字》全名字引入审计（来源No Film School+Anne Bellows+ScreenCraft+Stage32） |
| `references/motivation-system-open-source.md` | 检查人物动机/道具动机时：want-need双层结构+动机四类型表+动机合理性四问（为什么现在/与性格一致/赌注可指认/有后果）+契诃夫之枪铁律（被强调必兑现/两次枪响/plant-payoff）+《她的名字》全动机审计（来源Neil Chase+Fiveable+StudyRaid+契诃夫之枪体系+文艺报） |
| `references/action-tone-prompt-open-source.md` | 写人物动作/定全片基调节奏/动作转提示词时：Blocking六动作语言（approach/retreat/block/separate/circle/turn away）+四模式速用+手势微表情设计+动作→AI提示词六规则（概括性优先/肢体量化/低缓优先/微表情写实/幅度速率效果/动作镜头分离）+Tone Bible+张弛调制+节奏曲线+《她的名字》动作审计表（来源Robert Bruton+Orbital Ranch+火山方舟Seedance官方+VideoLens+Memons+BlockReel） |
| `references/voice-speed-plan.md` | 配音语速规划时：中文语速基准（正常4/旁白3.5-4/慢速3字每秒）+全片台词密度分级（纯对白≥3/剧情1.5-2.5/安静叙事≤1.5字每秒）+AI TTS参数（豆包speech_rate -50~100/剪映0.5-2x）+角色语速分配表+留白规则+逐场密度验算表（来源百科/数字科技馆/火山引擎/剪映） |
| `references/audience-attack-checklist.md` | 剧本/分镜交付前必跑：刁钻观众多角色攻击自检——51 类 20 维身份库（算术控/时间线控/体制过来人/物理控/道具控/状态跳变/信息边界/二刷控/快进静音控/目标人群/听觉控/表演控/摄影转场控/资方答辩控/平台审核/伦理/成本/错别字/版式/职业动作/路人噪音/光影方向/季节天气/记忆力/一句话复述/转述推荐/声音层次/电话戏/伏笔强度/悬念持续/留白/金句/情绪心电图/标题海报/传播切片/参赛规格/片长密度等）+三步执行法（选未用身份→逐问攻击→依据判定）+13 级攻击优先级+轮换规则（每轮换维度、三轮后启用深层身份）+实战三轮真 bug 案例（年龄矛盾/周三冲突/夜灯/顶替机制），与 script-attack-checklist（编剧结构）互补 |
| `references/aigc-bloggers.md` | 系统性学优秀 AIGC 博主时：博主清单+7 条蒸馏金规律（母提示词/尾帧锚/留接动作/时间分配） |
| `references/cross-domain-tips.md` | 跨领域技巧蒸馏：音频（Codex配乐/音效分层）、剪辑（6种首尾帧转场/抠像卡点）、剧本（爆款公式/黄金开篇/人设碰撞）、国外工作流（静帧先行/平台规格/发布当发版） |
| `references/creator-deepdive.md` | 深拆博主视频全内容时：团子30版参数（6段式提示词结构）/小羊羊全流程4步/反推式工作流/深拆方法论模板 |
| `references/ksrsang-canvas-piracy-workflow.md` | 学"怎么偷学/拉片/做资产"时：Ksr桑画布偷学五步（公开画布→创作工程→抄提示词工作流→记录文档→小样片验证）+拉片四问（第一眼/前景背景/节奏情绪/构图压缩）+手尾帧同机位跳帧坑+制作8步（剧本通读→风格底图→角色概念融合→四宫格→场景图一张定所有站位→复述喂图→剧本再优化→逐镜生成）+参考图无杂质铁律（背景禁墙面地毯/头发细节干扰）+《她的名字》落地清单 |
| `references/canvas-open-source-map.md` | 找画布平台/开源提示词库时：画布平台地图（即梦画布CLI/LibTV/蝉镜/PixMind/Pixomi/RunningHub/Astorie）+awesome-seedance-2-prompt（按镜头意图分目录+单镜语法subject/action/camera/lens/lighting/atmosphere+三即用配方+continuation续写种子+graveyard失败库）+seedance-prompt-skill（10大能力+时间戳分镜+@多模态引用+分段拼接+台词音效分写）+平台硬约束（真人脸参考图屏蔽/12文件上限/4-15s）+《她的名字》落地清单 |
| `references/director-skill-ecosystem.md` | 写分镜/每镜提示词前的"导演层"参考：Flovaai三套Skill分工（Lira分镜图/ACTING情绪→动作/CINEDANCE运镜时间轴+声音）+SCAI电影感镜头轻量版哲学（完整流程9小时太重→单帧用轻量skill，光方向/硬光位置/暗面不糊黑）+电影导演Skill（先梳理故事→情绪→段作用，20套风格）+LibTV Skill Hub 100+（选Skill=先把成熟方法带进项目）+豆包装Skill方法 |
| `references/agent-skill-design-pattern.md` | 设计"生成型Skill"给下一个Agent用时：流程+知识库+自检三合一（十步工作流每步约束+69条默认值+render/diff复核）+五原则（状态驱动不跳序/硬停不凭记忆/美术方向优先模板/自检内置/默认值写死）+开源实例（img2threejs状态机/agentcad measure/cad-khana四命令）+对视频Skill迁移建议（分镜加状态标记列/每镜提示词前硬停/Brand Bible固化） |
| `references/model-prompt-language.md` | 选平台/写提示词前：平台口味表（Sora/可灵/即梦/Runway…）+ Seedance五要素导演法 + 负向词库 + 驯化模型迭代法 |
| `references/cinematic-look-prompt.md` | 写提示词要"电影质感"时：六要素（取景/焦段/布光/机位/动作/调色）+ 布光四件套 + 调色四杠杆 + 胶片参考词表 + 焦段情绪表 + 万能四cue公式 + 减法原则 + 负面词 + 《她的名字》全片视觉基准（Kodak Vision3 500T/纪实剧/三源布光/16:9） |
| `references/topic-market-map.md` | 参赛选题前查赛道饱和度：11类题材市场证据/饱和度/参赛契合/积分友好度 + 评委口味 + 选题决策规则 |
| `references/masterclass-digest.md` | 四方向批量深挖合集：获奖复盘（鲸鱼之歌3000元/无五官设计/编号绑定/戛纳16181次生成）、团队方法论（影视飓风AI参考片/Domer多模型角色/复盘资产回流/@一次原则）、风格美学（国风鎏金五因子/水墨单暖色/视觉语言迭代）、单技能（主观俯拍/大师运镜20式/反向表演/情绪三层叠加） |
| `scripts/budget.py` | 每版时长/分辨率变更后核积分 |
| `scripts/dialog_density.py` | 每次台词改动后 |
| `scripts/script_full_check.py` | **每次剧本定稿前必跑**：一键自动流=A基础体检7项+B深度规则6项（称呼契约/金句/词频/时代词/账本完整性/最长台词）+B2数值交叉+C人工语义必答7条；回归样本 tests/fail-samples 验证脚本能抓假 PASS |
| `scripts/learning_deposit_sop.py` | **所有学习步骤的统一沉淀门禁（学习→加载→运行自动流）**：每次学到新知识后跑它，把学习沉淀成 references/ 技能文件并接入 knowledge-route-table 与 SKILL.md 索引；支持 `--new "主题"` 生成新技能骨架，然后校验 ①文件存在非空 ②已接入路由表 ③已列索引 ④含可加载章节 |
| `scripts/validate_storyboard.py` | 分镜表定稿后跑客观校验（配合 storyboard-contract） |
| `references/3d-skill-ecosystem.md` | 需要 3D 建模/渲染/游戏资产/网页 3D 技能时：Snyk 权威 8 强（Game Developer 403★/Three.js R3F 70 规则/3D Modeling AAA 老兵/Shader/CAD）+DavinciDreams 全管线团队插件（9 skills+7 agents）+blender-web-3d/mesh-forge/Blender MCP（13.7k★）+开源模型（TRELLIS-2/3DTopia/Step1X）+ToxicSkills 安全警示（36% skill 含注入） |
| `references/video-skill-map.md` | 找"还有哪些厉害的视频 skill"/选底座/选工具时：180 个开源视频 skill 十大分类（剪辑35/讲解32/框架29/产品26…107 个<50 星）+渲染底座选型（排版为主→HTML HyperFrames/镜头为主→Remotion）+Pexo 九强排名（自动选模型/角色一致性/数字人/剪辑）+组合用法+安全习惯 |
| `references/scene-chaining.md` | AI 视频 >15s 分段生成/跨镜拼接/每镜转场提示词时：五大衔接法（动作重叠/接点留半/故事板分组 15s 一组/首尾帧截图/延展+剪掉）+Last-Frame Seed（末帧=下段首帧锚，跨边界动作的 #1 fix）+衔接提示词设计（每镜提示词结尾写转场：运动/打击/遮挡/相似体）+漂移定向修复表（ClawHub：身份减锚/布局强场景包/道具 allowed_changes/构图 same framing）+帧率参数+《她的名字》SEG 边界落地清单 |
| `references/editing-craft-open-source.md` | 成片剪辑/合成或分镜预埋剪辑点时：剪辑理论母语（蒙太奇第一定律/库里肖夫/理性蒙太奇/节奏剪辑含反直觉/微妙跳切控表演/蒙太奇类型库）+开源剪辑 skill（video-use/Diffusion Studio 免费/OpenMontage）+AI 剪辑实操（卡点分层 build密剪 drop拉宽/无缝转场四检查/B-roll 铁律仅6%在用+cutaway 拼接/音频桥）+对《她的名字》落地清单 |
| `references/editing-intent-and-genre.md` | 拿到剧本/分镜"怎么剪/什么风格剪/思路从哪来"时：类型×剪辑风格映射表（动作2-3s镜/喜剧反应镜头cutaway/剧情长镜呼吸/悬疑声音设计/蒙太奇类型功能）+剪辑构思方法论（读剧本三阶段/黄金镜头法/三度创作先线性后调/素材启动论/打破线性叙事/首镜定调）+AI短剧实体案例（蜀锦一梦精简原则/5000集黄金结构情绪微循环/三秒一切/H3钩子节拍/霍去病/九尾凤凰）+剪辑构思SOP+对《她的名字》落地清单 |
| `references/genre-identification.md` | 拿到剧本/选题先精确识别类型时：IMDb 类型公式（故事+情节+角色+设定=类型）+VFS 正确层级（主→次→亚→基调）+Save the Cat 十大类型识别器（鬼怪屋三要素/金羊毛路队伍奖品/伙伴情谊/制度之下等+例子）+三步识别法（定主引擎→落主类型→中点高潮仲裁）+logline 反推验证+《她的名字》应用（主=制度之下/次=金羊毛）+交付门禁（必须输出主次类型+类型承诺） |
| `references/doubao-jianying-auto-edit.md` | 用豆包连剪映做自动剪辑时：三步工作流（工作任务装 Jianying Editor skill→选 skill 输剪辑要求→剪映草稿箱取可编辑工程）+能力清单（粗剪/音乐卡点/字幕匹配/转场/配音配乐）+实测边界（还有环节需人工确认）+企业级 API 架构认知+限制（依赖 Windows 本地剪映，云端/手机不可用，需回 Windows 端）+与 video-edit-pipeline/beat-sync 互补分工 |
| `references/prompt-craft-mindset.md` | 写每镜提示词对齐顶级工匠时：工匠大脑（Stop Guessing Start Directing/公式=导演沟通镜头/token前10-15词权重最大/五模块缺则瞎猜/八控制层/具体词>形容词）+工匠用法（七段骨架含6节拍动作链/最后15-20字留给光线镜头/Style Blocks风格块库/五步蓝图/相机位置=权力语言/景别决定情绪基调/务实五律含prompt版本库）+整合（token权重纪律/环境音写全/风格块逐字复用/版本留档）+《她的名字》每镜模板落地 |
| `references/prompt-optimizer-models.md` | 打磨提示词借鉴"提示词优化模型"自动化思维时：代表框架逻辑（DSPy Program-not-prompt/TextGrad文本梯度/APE变异选择/OPRO黑盒优化/PromptWizard自进化/RPT诊断报告/FLARE错误感知+收敛对比）+七条共通思维（迭代闭环/失败模式诊断优先/一次改1-2变量/自批评/结构化优于长文/搜索空间/7要素评分清单）+可迁移（提示词质量评分卡/返工诊断闭环/版本留档/多候选） |
| `references/scene-3d-pano-modeling-open-source.md` | 做 M3 级场景（档案室等"先全景3D建模→再截多机位"写实3D场景）时：全景建模流程（Equirectangular 2:1/Cycles/相机(90,0,-90)/人眼高度）+Blender渲染空白6因排查+视频提示词公式（主体+运动+场景+镜头+光影+氛围+风格，≤512字符）+SuperMap提示词合集占位符复用+MJvsSD工具选型+灰模→Depth ControlNet（锁体量/透视）+SD英文prompt纪律 |
| `references/impact-frame-white-flash-technique.md` | 写打戏/蓄力/发射类镜头冲击感时：冲击帧=发射前短促爆发（非持续白屏闪光）——能量中心锚点+亮暗交替配色序列（黑红→红白→浅底墨绿反向→黑红）+25帧每张1帧≈0.4s+接回能量球再发射+爆发音效卡冲击帧+防"一直闪"负面词（来源AI急救局/复刻淞希） |
| `references/ai-canvas-mcp-workflow.md` | 把画布当 AI 短剧工作流编排层时：AI Canvas+MCP 智能体自动组织剧本/参考素材/生成节点，人只调效果；落地豆包 Canvas（+create 新版本/+update 仅brief-path/血缘source_ids/画布=权威+本地镜像+索引三联动）（来源Tenney） |
