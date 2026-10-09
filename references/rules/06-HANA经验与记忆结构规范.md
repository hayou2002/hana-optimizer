# 06 · HANA 经验与记忆的结构规范

> **版本** v0.1（知识法域上户口）｜ **建档** 2026-10-09 ｜ **状态** 调研已并入，未核实项见 §9
> **上游权威**：HanaAgent 内核 bundle（记忆流水线）｜ `agents/hanako/` 现场文件 ｜ 现场只读调研
> **关联**：README（总纲）｜ 02（Hana 本体）｜ 04（清理）｜ **07 整理（待建）**
> **本文件使命**：给「知识法域」上户口，讲清 **记忆与经验住在哪、怎么运转、什么算数**

---

## 0 定位：知识法域（第 4 法域）★

`agents/{id}/` 下的记忆与经验，此前三法域（工作区/本体/外部）**都没正式认领**。本套体系把它定为**第 4 法域**：

| 维度 | 说明 |
|---|---|
| 物理位置 | `~/.hanako/agents/{id}/`（属本体，但独立治理） |
| 归属单位 | **只有 agentId**（一 agent 一套；与哪个工作区无关） |
| 三个物 | ① `memory/`（编译式记忆）② `pinned-memory.json`（置顶）③ `experience/`（经验库） |
| 与工作区关系 | **正交**：切/删工作区**不动**记忆；记忆不随工作区走 |

> **为什么要单列**：04 的扫描矩阵与 05 的归位表此前**永远漏这一格**；而它恰恰是"越用越多、会干扰上下文"的那块地。

**一句话**：工作区管"文件往哪放"，知识法域管"**认识怎么攒**"。

---

## 1 对象台账（住哪 / 谁写 / 格式 / 生命周期 / 可否整理）

| 对象 | 住哪 | 谁写 | 格式 | 生命周期 | 可否整理 |
|---|---|---|---|---|---|
| **记忆总装** | `memory/memory.md` | 代码拼装（非人写） | 四段：`重要事实/今天/本周早些时候/长期情况` | 每次编译/Dream 后重写 | **派生**，手改会被覆盖 |
| 重要事实 | `memory/facts.md` | `compileFacts` | 纯正文（无标题） | 长期 | 可 `PUT /memories/compiled/facts` |
| 今天 | `memory/today.md` | `compileToday` | 纯正文列表 | 逻辑日切换清空重写 | 可 PUT；**Dream 禁改** |
| 本周 | `memory/week.md` | 从 `daily/` 近 6 天拼装 | `## 日期` 分节 | 滚动窗口（超 1200 字丢最老） | **派生**，不手改 |
| 长期情况 | `memory/longterm.md` | `compileLongterm` + daily 溢出 | 纯正文段落 | 长期 | 可 PUT；Dream 主对象 |
| 日条目 | `memory/daily/YYYY-MM-DD.md` | `compileDaily`（≤60 字） | `## 日期` + 两三句 | **保留 6 个逻辑日**，更老折进 longterm 后**删文件** | 可 PUT |
| 元事实库 | `memory/facts.db`(+`-wal`/`-shm`) | `deepMemory` 抽取 | SQLite（`facts` + FTS5 + 触发器） | **只增不减**（除会话失效/整库清） | 无单条 API，只能整库清或 SQL |
| 会话滚动摘要 | `memory/summaries/sess_*.json` | `SummaryManager`（每 10 轮/结束/分支） | JSON（摘要 + 游标 + 时间范围） | 一 session 一份，**无自动清理** | 可删（清空即重置游标） |
| 日任务账本 | `memory/daily-state.json` | ticker | 五步 completedSteps | 逻辑日重置 | 幂等状态，别手改 |
| 编译游标 | `editable-facts-state.json`、`today-state.json` | 编译步骤 | 时间戳 | 长期 | 别手改 |
| Dream 状态 | `memory/dream/state.json` | dream runner | 触发/状态/前后字数/合并数/遗忘数 | 长期 | 只读观测 |
| Dream 修订 | `memory/dream/revisions/*.json` | runner | 每版 before 四段全文 | **只留最近 10 版** | `POST .../restore` 回滚 |
| **置顶（真源）** | `pinned-memory.json`（agent 根） | 工具双写 | `{version, items:[{id, content, createdAt}]}` | 永久 | `pin/unpin` 工具 |
| **置顶（人读镜像）** | `pinned.md`（agent 根） | 同上 | `- 内容`（续行缩进） | 永久 | **人工整理的首选入口**（md 新于 json → 以 md 为准重建） |
| 经验正文 | `experience/{slug}-{sha10}.md` | `record_experience` | 首行 `<!-- experience-title: base64 -->` + 编号条目 | 永久，无自动清理 | **无删除/编辑 API**，只能改文件 |
| 经验索引 | `experience.md`（agent 根） | `record` 时刷新 | `# 分类（N 条）` + 摘要行 + 链接 | **派生** | 手改后索引不立刻更新 |
| `book/`、`heartbeat/` | agent 根 | **无人写** | — | 平台补骨架 | 空壳遗留 |

