---
name: hana-optimizer
description: "优化大师（hana-optimizer）：把「工作区治理规范集」落成三个动作——初始化 / 优化 / 整理，覆盖四个板块（工作区 / 专项工作文件夹 / 拓展·本体 / 经验·记忆），四个板块都做清理与整理，各按各的规则与通道。当用户说「优化大师」「优化工作区」「初始化工作区」「整理工作区」「扫描清理工作区」「整理散件」「体检工作区」「四板块盘点」「清理本体」「整理记忆」「清理外部文件夹」，或要清理超龄临时件、处理反悔区、归位文件、修剪经验与记忆、卸载扩展时使用。Optimizer: workspace + external folders + Hana home + memory, all four zones get cleanup and tidy."
---

# 优化大师（hana-optimizer）

把「工作区治理规范集」落成**三个可执行动作**：**认识对象 → 制定怎么动 → 让它按节奏发生**。

**这是 skill，不是程序——没有独立引擎。** 手就是我自己：用 `read`/`ls`/`find`/`grep`/`exec_command` 现场扫描与搬运，加上平台工具（`extension_manager`、记忆工具、`automation`）。规则内置在 skill 里，判断留给我，拍板留给人。

- **我（脑 ＋ 手）**：扫描、分类、出方案、执行、验证。
- **人（拍板）**：所有"需确认"的清理，以及本体红门、外部跨根的最终执行。
- **规则三来源（优先级从高到低）**：① `<工作区>/.optimize/rules.json`（机器可读阈值/白名单/保护区）＞ ② 本 skill **内置** `references/rules/`（自包含、装即用的规范快照）＞ ③ 项目维护者的权威源副本（hana-optimizer 仓库 `docs/`，不随安装包分发）。

**统一护栏（谁都绕不过）**：`扫描 → 分类 → 出三档方案 → 人确认 → 执行 → 验证`。删除一律先软删（进反悔区）或备份。

**环境对位（读前必知）**：本套文档写自一个具体实例；文中的目录名（如出厂默认名 `OH-WorkSpace`）、任务时刻、统计数字与历史事件，都是**那个实例的现场痕迹，不是你的现场**。你的实例可能结构不同、没建过 `.optimize/rules.json`、任务与记忆体量各异——**对不上是常态**。执行任何动作前，先做一轮对位：

1. **探测现场**：实际查看工作台根结构、`rules.json` 是否存在、挂载表（`studio-mounts.json`）与 `agents/{id}` 的真实现状；
2. **对照假设**：与本文档的「推荐结构 / 目标态」比对，列出差异清单；
3. **落差处理**：目标结构缺失 → 先走「01 初始化」补齐，或在方案中声明以现场为准；现场是另一套成熟结构 → 不为它动刀，把本 skill 的方法对位到现场实际目录执行；判不准 → 停下来问用户；
4. **不信纸面**：文档中的任何具体数字、任务名、历史事件，都不作为现场事实去核对或执行。

---

## 〇、四个板块（治理对象）

| # | 板块 | 根 | 怎么扫 | 执行通道 | 红线 |
|---|---|---|---|---|---|
| 1 | **工作区** | 主工作台（= `desk.home_folder`；出厂默认名 `OH-WorkSpace`，以你的实际值为准） | 我现场盘点根 + 六格 + `_temp` 时钟 | 我用 `exec_command` 搬进 `_temp/反悔区/{批次}/`（只搬不删） | 保护区（projects/storage/个性化/.server/平台常驻户）只由 FULL 域动 |
| 2 | **专项工作文件夹** | 工作台外挂载根（`<外部根>` 等）；**开工动态读挂载表** | 我按各根入口件/产物格/运行态/临时件盘点；**每日随治理任务只读巡检**＋季度深度盘点 | **默认人执行**；经许可由我用 `exec_command`（FULL 域）按方案清 | 运行态（`world`/`data`/`*.db`）、唯一存底不删；默认只清"可再生" |
| 3 | **拓展 · 本体** | `~/.hanako`（apps / skills / recipes / extensions ＋自管目录） | 我读四分区 + 自管体量 + 孤儿 `app-data` | `extension_manager`（卸载/更新）＋ 黄门项清（人/许可脚本） | **红门绝不碰**；绿门平台自清 |
| 4 | **经验 · 记忆** | `~/.hanako/agents/{id}/`（记忆 / 置顶 / 经验库） | 我读 pinned / experience / memory 计数与体量 | 平台记忆工具 ＋ 文件面（`pinned.md` / `experience/*.md`） | 派生不手改；`facts.db` 的 `-wal/-shm` 不动；取代 ≠ 删除 |

