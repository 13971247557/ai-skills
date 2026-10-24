---
name: skill-distiller
description: 技能组的外部能力孵化器：把网上开源的 AI 视频/剪辑/创意 skill 调研、拆解，孵化成本组能直接跑的可执行资产（脚本/命令/工作流分支/模板），而不是收藏文档。闭环五问：何时触发→如何调研→如何拆解→如何判定可执行资产还是纯资料（拒绝纯资料）→如何复配嵌入本组工作流并过门禁。产出标准是"学完即会用"：每个孵化出的脚本都要在本机真跑一遍，跑通记录输出，跑不通如实说明缺 Key/缺依赖/缺 GPU 的原因。配套 references/distill-methodology.md（调研方法论）、references/distill-template.md（单技能拆解模板）、references/absorb-sop.md（复配 SOP）、scripts/distill_check.py（蒸馏后快速门禁）。触发：蒸馏开源 skill、孵化新能力、学习网上的 AI 视频 skill 并吸收、把外部技能装进本组、install-external-open-source-skills。
license: MIT
metadata:
  version: 1.1.0
  role: 技能组自我孵化能力
---

# skill-distiller

> 本技能不是"资料收藏夹"。它的产物是**本组工作流里能直接跑的新能力**。凡是拆出来只能当参考文章、不能变成命令或脚本或工作流分支的，判定为"纯资料"，拒绝沉淀进技能目录，最多在 `skill-evolution/references/open-source-skill-library.md` 记一行思路。

## 一、闭环五问（每次孵化都走一遍）

### Q1 何时触发

- 用户说"学一下网上某个开源 AI 视频/剪辑 skill，把它的优点吸收进来"。
- 本组某个环节是空白（例：缺自动场景切分、缺静音粗剪、缺去水印方案），去 GitHub 找现成实现补位。
- 装完一个外部仓库后，要把它从"原样 clone"复配成本组规范、可独立过门禁的技能。
- 不触发：用户只是问概念、只要资料综述、或该能力本组已有（先查 `open-source-skill-library.md` 避重）。

### Q2 如何调研

用真实网络检索（general_search / web_fetch / GitHub API / git clone --depth 1），不许凭印象编仓库。检索词与评估维度见 `references/distill-methodology.md`。每个候选必须实测：web_fetch 打开仓库页成功，或 git clone 成功。产出候选清单：仓库 URL + star + 许可证 + 一句话形态 + 是否选中。

### Q3 如何拆解

clone 下来后精读关键文件（SKILL.md / README / scripts/，用 Grep/head 省 token），按 `references/distill-template.md` 一页一个地拆：触发方式、工作流、脚本、可复用点、不适配点。**重点找"可移植的命令和脚本"，而不是读起来很爽的长文。**

### Q4 如何判定"可执行资产 vs 纯资料"（核心闸门）

拆出的东西先过这张判定表，**三条全过才算可执行资产，否则是纯资料，拒绝装进技能目录**：

| 判定项 | 可执行资产（收） | 纯资料（拒） |
|---|---|---|
| 有无可移植命令/脚本 | 有可拷贝的 .py/.sh 或可复用的 ffmpeg 命令链 | 只有散文式流程描述、没有任何命令 |
| 能否脱离原仓库跑 | 拷进本组后，删掉原仓库目录也能跑（依赖可装） | 强耦合原仓库目录结构/私有服务/私有 Key 才能跑 |
| 依赖补不补得上 | 本机/组内能装依赖（ffmpeg/python 包），或缺的 Key 有明确获取路径并如实标注 | 需要 GPU/付费服务且无替代，或根本不知道怎么跑 |

- 半可执行（如脚本依赖某个 API Key 本机没有）：**脚本照收，但在 SKILL.md 里如实标注"暂不可跑，缺 XX Key"**，其余不依赖该 Key 的部分先真跑验证。
- 纯资料：只在 `open-source-skill-library.md` 记一行"思路资产 + 未来怎么变成可执行"，不建技能目录。

### Q5 如何复配嵌入工作流 + 过门禁

按 `references/absorb-sop.md`：拷脚本进新技能目录 → 适配 frontmatter（白名单字段、中文 description、无尖括号、≤1024）→ 去残留 → **本机真跑一遍** → 过门禁 → 登记。嵌入点要写清：这个新能力挂在本组哪条链上。

## 二、孵化的硬性纪律

1. **学完即会用，不是学完即收藏**：每个孵化脚本必须本机真跑一次，跑通贴输出、跑不通写清缺什么。
2. **只改目标新目录**：不动本组已有技能的既有文件（除非用户要求）。
3. **URL 必须实测**：写进文档的来源链接都要打开过，无效占位一律删。
4. **不重复造轮子**：先查 `open-source-skill-library.md` 和本组已有技能，重复的不装。
5. **诚实标注**：缺 Key/缺依赖/未验证的部分明写"暂不可跑/待验证"，不许写成已实现。
6. **装完必过门禁**：quick_validate + self_check 全量无 FAIL（并发任务造成的他目录 FAIL 除外）才算完，然后写 UPDATE_LOG。

## 三、与本组既有门禁的衔接

- frontmatter 白名单、description 规范、断链/残留/参数核对：走 `skill-evolution/scripts/self_check.py`（全量自检）。
- 蒸馏侧快速预检：本技能 `scripts/distill_check.py <技能目录>`（frontmatter 白名单 + description 长度 + 残留扫描）。
- 质量门禁七项标准：见 `skill-evolution/references/quality-gate.md`。
- 外部技能登记：装完在 `skill-evolution/references/open-source-skill-library.md` 追加条目，标注"✅ 已安装"或"待验证"。

## 四、目录

```
skill-distiller/
├── SKILL.md
├── references/distill-methodology.md   # 调研方法论（检索词/评估维度/防假仓库）
├── references/distill-template.md      # 单外部技能一页拆解模板
├── references/absorb-sop.md            # 复配 SOP（安装→适配→去残留→真跑→门禁→登记）
└── scripts/distill_check.py            # 蒸馏后快速门禁（frontmatter/残留/description）
```