> **逻辑日 04:00 切换**（跨日后 `today.md` 清空、账本重置）。整理规范的"昨天/本周"要和平台口径一致，**不用自然日**。

---

## 2 真源 vs 派生（改哪个算数）★

**最容易踩的坑**：改派生 = 等着被覆盖；改真源 = 合法。每个真源对应的正规写入口：

| 真源 | 正规写入口 |
|---|---|
| `facts.md` / `today.md` / `longterm.md` / `daily/*` | HTTP `PUT /memories/compiled/{facts,today,longterm,week/days/:date}`；或 `update_settings("memory.facts")` |
| `facts.db` | 无单条入口（`DELETE /memories` 整库清） |
| `pinned-memory.json` | `pin_memory` / `unpin_memory`；**或改 `pinned.md`**（mtime 反超 json 即生效） |
| `experience/*.md` | `record_experience` 追加；**编辑/删除只能改文件** |
| `memory.md` / `week.md` / `experience.md` | **纯派生**——不要手改 |

---

## 3 运行机制（数据流）

```
会话消息 ──每10轮/结束/分支──▶ summaries/sess_*.json（滚动摘要）
      │                              │
      │                       compileToday / deepMemory（事实抽取 → facts.db）
      │                              ▼
   日切(04:00) 每日五步：
   compileDaily → daily/*.md（≤60字）
   compileToday → today.md
   rollDailyWindow → 超6天的 daily 折进 longterm 后删
   compileFacts → facts.md（≤200字）
   assemble → week.md（近6天，≤1200字）→ memory.md（四段总装）
```

模型口径：记忆流水线用 `models.utility_large`，缺则退 `chat`；两者都失败则 **memoryTicker 不启动**。

---

## 4 注入与预算（谁进 prompt、多少）

- **进 prompt 的只有两样，且是全文**：`memory.md`（≈918 字）+ `pinned.md`（**≈8600 字符**），落在 prompt `order 9000` 的 memory 段（scope `self`），前置一段「记忆使用规则」。
- **经验库不进 prompt**——只能靠 `recall_experience` 工具取（别搞混）。
- 三条闸门：`memory.enabled=false` → 整段不注入；`forSubagent` → 不注入（**子代理没有记忆段，也没有四个记忆工具**）；`tool_call_mode=minimal` → 只注入 `facts`+`today`。

> ⚠️ **`pinned.md` 是唯一的 token 压力点**：无上限、不进 Dream、不受预算约束——**唯一完全由人工负责的常驻上下文**。

---

## 5 配置里的「死键」（重要警告）★

`config.yaml` 的 `memory` 段里，**六个键内核根本不读**（bundle 零命中）：

`token_budget` · `decay_per_day` · `hit_bonus` · `base_importance` · `compile_threshold` · `forget_speed`

