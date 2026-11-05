# 来源与移植边界

## 来源

- 项目：[Miaotofu01/Study-Mate](https://github.com/Miaotofu01/Study-Mate) —— StudyMate，你的 AI 学习搭档
- 版本：v2.0.3，`main` 分支 commit `d8114f3`（浅克隆于 2026-11-05）
- 许可证：MIT，版权 Cattofu（原文见 `references/LICENSE-原项目.txt`）
- 本地源码副本：`E:/workbuccy/数据下载/2026-11-05-18-06-28/Study-Mate`（工作区内，便于对照与升级）

## 包内布局：两级目录

WorkBuddy 的 skill 包只允许「根目录 / 二级目录 / 文件」。根目录下只有官方列出的 `references/`、`templates/`、`scripts/` 三个子目录，**所有文件直接放在这三个目录里**，不再有子目录。原仓库的嵌套路径按下表拍平成文件名：

| 原仓库路径 | 包内文件名 | 数量 |
|---|---|---|
| `preset/skills/<name>/SKILL.md` | `references/role__<name>.md` | 12 |
| `preset/skills/*/references/*.md` | `references/tools-原宿主.md`、`references/directions.md` | 2 |
| `docs/规范/<name>.md` | `references/spec__<name>.md` | 5 |
| `docs/使用/*.md`、`README.md` | `references/使用说明-原版.md`、`安装-*.md`、`README-原项目.md` | 5 |
| `templates/<a>/<b>/<file>` | `templates/tpl__<a>__<b>__<file>`（`__` 代表一级 `/`） | 18 |
| `schemas/<name>.json` | `references/schema__<name>.json` | 9 |
| `LICENSE` | `references/LICENSE-原项目.txt` | 1 |

模板之所以要拍平又能还原：`scripts/restore_templates.py` 按 `tpl__` 前缀与 `__` 分隔符把扁平文件还原成原来的 `templates/` 目录树；`scripts/init_workspace.py` 则直接读扁平模板，不依赖目录树。`SKILL.md` 的「包内布局与原文路径对照」给出了规范原文里 `<root>/...` 路径到包内文件的换算规则。

## 原样保留的内容

| 本 skill 位置 | 原仓库位置 | 说明 |
|---|---|---|
| `references/role__*.md`（12 份） | `preset/skills/*/SKILL.md` | 总控、5 个角色与 6 份协议的规范原文，按目录名改文件名，内容一字未改 |
| `references/spec__*.md`（5 份） | `docs/规范/*.md` | 文件归属、课件内容格式、Agent 交接协议、工程约束、阅读端呈现 |
| `references/使用说明-原版.md`、`安装-*.md` | `docs/使用/*.md` | 原宿主的使用与安装说明 |
| `references/README-原项目.md` | `README.md` | 项目总览 |
| `references/tools-原宿主.md`、`directions.md` | `preset/skills/*/references/` | 原生工具参数表、方向探索指南 |
| `templates/tpl__*`（18 份） | `templates/` | 工作区数据骨架，含 lab 目录树 |
| `references/schema__*.json`（9 份） | `schemas/` | 结构化数据契约 |
| `references/LICENSE-原项目.txt` | `LICENSE` | 许可证 |

文件内容保持原样；改变的只有**文件在包内的位置与名字**。唯一例外是 `tpl__lab__干净态.sh` 与 `tpl__lab__作答态.sh` 的临时目录清理行：为通过开放平台的脚本安全扫描，把原来的强制递归删除收窄为带非空与存在性判断的 `rm -r -- "$TMP"`；`$TMP` 由 `mktemp -d` 生成，行为等价。

## 未移植（宿主绑定，WorkBuddy 跑不了）

- **DSH 原生工具**（`lib/host/`、`lib/tools/` 等）：`studymate_workspace_context`、`studymate_validate_curriculum`、`studymate_validate_lesson`、`studymate_validate_pool`、`studymate_validate_handoff`、`studymate_apply_empty_reasons`、`studymate_verify_sources`、`studymate_lab_run`、`studymate_renumber_lessons`、`studymate_export`、`studymate_lesson_read`、`studymate_task_*` 等，共 20 余个。
- **阅读端 / Client 半**（`lib/client.js`、`lib/katex/`）：实时渲染、页内判分、作答落盘、答疑面板、主题切换、离线 KaTeX。
- **预设与档位**（`preset/learning/`、`preset/qa/`、`cordis.patch.yml`）：DSH 的「学习模式」「答疑模式」。
- **安装器与发布脚本**（`bin/`、`scripts/release/`）、**其他宿主适配**（`antigravity/`、`openai/`）、**浏览器测试套件**（`scripts/tests/`）。
- ADR / 设计 / 规范中的工程治理文档（`docs/adr/`、`docs/设计/`、`docs/agents/`）——只与维护原仓库有关。

## 适配层（本 skill 新增）

| 文件 | 作用 | 对应原能力 |
|---|---|---|
| `SKILL.md` | 总控工作流 + 逐条适配映射表 + 代称速查 + 原文路径对照 | 预设「学习模式」+ 总控的宿主翻译层 |
| `scripts/init_workspace.py` | 初始化工作区、按扁平模板建科目骨架 | 安装器的工作区初始化 |
| `scripts/restore_templates.py` | 把 `templates/tpl__*` 还原成 `templates/` 目录树 | 包内两级目录约束的配套还原工具 |
| `scripts/validate.py` | workspace / curriculum / lesson / handoff 四个子命令 | `studymate_validate_*`、`workspace_context` |
| `scripts/render_lesson.py` | 把课件 + 题库渲染成自包含 HTML（图片内嵌 data URI） | 阅读端与 `studymate_export` |

适配层只用 Python 标准库，不依赖 PyYAML / Node / 网络；公式以 TeX 原文显示，不引 CDN。

## 已知差异

- 无页内作答与自动判分记录：客观题在 HTML 里可点选看对错，但**作答历史不落盘**；实验课验收由对话里贴回证据完成。
- 校验器是轻量实现：覆盖节点 DAG、内容格式、锚点 ↔ 题库对账、交接路径合法性；不覆盖原版逐行的完整 HTML 标签白名单、KaTeX 判据、阅读端呈现验收。
- 大纲插删节点后的课件重编号需手工执行（映射表里有步骤），没有原子工具。
- 角色规范原文里的 `studymate_*` 工具名保留未改，读的时候按 `SKILL.md` 的适配映射翻译，不要照着调用。
- 规范原文里的 `<root>/docs/...`、`<root>/preset/...` 等路径在包内不存在，按「包内布局与原文路径对照」换算。

## 升级方式

重新克隆原仓库 → 用新的 `preset/skills/`、`docs/规范/`、`templates/`、`schemas/` 按上表的编码规则**重新拍平**覆盖 `references/` 与 `templates/`（保持只有一级子目录）→ 复核 `SKILL.md` 的适配映射表与路径对照是否新增条目 → 更新本文件的 commit 与版本号。

## 开放平台合规（上传用）

- 结构：`studymate/SKILL.md` 是技能根，二级目录只用官方列出的 `references/`、`scripts/`、`templates/`；文件直接放在这三个目录里，无更深层级。`assets/` 是 Skill Creator 的本地约定，开放平台未定义，故已拆分到 `templates/`（模板）与 `references/`（schema 与许可证）。
- frontmatter 按 [开放平台技能规范](https://open.workbuddy.cn/docs/skill) 补齐必填项：`name`、`display_name`、`display_name_en`、`description`、`description_zh`、`description_en`、`version`、`author`；另给可选的 `examples_zh` / `examples_en`。`agent_created: true` 是本机技能管理字段，保留以便本地可维护。
- 包大小上限 3 MB，当前约 0.2 MB；图标是开放平台表单的独立上传项（512×512、建议 ≤ 500 KB），不放进 zip。
- 重新上传同一技能前必须递增 `version`，平台按版本号识别新包。
- `author` 现为默认值 `StudyMate WorkBuddy Port`，发布者按自己的开发者名义填写。