> 板块编号 vs 规范集"三法域＋第 4 法域"的映射：**1=工作区、3=本体、2=外部、4=知识法域**。
> **"只报告"只剩一个精确含义**：**不产生可执行动作的那部分**（平台自清项、不可再生项）。它们照样列进方案，只是不必动手。

---

## 一、入口：说「优化大师」

**一进来就展示这个菜单，等用户选，不自行推进。**（直接输出文字，不用卡片。）

```
优化大师（hana-optimizer）—— 选一个动作

  01  初始化   建骨架 / 落规则 / 注入落盘纪律 / 创建定时任务
  02  优化     扫描 → 三档方案 → 你确认 → 清理 → 整理 → 检查点
  03  整理     文件与文件夹归位 + 经验记忆修剪
  ─────────────────────────────────────────────
  00  功能     看看优化大师能做什么（总览）
```

- 用户回一个数字就执行对应动作；也可直接说"优化"/"整理"/"初始化"。
- 动作执行完，回到菜单问下一步，**不要连着把三个动作全跑完**。
- 02 / 03 默认**四板块全做**；用户点名某一板块就只做那一块。

**回 00 时输出这份「功能总览」**（不是体检查询，只介绍能做啥）：

```
优化大师能做什么

【四板块 · 清理与整理】
  工作区     六格散件归位、超龄临时件清理、反悔区回滚        （每日）
  外部文件夹  多版 dist / 环境格 / 临时件 / 路径腐烂，出方案清  （每日只读＋每季深盘）
  本体拓展   扩展差集、孤儿数据、缓存/日志/回收站长尾，走通道  （每月）
  经验记忆   pin 下沉、经验去重合并、断档/双源校验            （每月 + 每季）

【三个动作】
  01 初始化   建骨架 · 落规则 · 注入纪律 · 建定时任务
  02 优化     四板块体检 → 三档方案 → 清理 → 整理 → 检查点
  03 整理     文件归位（十步关联同步）＋ 记忆修剪（九项操作）

【安全底线】
  删除先软删进反悔区；本体红门绝不碰；每个动作都出方案、等你确认
```

---

## 二、通用纪律（三个动作都守）

1. **绝对路径 + 开工核对**：命令里一律绝对路径；开工先 `(Get-Location).Path` 核对，不符只读汇报。
2. **两段式**：清理/归位必走 `扫描 → 出方案 →（给用户看）→ 确认 → 执行`。执行前再核一次目标仍在、没被别人动过（用前一步记下的 mtime/size 快速比对）。
3. **只搬不删（工作区）**：删除一律先搬进 `_temp/反悔区/{批次}/` 并写 `.manifest.jsonl`，保留可回滚；**硬删只在用户明确同意后**做。搬移前逐件确认路径与体积；批量先小后大。
4. **结构格自身不判空**：六格（projects/storage/inbox/_temp/个性化/.server）**永不被当"空目录"清理**——`inbox` 是前门、`_temp` 是作业区，空着是正常的。
5. **四板块红线（各自不同，逐板块守）**：
   - **工作区**：`protectedPaths`（projects / storage / 个性化 / .server / 平台常驻户）只由 FULL 域执行，不做无谓搬移。
   - **外部**：运行态格（`world`/`data`/`*.db`）与唯一存底**不进删除计划**；`.git` 与凭据不碰。
   - **本体**：**红门绝不碰**（凭据 `auth.json`/`provider-catalog.json`/`security/`、核心索引 `session-manifest.db`/`data-epoch.json`、`artifacts/` 整体除 staging、`agents/` 正文、`attachments/`、`checkpoints/`、`migration-backups/`、`user/*.json`）；扩展**卸载/更新只走 `extension_manager`**，不手删目录。
   - **记忆**：不手改派生件（`memory.md`/`week.md`/`experience.md`）；不动 `facts.db` 的 `-wal`/`-shm`；**取代 ≠ 删除**（矛盾事实标失效、留历史）。