它们只活在 `config.example.yaml` 的注释和设置页文案里。**高斯衰减 `score × e^(-λt²)` 这套模型在当前实现里不存在**（`facts.db` 表里连 score/last_access 列都没有）。

**真实的淘汰机制**（取代了分数衰减）：
1. **字符/Token 预算 + 截断**（各层编译都有上限）；2. **6 天窗口滚动**（daily 溢出折进 longterm 后删）；3. **Dream 语义整理**（见 §6）；4. 会话派生态失效 + 显式重置。

**内核真正读的键**：`memory.enabled`、`memory.dream.auto_enabled`、`experience.enabled`、`models.utility_large`。

---

## 6 自动整理机制（结构侧）

**Dream**（唯一的语义整理器，五阶段）：`atomization → dedupe → optimize → compose → verify`。
- 删除**只允许三种客观理由**：`completed_transient` / `obsolete` / `operational_noise`；**证据不足一律保留**（禁止按比例/条数/年代删除）。
- 硬护栏：总字数**只许缩不许胀**；只重写 `facts.md`+`longterm.md`；每版落 revision（留 10 版，可 restore）。

**其他**：窗口滚动与截断（§5）｜会话派生态失效（删会话→清其抽取事实）｜显式重置（`DELETE /memories/compiled`）。

> 整理操作见 **07**；此处只交代"机制存在"。

---

## 7 工具与通道

| 工具 | 作用 |
|---|---|
| `search_memory` | 标签精确 + FTS5 + LIKE 回落；跨频道需 `cross_channel` |
| `pin_memory` / `unpin_memory` | 置顶增删（unpin 支持 **keyword 批量命中**）；**无 update**（改 = 撤旧 + 存新） |
| `recall_experience` / `record_experience` | 经验查/记；**无 delete/merge/move** |
| HTTP | `/memories/health`、`GET/DELETE /memories`、`export/import`、`PUT /memories/compiled/*`、`/memories/dream/*` |

> **通道缺口（07 的核心问题）**：`experience/`、`pinned`、`facts.db` 三个「只进不出」的对象，**没有逐条清理 API**——整理只能靠文件手术或人工。

---

## 8 安全边界

- `pin_memory` 与 facts 写入前**自动脱敏**：api_key 前缀、`password=…`、PEM 私钥、卡号、身份证、SSN → `[REDACTED]`。
- **脱敏不管**：手机号、住址、`record_experience` 正文——**07 得自写红线，别把自动脱敏当兜底**。
- 凭据不落明文记忆；红线同 02 §8。

---

## 9 与其它文档联动

- **02**：知识法域物理上属本体（`agents/`）；02 红门"绝不碰"需细化为"**别删，但内部怎么动见 06/07**"。
- **04**：扫描矩阵应加"知识法域"一行（摘要/经验/facts 计数、pin 字数基线）。
- **05**：关联同步第 1、2 步（记忆/经验）已有；归位表加"知识法域"。
- **07**：本文件的"真源/派生""通道缺口"是 07 的输入。

---

## 10 待优化与未核实

- **`config.yaml` 死键**：文档/设置页若仍展示"衰减系数"等，属误导——建议标注。
- **`deepMemory` vs Dream** 两个状态机的关系待核。
- **`summaries/`、`daily/` 是否有平台 TTL**：未见清理代码。
- **embedding 空**（`embedding_api.provider=""`）：语义检索大概率退化为关键词档。
- `experience.md` 索引与正文一致性、`pinned` json↔md 双源一致性——**无校验**。
- `book/`、`heartbeat/` 空壳遗留，应按可清理遗留处理。

---

## 附录 A · 一页速查

**三个物**：`memory/`（编译式，分四段）｜`pinned-memory.json`（置顶，全文常驻）｜`experience/`（经验库，不进 prompt）。
**两个只进不出**：`facts.db`、`experience/`。
**一个 token 压力点**：`pinned.md`（≈8600 字符，人工负责）。
**一条铁线**：改派生没用（`memory.md`/`week.md`/`experience.md`），改真源才算数。