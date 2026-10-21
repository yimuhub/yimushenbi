# 一幕 · 神笔（ShenBi NovelCraft）· AI 小说工作台

一个**聊天即开写**的 AI 网文创作工作台：丢一个主题 → 自动规划 → 逐章创作 → 三层质检 → 定向重写 → 放行成书。内置**「一幕·神笔 · ShenBi NovelCraft」原创创作系统**（全平台通配网文），支持**多模型供应商**（自定义绑定、按任务选模型、模型规则引擎）、**多 Agent 并发宇宙**、**一键完稿**、**写作骨架**，产物是**对标网文平台排行前列**、带**百分制达标门禁**的可下载成书。

> **一句话**：把「写一整本网文 + 保证质量 + 能续写/改写」交给 Agent，你只管在对话里说需求、点按钮。
> 版本：`VERSION`（单一来源，当前 **v0.9.2**）· 代码 **MIT** · 技能文本**自有版权**（见 `LICENSE` / `NOTICE.md`）。

---

## 目录
- [能力总览](#能力总览)
- [界面：三栏工作台](#界面三栏工作台)
- [核心功能详解](#核心功能详解)
- [创作流水线](#创作流水线)
- [快速开始](#快速开始)
- [模型供应商与密钥](#模型供应商与密钥)
- [内置原创技能](#内置原创技能)
- [API 一览](#api-一览)
- [目录结构](#目录结构)
- [配置要点](#配置要点)
- [许可与版权](#许可与版权)

---

## 能力总览

| 能力 | 说明 | 端点 |
|---|---|---|
| **聊天开写** | 在万能对话里丢主题/指令，自动判断开书或续写/改写/即时修正 | `POST /api/chat`、`/api/cmd`、`/api/continue` |
| **开书向导** | 题材/视角/基调/章数/卖点/设定/书名/供应商模型 全可选 | `GET /api/onboarding`、`GET /api/styles` |
| **书名 ABC** | 8 个候选弹选或自定义，不带名开书停在 `awaiting_title` | `GET /api/titles`、`POST /api/pick-title` |
| **多模型供应商** | 自定义绑定多家、按任务/按阶段选模型、模型规则引擎 | `/api/providers`、`/api/model_rules`、`/api/model_options` |
| **多 Agent 宇宙** | 每个子岗位单独配模型 + 全局并发上限 + 批量并发多书 | `/api/agents`、`/api/agents-set`、`/api/agents-dispatch` |
| **并行创作** | 单书内 N 个 Agent 分章齐写（先熔炼总架构） | `POST /api/parallel-write`、`/api/parallel-log` |
| **一键完稿** | 小白：熔炼→写剩余→逐章评分→不达标自动重写→放行 全自动 | `POST /api/one-shot` |
| **工作室总览** | 重度：所有书 mini 进度 + 运行态（并行/重写/放行/一键）任务看板 | `GET /api/studio-overview` |
| **总架构熔炼** | 全书单一事实来源 `02b-总架构`，先熔炼再动手（保全局一致） | `GET /api/blueprint` |
| **写作骨架** | `17-写作骨架.md`：写到哪/多少/重点/为何/续写提示，续写改写秒对齐 | `GET/POST /api/skeleton` |
| **三层质检** | 确定性引擎 + 五角色 LLM 审校 + 百分制达标（默认 90） | `POST /api/validate`、`/api/quality`、`/api/review` |
| **定向重写** | 按审查结论 + 主攻方向逐章定向改写，前后分对比，含「保留更好」守护 | `POST /api/rewrite`、`/api/rewrite-plan` |
| **放行门禁** | 成书后自动带要求重写到 ≥ 达标线才标「放行✅」 | `POST /api/release` |
| **风格蒸馏** | 把喜欢的书/作者语料蒸馏成可复用「风格卡片」 | `POST /api/distill`、`/api/styles/{name}` |
| **作品库** | 富元数据、多键排序、筛选、收藏/标签/归档回收站、批量、两本对比 | `GET /api/library`、`/api/compare` |
| **预览/章节** | 点开作品看成书或已写章节 + 逐章评分 | `GET /api/preview`、`/api/chapter` |
| **Token 账本** | 按书/按阶段/全局累计 token 与调用次数（并发安全落盘） | `GET /api/usage` |
| **定时任务** | 每 N 天写一批 N 章，60s 调度器巡检 | `GET/POST /api/schedule` |
| **任务队列** | 查看/暂停/继续 创作任务 | `GET/POST /api/queue`、`/api/pause`、`/api/resume` |
| **下载成书** | 成书打包下载 | `GET /api/download` |

---

## 界面：三栏工作台

网页端（`http://127.0.0.1:46821`）是 **左 · 中 · 右三栏工作台**，三栏宽度可**拖动**、左栏可**折叠**为图标条：

- **左 · 会话 = 书**：每本书一个「会话」，可**同时多本**（默认 2 本并发，超出排队）；每本可**绑定不同模型/供应商**；可**置顶**（📌）；状态点（进行中 / 待选名 / 已成书 / 已放行）；**完结（已成书 + 已放行）自动归档**到作品库（可关）。
  - **工作室总览**（点击自动收回 / 再点展开）：所有书的 mini 进度 + 运行态看板。
  - **已归档**：完结作品的分组管理。
- **中 · 该书对话**：针对当前书的对话 + 快捷操作（续写 / 审稿 / 门禁 / 下载 / 预览 / 并行创作 / 一键完稿，按书的状态动态显示）；「当前书」上下文条；**万能输入框**（默认 1 行，字数多了自动增高、再多才出滚动条）。`?book=<key>` 深链直接定位某本书。
- **右 · 3 个 tab**：
  - **产物**：该书进度 / 内容预览 / 质量报告 / Token 账本。
  - **设置**：该书门禁 · 定时 · 题材×风格技能 + 模型供应商 + 模型规则。
  - **作品库**：全局，按状态 / 题材分组管理。

---

## 核心功能详解

### 1. 多模型供应商 + 按任务选模型
- **自定义绑定**多家供应商（Agnes / DeepSeek / OpenAI / Gemini / 通义千问 / 智谱 GLM / Kimi / **通用自定义**），每家可填 inline key、base_url、模型列表。
- **按任务选模型**：写稿 / 规划 / 审校 各自可指定不同的供应商 + 模型（如写稿用旗舰、审校用 flash，省钱又快）。`defaults.stage_models.{planning,writing,review}`。
- **模型规则引擎**（`bot/model_rules.py`）：按 `base_url` 自动识别厂商 → 套 上下文窗口 / max_tokens / RPM / TPM / 429 退避 / extra_body / 免费限额；**通用**型则用户自由配置。`GET/POST /api/model_rules`。

### 2. 多 Agent 创作宇宙（v0.5+）
- 每个**子岗位**（规划 / 逐章 / 审校 / 改写 / 放行 …）可**单独绑定** provider + model（留空回落默认）。`POST /api/agents-set`。
- **全局并发上限**：多 Agent / 多书同时创作时的在飞上限。`POST /api/agents-maxglobal`。
- **批量并发**：勾选多本（最多 8）一键同时创作，每书一线程、LLM 并发受 `ConcurrencyGuard` 限流排队。`POST /api/agents-dispatch`。

### 3. 并行创作（单书内 N 个 Agent 分章齐写）
`POST /api/parallel-write` 选 worker 数（**2 / 3 / 5 / 自定义 1-8**）：先**熔炼总架构**（保全局一致），再 N 个 Agent **分章齐写**（写完一章立即取下一章）。`GET /api/parallel-log` 看进度。

### 4. 一键完稿（小白） vs 工作室总览（重度）
- **一键完稿** `POST /api/one-shot`：熔炼总架构 → 写剩余章节 → 逐章评分 → **不达标自动重写**（含「保留更好」守护，绝不越改越差）→ 放行门禁。**全程后台跑**，实时看对话区「本书在干什么」活动卡。
- **工作室总览** `GET /api/studio-overview`：所有书 mini 进度 + 运行态（并行 / 重写 / 放行 / 一键），像任务看板，重度多书管理。

### 5. 总架构熔炼 + 写作骨架（v0.6 / v0.9）
- **总架构**（`02b-总架构`）：全书**单一事实来源**——主线脉络 / 卷结构 / 关键节点 / 伏笔兑现计划 / 世界观锚点。动手前**先熔炼再写**，保证全局一致（`GET /api/blueprint`；未生成给规则式即时预览不落盘）。
- **写作骨架**（`17-写作骨架.md`，`GET/POST /api/skeleton`）：每写完/重写/放行自动聚合一份 **200–400 字**的可读说明——**写到哪里了 / 写了多少 / 这次重点是什么 / 为什么这么写 / 续写时应先读什么**。Agent 重写或续写时**先读骨架**即可快速对齐，不用重翻全书。**纯文件聚合，不花 LLM**，`regen=1` 立即重算。

### 6. 三层质检（确定性引擎 + 五角色 + 百分制达标）
1. **L1 确定性引擎**（`bot/continuity.py`，零 LLM）：伏笔闲置 / 超期未兑现 / 主角失踪 / 字数·爽点阈值 / 黄金三章门。
2. **L2 五角色 LLM 审校**：Architect / Scribe / Editor / Guardian / Curator，按**最低维度**给主攻方向。
3. **L3 百分制**：结构 20 / 因果 20 / 人物 15 / 连续性 20 / 语言 15 / 读者体验 10，**达标线默认 90**；未达标 → 有界 autofix 多轮 → 重评。
   产物：`13-修订记录.md`（五角色结论）+ `14-质量报告.md`（L1 + 五角色 + 百分制 + 达标判定）。`POST /api/validate {mode=report|autofix}`。

### 7. 审查后定向重写（含「保留更好」守护）
`POST /api/rewrite-plan` 从 13 / 14 / L1 / 读者试读 提炼**全局主攻方向（最低维度）+ 逐章待修**，写 `16-重写要求书.md`；`POST /api/rewrite` 把要求书**逐条注入**目标章定向重写（后台线程）→ 重评出**前后分对比** → 写 `16-重写记录.md`。带**「保留更好」守护**：改写后若分低于改写前自动回滚，**绝不越改越差**。

### 8. 放行门禁
`POST /api/release`：成书后自动带要求重写到 **≥ 达标线（默认 90）** 才标「放行✅」；未放行的下载标「beta·未放行(分)」（仍可下载）。`GET /api/release-log` 看进度 / 判定。`config.json → defaults.release_gate`（true=成书自动跑，false=仅手动）。

### 9. 风格蒸馏 + 复用
`POST /api/distill`：粘贴喜欢的书 / 作者语料（≥500 字效果佳）→ 分析其**写法**（句式句长 / 高频指纹词 / 节奏爽点 / 对话口吻 / 视角人称 / 情绪基调 / 反 AI 禁用清单）→ 生成**风格卡片** `styles/{name}/style.json`，开书或写稿时挂上「按 XX 的写法」。`GET /api/styles` / `GET /api/styles/{name}` / `DELETE`。跨书学习到的共性写作倾向落 `data/learn_profiles.json`（`GET /api/learn`）。

### 10. 作品库管理（富元数据）
`GET /api/library`：题材 / 时间 / 字数 / 质量分 / 达标 / 状态 / 收藏 / 标签 / 归档；**多键排序**（时间 / 题材 / 完成度 / 质量分 / 字数 / 章节 / 状态，升降序）；**筛选**（状态 / 题材 / 达标 / 搜索）；**统计 + 题材分组**。
`POST /api/library/{key}/favorite|tags|archive|rename`、`DELETE /api/library/{key}`（软删进回收站，`?hard=true` 彻底删）、`POST /api/library/batch`（批量审稿 / 对比 / 收藏 / 归档 / 恢复）、`GET /api/compare?keys=a,b`（两本对比）。

### 11. 开书向导 + 即时修正指令
- **开书向导**：顶部「✍ 开书向导」全可选（题材 / 篇幅 / 风格 / 卖点 / 设定 / 书名 / 供应商模型），`GET /api/onboarding` 出枚举；也能**直接打字**丢需求，自动判断开书。
- **即时修正指令** `POST /api/cmd`：对当前书追加一条「接下来写 / 改要注意…」的指令，**立即生效、不改已写正文**（后续章按新指令写）。

### 12. Token 账本 + 定时任务 + 队列
- **Token 账本** `GET /api/usage`：按书 / 按阶段 / 全局累计 token 与调用次数（`usage.json` + `usage_global.json`，并发安全）。
- **定时任务** `GET/POST /api/schedule`：每本可设「每 N 天写一批 N 章」，`schedules.json` + 60s 调度器巡检触发，已完成书自动停用。
- **任务队列** `GET/POST /api/queue` + `/api/queue/action` + `/api/pause` / `/api/resume`：查看 / 暂停 / 继续创作任务。

---

## 创作流水线

**规划**：生成 `00 人物` / `01 大纲` / `03 文风` + 4 个连续性结构文件（`defaults.v3_pipeline` 可关）：
- `09-故事圣经.md`（全书唯一事实来源）· `10-线索账本.md`（悬念/伏笔 + 兑现计划，随章追加）
- `11-章节摘要.md`（随章补写）· `12-时间线与连续性.md`。

**总架构熔炼**（v0.6+）：写正文前先熔炼 `02b-总架构`（全书单一事实来源）。

**逐章创作**：写前注入 故事圣经 / 线索账本 / 章节摘要 / 时间线 + **题材 craft（推理/言情/恐怖/都市/玄幻，全平台通配）** + **反 AI 腔清单**；前 3 章过**黄金三章门**。每写完一章自动刷新**写作骨架**（`17-写作骨架.md`）。

**P5 故事图谱 + 语义检索**（纯标准库，无外部 DB / 向量库，仍「任何电脑可跑」）：
- **故事图谱** `bot/graph.py` → `15-故事图谱.json`：抽 人物 / 术语 / 伏笔 为节点，按章取「相关实体 + 活跃伏笔」注入写作；
- **语义检索** `bot/retrieval.py`：TF-IDF 召回跨章既往片段，补「只看上一章」看不到的长期上下文；
- **读者人格试读** `reader_panel`（默认关，按需开）：3 类读者试读 → 追读 / 弃书 / 建议，写入 14。

**质检 → 重写 → 放行**：三层质检（见上）；未达标 `autofix` 多轮；放行门禁重写到 ≥ 达标线。
产物：`13-修订记录` / `14-质量报告` / `16-重写要求书·记录` / `17-写作骨架`。

---

## 快速开始

```bash
# 需要 Python 3.9+（自动建 .venv、装依赖、起服务）
python run.py            # 各平台一键：./start.sh (macOS/Linux) · start.bat / start.ps1 (Windows)
```

- 打开 `http://127.0.0.1:46821`（默认端口；`python run.py -p 5000` 或 `NOVEL_BOT_PORT=5000` 可改）。
- 首次运行：① 自动从模板生成 `config.json`（相对路径、无密钥）；② 网页「模型供应商」绑定一家供应商 + 填 API key（或用环境变量）；③ 左侧丢主题开写，或点「✍ 开书向导」。

> 依赖只有 `fastapi / uvicorn / httpx`（见 `requirements.txt`）。**不含任何**用户目录 / 外部技能仓库 / 向量库 / 数据库依赖——任何一台装好 Python 的电脑都能跑。

---

## 模型供应商与密钥

对每个已绑定供应商，API key 依次取（缺了不崩，逐级回落）：
1. 网页「绑定 / 编辑供应商」里填的 inline key（存本地 `providers.json`）；
2. 环境变量：`AGNES_API_KEY` / `OPENAI_API_KEY` / `DEEPSEEK_API_KEY` / `QWEN_API_KEY` / `GLM_API_KEY` / `KIMI_API_KEY`，或通用 `NOVEL_BOT_API_KEY`；
3. 本地 `secrets/<slug>.key`（项目内 `secrets/`，已 gitignore）；
4. 兼容旧 `api_key_file` 绝对路径。

> **像 git 那样**：`config.json` / `providers.json` / `secrets/` / `projects/` / `data/` 全在 `.gitignore`——**每台电脑各自生成、各配各 key，不共享、不入库**。公开仓库里没有密钥（`config.example.json` 是无密钥模板，clone 下来各填各的）。

---

## 内置原创技能

- 技能随包放在 `skill/`：**「一幕·神笔 · ShenBi NovelCraft」全部文字原创**（`SKILL.md` + 各 guide + 6 模板 + 5 题材 craft + 反 AI 腔 + 黄金三章门），**零第三方文本依赖**。
- 方法论吸收的是**业界通用思想**（故事圣经 / 伏笔账本 / 连续性核对 / 黄金三章 / 爽点密度 / 反 AI 腔等），不复制任何第三方技能文本；详见 `NOTICE.md`。
- `config.json.skill_path` 默认相对路径 `"skill"`；`bot/skill.py` 实时读取，**改技能文本无需改代码**。

---

## API 一览

> 完整端点 68 个，按功能分组（参数略，详见 `app.py` 各 docstring）。所有 `/api/*` 返回 JSON；`key` 为作品唯一标识。

**开书 / 对话 / 指令**
`POST /api/create`（REST 直接开书，`cfg_overrides` 可带 stage_models / p5_pipeline / reader_panel）· `POST /api/chat`（万能对话，自动判断开书/续写/改写）· `POST /api/cmd`（即时修正指令，立即生效不改已写）· `POST /api/continue`（续写 N 章）· `POST /api/pause` / `POST /api/resume`（暂停 / 继续）。

**作品库 / 对比**
`GET /api/library?sort&order&genre&status&q`（多键排序 + 筛选 + 统计 + 题材分组；`status=archived`=回收站）· `POST /api/library/{key}/favorite|tags|archive|rename` · `DELETE /api/library/{key}`（`?hard=true` 彻底删）· `POST /api/library/batch {keys,action}`（批量审稿/对比/收藏/归档/恢复）· `GET /api/compare?keys=a,b`。

**书内查看 / 骨架**
`GET /api/book-live?key=`（这本书正在干什么 + 各章进度）· `GET /api/status?key=`（阶段 / 章节 / 日志）· `GET /api/chapter?key&n`（取某章正文）· `GET /api/preview?key=`（成书 / 已写章节 + 章节清单 + 逐章评分）· `GET /api/blueprint?key=`（总架构，未生成给规则式预览）· `GET /api/skeleton?key&regen=` / `POST /api/skeleton`（写作骨架，写到哪里/多少/重点/为何/续写提示）· `GET/POST /api/bkchat?key=`（该书独立对话记录）。

**质检 / 重写 / 放行**
`POST /api/validate {key,mode,max_rounds}`（`report`=诊断 / `autofix`=有界重写，写 13/14）· `GET /api/quality?key=`（14 质量报告）· `GET /api/review?key=`（13 修订记录）· `POST /api/rewrite-plan {key}`（提炼主攻方向 + 逐章待修，写 16 要求书）· `POST /api/rewrite {key,chapters?,dimensions?}`（定向重写 + 前后分对比，写 16 记录）· `GET /api/rewrite-progress` / `POST /api/rewrite-cancel` / `GET /api/rewrite-log` · `POST /api/rewrite-section`（单段重写）· `POST /api/release?key`（放行门禁）· `GET /api/release-log` · `POST /api/release-cancel`。

**多 Agent / 并发 / 一键 / 总览**
`GET /api/agents`（各岗位模型绑定 + 模型树 + 全局并发上限）· `POST /api/agents-set`（给某岗位配 provider/model）· `POST /api/agents-maxglobal`（设全局并发上限）· `POST /api/agents-dispatch {keys}`（批量并发多书，最多 8）· `POST /api/parallel-write {key,workers}`（单书 N 个 Agent 分章齐写，1-8）· `GET /api/parallel-log?key=` · `POST /api/one-shot {key,auto_rewrite,auto_release}`（一键完稿全自动）· `GET /api/studio-overview`（所有书 mini 进度 + 运行态看板）。

**开书向导 / 书名 / 风格 / 蒸馏**
`GET /api/onboarding`（题材/基调/卖点/视角/章数 + 默认门禁/P5）· `GET /api/titles?key=`（8 个候选 A/B/C…）· `POST /api/pick-title {key,title}` · `GET/PUT /api/style`（该书技能）· `GET /api/styles` / `GET /api/styles/{name}` / `DELETE /api/styles/{name}` · `POST /api/distill`（语料蒸馏成风格卡片）· `GET /api/learn`（跨书学习画像）。

**模型供应商 / 规则 / Token**
`GET /api/providers`（已绑定供应商列表）· `GET /api/providers/{pid}/models` / `POST /api/providers/{pid}/models` · `GET /api/model_options`（各供应商可用模型，供下拉）· `POST /api/providers`（绑定）· `PUT /api/providers/{pid}` / `DELETE /api/providers/{pid}` · `POST /api/providers/{pid}/default|active|test` · `GET/POST /api/model_rules`（模型规则引擎）· `GET /api/usage`（Token 账本）。

**定时 / 队列 / 杂项**
`GET/POST /api/schedule`（定时任务）· `GET/POST /api/queue` + `POST /api/queue/action`（任务队列）· `GET /api/projects`（运行中项目）· `GET /api/debug_sessions` · `GET /api/download?key=`（下载成书）· `GET /api/version`（当前版本）。

---

## 目录结构

```
app.py              FastAPI 服务（路由 + 会话状态机 + 调度器 + 并发护栏）
bot/                创作引擎
                    llm（多供应商 LLM + Token 采集）· workflow（规划流水线）· writer（逐章/并行/重写/一键）
                    continuity（L1 确定性引擎）· graph（故事图谱）· retrieval（TF-IDF 语义检索）
                    skeleton（写作骨架聚合）· distill（风格蒸馏）· learn（跨书学习）· model_rules（模型规则引擎）
                    agents（多 Agent 宇宙 + 并发护栏）· skill（内置技能加载）· wordcount / version
skill/              内置「一幕·神笔」原创技能（SKILL.md + guides + templates + genre）
static/             网页端（三栏工作台 + 全部功能，单文件内联）
run.py / start.*    跨平台启动器（自动建 venv + 装依赖 + 起服务）
requirements.txt    依赖（fastapi / uvicorn / httpx）
config.example.json 配置模板（无密钥；首次运行生成 config.json）
LICENSE / NOTICE.md 代码 MIT + 技能原创 / 借鉴说明
tests/              后端 pytest + 前端 jsdom 交互回归
VERSION             版本号（单一来源）
projects/  providers.json  secrets/  data/   运行期数据（全部 gitignore，每台电脑各自）
```

---

## 配置要点（`config.json → defaults`）

- `validation`：`mode`（`report`/`autofix`）、`enabled`、`max_fix_rounds`、`pass_line`（默认 **90**）。
- `release_gate`：`true`=成书自动跑放行门禁，`false`=仅手动；`max_release_rewrite` 每轮改写 top-N 章。
- `concurrency`：默认 **2**（同时写几本，超出排队）。
- `stage_models`：写稿 / 规划 / 审校 分阶段模型路由（空=沿用默认；可填 `{provider_id, model}`）。
- `p5_pipeline`（默认开）：故事图谱 + 语义检索；`reader_panel`（默认关）：读者人格试读。
- 节奏 / 超时（防限流）：`call_gap`（两次 LLM 调用最小间隔秒，默认 2，规避 429）、`llm_timeout`（每调用，默认 900）、`planning_timeout`（规划放宽，默认 1200）。

---

## 许可与版权

- **代码**：MIT（`LICENSE`）。**技能文本**：自有版权（全原创，可安全商业授权；借鉴说明见 `NOTICE.md`）。
- 商业授权交付 `LICENSE` / `NOTICE.md`；`config / providers / secrets / projects / data` 不入库、每台电脑自管。

> 核心功能**只增不删**：旧作品照常 读 / 续写 / 预览 / 下载 / 审稿 / 质检（缺新文件自动省略）。