6. **本体/外部/记忆不做自动删除**：这三板块出方案后，执行走通道（`extension_manager` / 平台记忆工具 / 人 / 经许可的 FULL 域命令）。
7. **检查点**：每次动了东西，写 `_temp/检查点/{YYYYMMDD}-归类检查点.md`，三张表（操作记录 / 关联更新 / 指针复核），保留 10 份。没动东西 → 一句"工作区很干净 ⭐"。
8. **规则来源**：见开头三来源。规则改动同批更新 `<工作区>/.optimize/rules.json` 与内置规则文档，防抄本漂移。

---

## 三、动作 01 · 初始化

**目标**：把工作区变成"按规则运转"的形态：骨架就位、规则落盘、纪律入场、定时任务接管。

1. **确认目标根**：默认主工作台。读 `~/.hanako/studio-mounts.json`（UTF-8）列出挂载，选带 write 的那个；含中文/空格的路径全程加引号。
2. **建骨架**（`exec_command`，幂等，只补缺失）：
   - 六格：`projects` `storage` `inbox` `_temp` `个性化` `.server`
   - `storage`：`docs` `存底` `tools`
   - `_temp`：`截图` `数据` `一次性` `导出` `jobs` `检查点` `反悔区`
   ```powershell
   $root='<工作区>'
   $dirs=@('projects','storage','inbox','_temp','个性化','.server',
           'storage\docs','storage\存底','storage\tools',
           '_temp\截图','_temp\数据','_temp\一次性','_temp\导出','_temp\jobs','_temp\检查点','_temp\反悔区')
   foreach($d in $dirs){ New-Item -ItemType Directory -Force -Path (Join-Path $root $d) | Out-Null }
   ```
3. **落规则**：把 skill 内置的 `engine/rules.default.json` 复制为 `<工作区>/.optimize/rules.json`（已存在则不覆盖）。
4. **注入纪律**：在根 `AGENTS.md` 注入 `<!-- optimize-master:begin --> … <!-- optimize-master:end -->` 标记块（内容见 §八「落盘纪律」）；已存在则原位替换，不存在则新建。
5. **收规范**（可选）：把本 skill 的规范文档（`references/rules/`）复制进 `storage/docs/`（不覆盖已有）——让工作区自持一份，便于离线查阅。
6. **报告**：一句话说清建了哪些目录、规则落在哪、纪律注入了没、收了哪些规范。**本地可逆动作，直接做，不等确认。**
7. **创建定时任务**（可选，**必须让用户确认**）：用 `automation` 建“每日治理巡检”（多根版）；**无同名→`create`，已有→`update`（附 id）；同一任务同一轮只提一次**（防建议互相顶替）。prompt 自包含（第〇步核对 cwd + 动态读挂载表、绝对路径、主工作台 safe 档、外部根只读、本体只报告、检查点三表）。模板见 `references/自动任务模板.md`。`create`/`update` 只产待确认建议，用户点确认或回 `/apply [短码]` 才落库；**“确认”二字无效**。
8. **校验**：现场 `ls` 根，确认六格到位；把 `rules.json` 读回核对。

> 初始化**不碰本体、不碰外部**，只动主工作台的顶层骨架。

---

## 四、动作 02 · 优化（四板块）

**流程（严格按序，四板块同构）**：

