---
name: skill-evolution
description: "技能组自进化闭环（常设机制）：统一自检清单与执行步骤（quick_validate校验/断链/残留/命令可用性核对/交叉引用/URL实测）、持续学习与深度调研沉淀流程（四要素+四视角+大能来源分级）、版本化更新记录、bug定位-修复-复检闭环、质量门禁。触发时机：技能文件每次改动后、成片每次交付前、新调研成果沉淀时、发现技能异常时。执行入口：scripts/self_check.py。"
---

# 技能组自进化闭环（Skill Evolution）

## 0. 一句话用途 + 何时读

本技能是技能组的**常设维护机制**，不是一次性动作：它规定「何时自检、怎么学习、怎么更新、怎么修 bug、什么算过关」，让技能组在每次出片、每次调研、每次改动后自动打磨升级。

- **技能文件被改动后** → 读 self-check.md，跑 self_check.py
- **新调研成果要沉淀时** → 读 learning-loop.md + update-log.md + quality-gate.md
- **发现技能/脚本异常时** → 读 bug-fix-loop.md
- **成片交付前** → 只跑自检（self_check.py + quick_validate），不读全篇

## 1. 触发时机（常设规则）

| 时机 | 动作 | 依据 |
|---|---|---|
| 每次技能改动后（新增/修改/删除任何 SKILL.md、references、scripts） | 跑 self_check.py 全量自检 + 写 UPDATE_LOG 条目 | self-check.md / update-log.md |
| 每次成片交付前 | 跑 self_check.py（结构/残留/断链/命令核对） | self-check.md |
| 每次新调研/新学习成果落地前 | 走「四要素 + 四视角 + 大能分级」沉淀流程，过质量门禁 | learning-loop.md / quality-gate.md |
| 自检出现 FAIL 或运行中发现异常 | 「定位 → 修复 → 复检」闭环，修复经验写进排障库 | bug-fix-loop.md |
| 每月巡检 / 平台额度变化 / 新模型发布 | 触发一轮轻量学习与更新 | learning-loop.md |
| 两个任务同时改同一技能目录时 | 先确认无并发写，串行化，避免文件冲突 | bug-fix-loop.md（教训） |

## 2. 五大环节闭环

```
自检发现短板/用户提新需求
      │
      ▼
① 自动检查（self_check.py + quick_validate）── 发现问题 → ④
      │ 通过
      ▼
② 自动学习（真实网络检索 → 精读 → URL 实测 → 四要素结构化 → 大能分级）
      │
      ▼
③ 更新升级（沉淀进 SKILL.md/references + UPDATE_LOG 版本记录）
      │
      ▼
⑤ 质量门禁（quick_validate / 无残留 / URL 实测 / 命令核对 全 PASS）
      │
      ▼
交付 & 下次出片继续用
④ 修 bug 闭环（定位 → 修复 → 复检 → 经验入排障库）——任何环节发现问题都进入这里
```

## 3. 快速上手（自检命令）

```bash
python3 scripts/self_check.py
# 可选参数：
#   --repo <技能名>           只自检单个技能
#   --check-urls             额外做文档 URL 可访问性抽查（网络请求，较慢）
#   --skills-dir <路径>      指定技能组根目录（默认路径不存在时自动探测脚本位置向上定位）
#   --quick-validate <路径>  指定 quick_validate.py（默认自动用本技能 scripts/quick_validate.py）
# 退出码 0 = 全部 PASS；1 = 存在 FAIL
```

> Windows 上直接 `python scripts\self_check.py` 即可，默认路径探测会自动找到本机技能组根目录和本技能的 quick_validate.py。

## 4. 环节详解

### 4.1 自动检查（自检 bug / 质量）→ `references/self-check.md`
统一自检清单：① quick_validate 全量校验 ② 结构检查（frontmatter/目录） ③ 断链检查 ④ 残留检查（TODO/占位/example.com） ⑤ 命令可用性核对（文档 `--param` vs 脚本 argparse） ⑥ 交叉引用完整性 ⑦ URL 实测抽查。何时自检、结果如何分级处理，见该文件。

### 4.1.5 执行记忆循环（每环节记忆+自学习）→ `references/execution-memory.md`
**用户硬要求（2026-10）**：每个环节（素材/出片/配音/字幕/合成/质检）执行后自动记录经验——成功记「已验证经验」下次原样复用，失败/返工记「教训经验」附修正方法；开始新任务前先查记忆（lessons.md + search_memory），命中即应用；准确性验证通过才入库。详见 `execution-memory.md`。

### 4.2 自动学习（持续调研）→ `references/learning-loop.md`
持续调研优秀作品/产品/方法论：真实网络检索 → web_fetch 精读原文全文 → URL 实测可访问 → 按「原理 → 适用场景 → 实战落地 → 反例/易错点」四要素 + 技术/创作者/产品设计/真实案例四视角结构化 → 大能来源分级表。调研成果必须沉淀，不许一次用完就丢。

### 4.3 及时更新升级（版本化记录）→ `references/update-log.md`
所有技能变更写入技能组根目录 `UPDATE_LOG.md`（append only）：日期 / 技能 / 变更内容 / 原因与来源 / 校验结果。版本化记录让每次升级可回溯、可回滚。

### 4.4 修 bug 闭环 → `references/bug-fix-loop.md`
自检发现问题按「定位 → 修复 → 复检」处理，禁止跳过；常见 bug 与修复经验沉淀进排障库（含「并发写同一技能目录」教训）。修复后必须再过门禁。

### 4.5 打磨质量门禁 → `references/quality-gate.md`
所有更新必须过：quick_validate、self_check 全 PASS、URL 实测可访问、无 TODO 残留、无无效占位、命令与现有脚本（build_video.py 等）参数核对一致。未过门禁禁止沉淀。

## 5. 本技能自身的维护

- 本技能同样接受自检：改完本技能的任何文件，跑一次 self_check.py。
- 本技能文档里的命令/路径若有变化，同步更新；`UPDATE_LOG.md` 记录本技能自身变更。

## Resources

- `scripts/self_check.py` —— 自检执行器（可独立运行，输出 PASS/WARN/FAIL 汇总；默认路径自动探测）
- `scripts/quick_validate.py` —— 通用技能校验器（validate_skill 接口：SKILL.md 存在 + frontmatter 合规 + 基础结构检查）
- `references/self-check.md` —— 自检清单与执行步骤（何时自检）
- `references/execution-memory.md` —— 执行记忆循环（每环节记录经验/自学习/准确性验证入库）
- `references/learning-loop.md` —— 学习流程（调研 → 四要素 → 大能分级 → 沉淀 → **第7节调研盲区防御协议**）
- `references/open-source-skill-library.md` —— 开源优秀 Skill 学习库（收录 ComfyUI 工作流体系/Wan2.2-S2V/动作转移等，含可复用点与沉淀位置）
- `references/update-log.md` —— 版本化记录规范（UPDATE_LOG.md 格式）
- `references/bug-fix-loop.md` —— 修 bug 闭环 + 技能维护域排障库
- `references/quality-gate.md` —— 质量门禁清单
- 技能组根目录 `UPDATE_LOG.md` —— 集中变更日志（append only）