```
① 扫描   我按板块现场盘点（见下表）→ 只读体检
② 分类   我按四类对象（可再生/到期/孤儿/不可再生）与板块归属组织
③ 出方案 我凑成三档方案（每板块一份）：可现清 / 需确认 / 绝不碰
④ 人拍板 把方案摊给用户
⑤ 执行   板块1：我搬进反悔区；板块2：人/许可后我 FULL 域清；板块3：extension_manager/人；板块4：平台工具/文件面
⑥ 整理   对"待归位"散件走动作 03 的归位逻辑（不省空间，单独说清）
⑦ 沉淀   写检查点三表 ＋ 关联同步（见 03）＋ 验证闭环（见 §七）
```

**四板块各扫什么、怎么处置、执行通道**：

| 板块 | 怎么扫 | 方案要点 | 执行通道 |
|---|---|---|---|
| **工作区** | 见 §四.1 十二类扫描 | safe（超龄 / 空目录 / junk / 自动归家）→ 反悔区；confirm（散件 / inbox 滞留）→ 人判去向；oversized 永不进删除计划 | 我 `exec_command` 搬反悔区 |
| **专项工作文件夹** | 各挂载根（**动态读挂载表**）：入口件、`dist*` 多版、环境格、运行态、`_temp`、大文件、路径腐烂；**日常只读巡检 + 季度深度盘点** | 多版 `dist*`（留当前+上一版）、`node_modules/.venv`（可再生）、运行态（不删）、临时件、大文件 → 三档 | **默认人**；许可后我 FULL 域清（日常巡检**零写入**） |
| **拓展 · 本体** | 四分区 + `installs.json`；自管体量（`logs`/`uploads`/`.ephemeral`/`artifacts`/`plugin-backups`/`trash`/`.cache`）；孤儿 `app-data` | 扩展差集 / 孤儿 / 黄门项 → 三档；红门只列名 | `extension_manager`（卸载/更新）；黄门项人/许可脚本清（**先退出 Hana**） |
| **经验 · 记忆** | `pinned.md` 字数、`experience/` 篇数与类数、`memory/summaries`、`daily`、`facts.db` 体积 | pin 超限 / 类粒度过细 / 断档 / 双源漂移 → 07 的九项操作 | 原生通道 ＞ 文件面 ＞ 人工 |

### 4.1 工作区扫描清单（十二类，照 04 附录 C）

| # | 类 | 判据 | 档 |
|---|---|---|---|
| 1 | `rootStrayAuto` | 顶层文件后缀命中 `rules.json.classify`（图片类→inbox） | 可现清（自动归家） |
| 2 | `rootStray` | 其余顶层散件（白名单外、非豁免） | 需确认 |
| 3 | `expired` | `_temp/*` 超 `tempLifecycle` 天数 | 可现清 |
| 4 | `inboxStale` | `inbox/` 项滞留 > `inboxStaleDays`(14) | 需确认 |
| 5 | `pruneCheckpoints` | `_temp/检查点` 超 `checkpointKeep`(10) 份（按日期留最新） | 可现清 |
| 6 | `pruneLogs` | `_temp` 内 `*.log`/`*.tmp` 超 `logKeepDays`(3) | 可现清 |
| 7 | `pruneBackups` | `storage/存底` 超 `backupKeep`(7) 份（只认我方命名前缀） | 需确认 |
| 8 | `junk` | `Thumbs.db`/`desktop.ini`/`.DS_Store`/`$null` | 可现清 |
| 9 | `oversized` | 单件 > `oversizedMB`(50)，**永不进删除计划** | 仅报告 |
| 10 | `idleTools` | `storage/tools` 脚本闲置 > 90 天 | 需确认（归档提示） |
| 11 | `emptyDirs` | 非豁免空目录（**六格自身豁免**） | 可现清 |
| 12 | `reclaim` | 体量统计（清理前后对比用） | 仅报告 |

**护栏**：执行前再核目标存在且未被改动；保护区、本体红门、oversized 不进删除计划；单批 ≤30 项、≤200MB、顶层 ≤2。

**反悔区（仅板块 1）**：
- 看：`ls _temp/反悔区/`；每批次一个目录 + `.manifest.jsonl`。
- 放回：按 manifest 的 `from/to` 原路 `Move-Item` 回去（**原位被占用即拒**）。
- 出清（硬删）：用户明确同意后 `Remove-Item -Recurse`；先确认批次是到期件。
- 板块 2/3/4 **默认不留反悔区**；要留退路用该域自建 `_temp/`，或先备份。

---

## 五、动作 03 · 整理（四板块）

**两截：文件与文件夹归位（板块 1/2/3）+ 经验记忆修剪（板块 4）。**

### A. 文件与文件夹归位（板块 1/2/3）
1. 来源：工作区散件 / `inbox` 滞留；外部根散件；本体四分区。
2. 判去向走 **Q0 + 四问**（见 `references/rules/01-OH-WorkSpace结构规范.md` §4）：
   - **Q0** 撞平台保留名（OH-Works / OH-媒体库 / 魔术卡）→ 就地不动。
   - **Q1** 本地活 → `projects/{名}/`；**Q2** 运维/产品线 → `<外部根>`；**Q3** 临时件 → `_temp/{时钟子目录}/`；**Q4** 说不清 → `inbox/`。
3. **外部根归位**：显式绝对路径、一次一根、各自入口件（`README.md` + `项目说明.md`）；散件落该域的归档位，不塞回工作区。
4. **唯一存底先判**：在 `projects`＋`storage/tools` 检索（grep 文件名+首行特征），零命中 → 判"不可再生/唯一存底" → 落 `projects/{项目}/docs/{原名}-待并入.md`，**绝不进一次性/反悔区/删除**。
5. **执行**：`storage/`、`projects/` 是保护区，我用 `exec_command`（FULL 域）搬运；外部根同样走 FULL 域；进 `inbox`/反悔区同用 `exec_command`。**搬前必出方案、必留痕。**
6. **撤销窗口**：归家后别急在原位放新文件（放回时"原位被占用即拒"）。
7. **关联同步（动了就必须做，顺序固定十步）**：① pinned 记忆（`unpin`+`pin`）→ ② 经验库（`recall_experience` 查重 → `record_experience`）→ ③ 其它 SKILL.md/README/项目索引 grep 旧路径改写 → ④ `<工作区>/.optimize/rules.json` 归类表 ← ⑤ cron prompt 与根 `AGENTS.md` → ⑥ 外部根内引用 → ⑦ 本体台账与技能引用 → ⑧ git tag / 历史随迁 → ⑨ `preferences.json` → ⑩ `config.yaml` ＋ `studio-mounts.json`。

### B. 经验记忆修剪（板块 4 · 知识法域）
1. 先现场盘点：`pinned.md` 字数、`experience/` 篇数与类数、`memory/summaries`、`daily`、`facts.db` 体积。
2. 按 `references/rules/07-经验与记忆整理规范.md` 的**九项操作**出方案：去重 / 合并 / 拆分 / 降权下沉 / 退役（取代标记）/ 删除 / 消冲突 / 类目治理 / 格式修复。
3. **通道优先级**：原生通道（`PUT /memories/compiled/*`、`pin_memory`/`unpin_memory`、`DELETE /memories`）＞ 文件面（**仅 `pinned.md` 与 `experience/*.md`**）＞ 人工。
4. **红线**：不手改 `memory.md`/`week.md`/`experience.md`（派生，会被覆盖）；不动 `facts.db` 的 `-wal/-shm`；**取代 ≠ 删除**——矛盾事实优先标失效、保留历史；证据不足一律保留。
5. 分层归位：关系事实→pinned；大主题→today/daily；"以后遇同类任务怎么做"→经验库或技能。

### C. 拓展 · 本体整理（板块 3）
- **卸载/停用**：只走 `extension_manager`（`kind=app|skill|recipe|connector`，`action=remove/disable`），不手删目录。
- **孤儿 `app-data/{id}`**：确认 id 不在台账、`apps/`、内置版三处 → 判孤儿，属黄门（确认后删、先退出 Hana）。
- **黄门可再生项**（`.cache/*`、`model-catalog`、`.ephemeral/*`、`artifacts/staging`）：出方案，人/许可脚本清（关应用后）。
- **红门**：只列名，说明为何不动。

---

## 六、工作区操作手册（怎么"用我自己"干活）

> 一切从"现场实测"出发，**只信盘出来的，不信记忆里的**。所有命令绝对路径。

**看根 / 找散件**
```powershell
$root='<工作区>'
Get-ChildItem -LiteralPath $root -Force | Select-Object Name,LastWriteTime,Length
```
白名单＝`projects/storage/inbox/_temp/个性化/.server/.optimize`（＋平台常驻户、`AGENTS.md` 等豁免；现场若留有历史目录如 `_tools`，按其既有状态只读处理）；白名单外的顶层项＝散件。

**算 `_temp` 超龄**（天数取自 `rules.json.tempLifecycle`）
```powershell
Get-ChildItem -LiteralPath "$root\_temp\截图" | ForEach-Object {
  [pscustomobject]@{ Name=$_.Name; AgeDays=[math]::Round(((Get-Date)-$_.LastWriteTime).TotalDays,1) } }
```

**找 junk / oversized**
```powershell
Get-ChildItem -LiteralPath $root -Recurse -Force -File |
  Where-Object { $_.Name -in @('Thumbs.db','desktop.ini','.DS_Store') -or $_.Length -gt 50MB } |
  Select-Object FullName,@{n='MB';e={[math]::Round($_.Length/1MB,1)}}
```

**搬进反悔区（软删，带 manifest）**
```powershell
$batch=(Get-Date -Format 'yyyyMMddHHmmss'); $bin=Join-Path $root "_temp\反悔区\$batch"
New-Item -ItemType Directory -Force -Path $bin | Out-Null
Move-Item -LiteralPath "<源绝对路径>" -Destination (Join-Path $bin "<扁平名>")
Add-Content -LiteralPath (Join-Path $bin '.manifest.jsonl') -Encoding UTF8 `
  -Value ('{"from":"<相对根路径>","to":"<扁平名>","ts":' + [DateTimeOffset]::UtcNow.ToUnixTimeMilliseconds() + '}')
```

**放回 / 出清**：按 `.manifest.jsonl` 原路 `Move-Item` 回去；出清＝用户同意后 `Remove-Item -Recurse -Force`。

**硬护栏（自己给自己上）**：单批 ≤30 项、体积 ≤200MB、涉及顶层 ≤2；超了就分批或请示。搬移前逐件 `Test-Path`。

---

## 七、节奏与验证

### 7.1 节奏（照 04 §7 + 07 §8）

| 板块 | 频率 | 触发线 |
|---|---|---|
| **工作区** | **每日**巡检（定时任务） | 散件出现 / `_temp` 超龄 / `inbox` 滞留 >14 天 |
| **外部文件夹** | **每日**随治理任务只读巡检 ＋ **每季**深度盘点 | 多版 `dist*`、路径腐烂、大文件堆积、挂载根活跃度变化 |
| **拓展 · 本体** | **每月**一次，或长尾报红时触发 | 审计日志 / `trash` / 缓存体量异常；扩展差集 |
| **经验 · 记忆** | **每月**体量审计（pin/经验）＋ **每季**类目治理 | `pinned.md` 超字数上限；单条类过半；编译断档；Dream 连续失败 |

### 7.2 验证闭环（照 04 §6，动了就必须做）

1. **前后体量对比**：清理**前后各跑同一份只读盘点**，报"预估可释放 vs 实际释放"；**连续偏差 >20% 就回炉**校准规则。
2. **逐件实测**：清理后逐件 `Test-Path` 确认真的消失/到位（Windows 递归删会静默失效，逐件核）。
3. **引用完整性**：搬移后 grep 旧路径归零；孤儿清理后扫台账/记忆/技能引用有无断链。
4. **应用健康**（本体）：清理后重启自检——台账 / 市场 / 模型目录能否自愈。
5. **回滚演练**：每季度从反悔区恢复一件，验证回滚路径没坏。

> 结果写入检查点「操作记录」附注；偏差大或断链 → 当轮修正，不留到下次。

---

## 八、落盘纪律（注入 `AGENTS.md` 的内容）

```markdown
## 一、工作区落盘
- Q0 撞平台保留名（OH-Works / OH-媒体库 / 魔术卡）→ 就地不动。
- Q1 本地工作台的项目 → projects/{名}/；Q2 运维/产品线 → <外部根>；Q3 有寿命临时件 → _temp/{时钟子目录}/；Q4 说不清 → inbox/。
- 顶层白名单：projects / storage / inbox / _temp / 个性化 / .server / .optimize（＋平台常驻户；历史遗留目录按既有状态只读处理）。白名单外落笔视为违规。
## 二、交付铁律
- 只报路径不算交付：成果进会话走 stage_files；中间物落 _temp/jobs/{当日}-{会话 slug}/。
- 生成新文件时同轮声明走的是第几问。
## 三、三条禁止
- 不绕过"扫描→方案→确认"手动 mv / rm。
- 不动平台保留名目录（搬动即断链）。
- 不把 .hanako 本体件跨法域搬（本体路径即身份，只读）。
```

---

## 九、references 索引

**本 skill 自包含且无引擎**：装好即能用——`references/rules/` 内置完整规范（README + 01~08），`engine/rules.default.json` 内置默认规则。**规范来自 doctor 引擎时代，其中的 `doctor_*` 全是历史写法，本 skill 无引擎、一律换算为现场命令**（对照表见 `references/rules/README.md` 顶部「去引擎说明」）。项目维护者的权威源副本（仓库 `docs/`）只作更新来源，不装也能跑。

按需读取，不要一次性全 load：

- `references/rules/README.md` + `01`~`08`——**内置的规范快照**（自包含）。**判任何去向/去留/节奏前先读对应那份。**
- `references/规则地图.md`——每份文档讲什么、何时读。
- `references/自动任务模板.md`——01 初始化里创建定时任务用的 prompt 模板。
- `references/通道与工具.md`——四板块的执行通道、记忆工具、`automation`、`extension_manager` 用法要点。
- `engine/rules.default.json`——**内置默认规则表**（init 时复制为 `<工作区>/.optimize/rules.json`）。
- （项目开发仓库内另有 `_drafts/` 待落地稿，不随发布包分发——发布包里没有属正常。）

**快照与权威源同步（项目维护者事项）**：内置 `references/rules/` 是"装即用"的底线；开发仓库的 `docs/` 更新后，用 `diff`/`grep` 比对两处同名文件，不一致就同步内置副本——**仓库 docs 是源，内置是镜像**，改动同批；镜像永不反向覆盖源。普通使用者无需关心此段。

> 规则与现场实测冲突时**以实测为准**；改规则要同批改 `<工作区>/.optimize/rules.json` 与内置规则文档。

---

## 十、版本沿革

| 日期 | 变更 |
|---|---|
| 2026-09-08 | 初版：优化大师（带引擎 App + skill 双形态） |
| 2026-09-27 | 骨架规范更新：定为六格 + `storage/docs` + `inbox` |
| 2026-10-09 19:31 | **去引擎**：删 `engine/cli.mjs`，SKILL.md 重写为纯 Agent 工作流 |
| 2026-10-09 21:15 | **多根化**：自动任务模板改为“动态读挂载表 + 按使用情况分级 + 外部根只读巡检”；白名单补 `_tools`/`.optimize` |
| 2026-10-09 22:00 | **删 App + 全量去引擎**：移除引擎附属 App 与数据；`references/rules` 01~08 全文清洗 `doctor_*` → 现场命令；4 份待落地稿移入 `_drafts/` |
| 2026-10-10 | **通用化改造**：清除/示例化实例环境痕迹；新增「环境对位」机制；修复悬空引用；默认白名单移除历史遗留项 `_tools` |