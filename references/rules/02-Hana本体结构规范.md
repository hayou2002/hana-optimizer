# 02 · Hana 本体文件夹结构与存放规则

> ⚠️ **无引擎提示**：本文历史 `doctor_*` 记法来自 doctor 引擎时代，本 skill 已无引擎、该 App 已不存在——等价现场做法见 `README.md` 顶部「去引擎说明」。

> **版本** v0.4（通用化改造）｜ **更新** 2026-10-10 ｜ **状态** 未核实项见 §10
> **上游权威**：平台内核 bundle 的存储登记表（`defineStore`）｜ 现场只读调研 ｜ 官方公开资料
> **关联文档**：README（总纲）｜ 01（工作区）｜ 04（清理）｜ 05（整理）
> **本文件使命**：**让用户敢清、清得对 `~/.hanako`**（Hana 软件自己的家）

---

## 1 定位：为什么需要这份文档

`<HANA_HOME>` 是 **Hana 软件自己的家**：装扩展、存会话、记记忆、跑运行时。它**不是工作区**，平常的清理工具碰不到这里，你也不敢随便删。

两个事实决定了这份文档的存在价值：

1. **官方没有成文的清理清单**，也**没有任何清理指引**——「哪些可删、哪些不可删」在公开资料里查不到完整版。
2. **平台 CLI 没有清理子命令**。`hana` 共 8 个子命令（`serve / status / sessions / continue / chat / bundle / data / help`），正文之外没有任何 `clean`。这个能力缺口，正是这份文档要补的。

**三条铁律（全篇贯穿）**：
- **AI 出方案、人执行**：本体清理**归 04**（三法域统一流程）。"只报告"＝**不产生机器可执行计划**，**不等于"没有清理方案"**——AI 照样出三档方案，执行权在人。
- **先退出 Hana，再动手**：`.ephemeral`、`file-history`、`logs`、`app-data`、`.cache/app-ui-snapshots` 全在进程用到或按 pid 认领；运行中删会撞 WAL / lease / ownership marker。SQLite 家族（`.db`+`-wal`+`-shm`）**必须三件同删**。
- **红门绝不碰**（§4）。

---

## 2 法域与动手前提

- `.hanako` **不在任何挂载内**：工作区挂载表（`studio-mounts.json`）只含主工作台（`<工作区>`）与若干外部根（`<外部根>`）。
- **别拿优化大师清 `.hanako`**：本 skill 的护栏要求工作区根带 `_temp`/`projects` 标记、拒盘符根与家目录，反悔区也是为工作区设计的——把 HANA_HOME 交给它属越界用法。
- **动手前先跑只读体检**（官方自带）：
  - `hana data diagnose`（只读：stamp / journal / checkpoints）
  - `hana data checkpoints`（列恢复点）
  - 恢复需显式确认词：`hana data restore <transitionId>`

---

## 3 顶层一览（脚本复测口径）

**项数会随运行浮动**——例如 `chat-card-state/` 是会话里第一次铸卡后才出现的；`-wal/-shm` 会随连接出现或消失。因此本文件**不写死项数**，以脚本复测为准。

完整清单见 **附录 A**。

---

## 4 清理三道门（本体视角 · 完整规则归 04）★

> **三色门按"人的动作"分**：绿门＝平台自己删（你不必管）；黄门＝你关应用后手清；红门＝你绝不要删。**平台自删（绿门）不构成对红门的例外。** 扫描/分类/通道/验证的完整规则归 **04**；此处只讲本体视角。

不要逐目录纠结，先分门：

### 绿门 —— 平台自己会清，你什么都不用做

| 机制 | 清什么 | 周期/触发 |
|---|---|---|
| 调试日志滚动 | `logs/{时间戳}.log`：5MB 滚动留 3 份，**删 7 天前的** | 写日志时 |
| 冷会话缓存 | `session-files/` 里冷会话的托管字节 + `.archives` | 启动 + 每 24h，判据是会话 mtime，**72h** |
| 归档会话自动删 | 超期归档会话 + 其子代理/工作流记录 + 缓存目录 | 启动 + 每 24h，**90 天**（已开启） |
| 过程记录清理 | `subagent-sessions` / `workflow-sessions` / `workflow-journals` | 启动 + 定时，**30 天**（已开启） |
| 卡片回收站 | `chalkboard/card-archive.json` 过期条目 | 惰性清理，**30 天 / 上限 200** |
| 文件时光机 | `file-history/*/history.sqlite` 超龄快照 | 打开工作区 + 每 24h，**30 天 / 单库 500MB** |
| 旧式上传 | `uploads/` 中 mtime > 24h 的项 | **懒触发**：下次无会话上传时才扫 |
| 媒体暂存 | `plugin-data/.../generated` 未入库中间件 | 默认开，留 **7 天** |
| 迁移备份裁剪 | `migration-backups/` 超龄目录 | 独立保留计划 |
| OTA 产物 GC | `artifacts/` 中不被任何指针引用的旧版本 | 桌面壳更新流程 |

> **结论**：约 80% 会长的东西平台已自清。你在设置→数据 / 存储与清理里能看到的开关，就是它们的总闸。

### 黄门 —— 可手动清，且清坏了能自愈（**执行者＝人**，退出 Hana 后）

| 对象 | 怎么清 | 清了会怎样 |
|---|---|---|
| `.cache/extension-market/`、`-downloads/` | 直接删 | 下次打开市场/安装时重新下载（语义：`regenerable`） |
| `.cache/builtin-apps/` | 直接删 | 下次载入重建 |
| `model-catalog/`（current/previous/state/staging） | 直接删 | 短暂回落内置基线，刷新即恢复 |
| `.ephemeral/` 请求级暂存（card-exports/imports、extension-exports 等 token） | 关应用后删 | 一次性身份，无损失 |
| `.ephemeral/win32-sandbox-runtime/`、`runtime/pi-sdk/bin/`、`win32-sandbox-env/` | 关应用后删 | 重拷/重下（官方诊断文案自己就建议清 sandbox-runtime） |
| `session-files/` 的**空壳目录** | 直接删空目录 | 无害（下次写入重建，路径只是定位器） |
| `logs/security-audit.jsonl`、`resource-audit.jsonl` | 关应用后删（建议先备份） | 丢审计链，无功能影响（见 §5 ①） |
| `crash.log`、`browser-cmd.log`、`diagnostics/` | 关应用后删 | 丢排障对照，无功能损失 |
| 孤儿 `app-data/{id}` | 确认后删（§6） | 该 App 数据清零；重装从零开始 |
| `artifacts/staging/` 的旧 tar.gz | 确认无 in-flight 更新后删 | OTA 暂存包丢弃（§5 ③） |

### 红门 —— 绝不碰

| 类别 | 成员 | 删了会怎样 |
|---|---|---|
| 凭据与安全 | `auth.json`、`provider-catalog.json`、`users.json`、`device-credentials.json`、`*-sessions.json`、`security/` | 登录/授权/密钥链断；部分为机器密钥密封态，换机不可读 |
| 核心索引 | `session-manifest.db(+wal/shm)`、`data-epoch.json`、`workspace-state/`、`usage-ledger.sqlite` | 会话身份/纪元门禁/资源身份断链；平台恢复路径是"改名 aside 重建"而非删 |
| 内核与更新 | `artifacts/` **整体**（除 `staging`）、`provider-plugins/` | 被指针引用的版本目录一删，通道无法激活/回滚（`activation.cjs` 遇不可解析指针**故意抛错**） |
| Agent 与记忆 | `agents/{id}/` 正文、`memory/`、`session jsonl` | 对话永久丢失、记忆清零 |
| 上传原件 | `attachments/`（渠道/桥接图片视频） | 路径直接拼进消息正文、无索引，删了老聊天图片永久打不开 |
| 回滚保险 | `checkpoints/`、`data-epoch-checkpoints/`、`migration-backups/` | 迁移/纪元回退点丢失 |
| 平台态 | `user/*.json`、`input-drafts-recovery/`（受保护的原始字节）、`*.corrupt-*` | 设置清零、草稿丢 |

> **平台对 `.hanako` 没有整体备份/一键回滚。红门一旦删，唯一后路是回滚保险那三处，而它们本身也在红门里。别以为有兜底。**

---

## 5 平台漏掉的三项（真正需要你动手）

平台自管得很好，但有三处**无界增长、无人清理**：

1. **审计日志**：`logs/security-audit.jsonl`、`logs/resource-audit.jsonl` 是 append-only，**文件名不匹配 7 天清扫的正则，没有 TTL**。这是最该点名的一处"长尾"。建议关应用后手动清、先备份。
2. **回收站 `trash/`**：ResourceIO 回收站（`trash/{namespace}/trash_{ts}_{rand}/`）。花笺删除走它、工作区文件删除也会落进去，**目前找不到 TTL/清扫代码**——一旦开始用就无声积累。
3. **孤儿 App 数据 + 更新暂存**：卸载**按设计保留** `app-data/`（可能留有孤儿目录），`artifacts/staging/` 也可能留旧 tar.gz。

---

## 6 孤儿 `app-data` 判定规则

```
app-data/{id} 且 id ∉ 台账 app 集（extensions/installs.json）
              ∪ apps/ 目录
              ∪ 当前版 artifacts/server/<cur>/apps/
→ 报孤儿
```

**现成用例**：
- 某残留 `app-data/{id}` → 台账无、`apps/` 无、内置版无 = **孤儿**（卸载残留）。
- 某内置 App → 台账有（`origin: builtin`），`source.path`/`location` 指向 `artifacts/server/.../apps/{id}` = **内置 App，不是孤儿**，它数据不能按孤儿删。

---

## 7 扩展的落点与卸载

| 资产 | 落点 | 卸载通道 |
|---|---|---|
| Skill | `skills/{name}/SKILL.md` | `extension_manager kind=skill action=remove` |
| Recipe | `recipes/{name}/SKILL.md` | 同（另有 recipes 路由） |
| App | `apps/{id}/`（`manifestVersion: 2`） | `extension_manager kind=app action=remove` |
| Connector | 台账在 `extensions/installs.json` | 平台连接器入口 |

- **卸载后数据保留**：插件代码删掉，`plugin-data/{id}/` **保留**；App 代码删掉，`app-data/{id}/` **保留**；升级前备份到 `plugin-backups/`。
- **内置技能/配方删了会重铺**（每次启动按打包内容原子重写）——手删白费功夫。
- **手删自定义技能 = 台账与 Agent 引用悬空**；只有走卸载入口才清引用。
- `plugins/`（v1，如 `time-sense`）**不在 installs.json 台账**，卸载入口未验证（§10）。

---

## 8 安全与凭据边界

- `user/preferences.json` 存有第三方搜索密钥**明文**；`migration-backups/` 与 `extensions/staging/*/connector.secret.json` 可能含同等凭据（部分为机器密钥密封态）。
- 因此：**不要把 `.hanako` 打包上传求助**。真要发日志，只发 `logs/*.log` 与 `diagnostics/`。
- 凭据家族的字节数**不参与任何清理讨论**。

---

## 9 与工作区的边界

| 维度 | 工作区（01 管） | 本体（02 管） |
|---|---|---|
| 根 | `desk.home_folder`（挂载内） | `<HANA_HOME>` |
| 治理 | 读写全通道（Agent 现场命令） | **只读只报告**（现场只读盘点） |
| 清理 | 04（工作区：机器执行） | 04（本体：AI 出方案、人执行） |
| 回收 | 反悔区 `_temp/反悔区` | 平台 `trash/` |
| 知识法域 | — | **记忆与经验见 06/07**（第 4 法域，物理住 `agents/{id}/`） |

**要动本体里的东西（设置/扩展/会话/记忆），走平台入口；不要用工作区工具越界。**

---

## 10 待优化与未核实

- **空间体量为定性**：各目录的"占用倾向"是按增长机制推的，未逐项实测字节数。要写"清这里能省多少 G"，需现场补一轮带 shell 的测量。
- **`artifacts` GC 的保留份数**（保留 current+previous？宽限期几天？）在桌面壳包里，可读范围未读到。
- **`plugins/`（v1）卸载通道**未验证。
- **`trash/` 的 TTL**：机制存在，未找到清扫代码。
- 顶层计数不写死，改用脚本复测口径。

---

## 附录 A · 顶层清单（三色门分组）

> 项数随运行浮动，以脚本复测为准。

**绿门（平台自清）**〔多为②到期，OTA GC 属①〕：`logs/*.log`(7天)、`session-files/`(72h)、归档会话(90天)、`subagent-sessions`/`workflow-*`(30天)、`chalkboard/card-archive`、`file-history/`、`uploads/`(懒清)、`plugin-data/image-gen/generated`、`migration-backups`(保留期)、`artifacts` 旧版本(GC)

**黄门（可手清·自愈）**〔多为①可再生；审计/诊断属②；孤儿 app-data 属③〕：`.cache/app-ui-snapshots`、`.cache/extension-market`、`.cache/extension-market-downloads`、`.cache/builtin-apps`、`model-catalog/`、`.ephemeral/*`（请求暂存、`terminal-sessions`、`win32-sandbox-env`、`win32-sandbox-runtime`）、`runtime/pi-sdk/bin`、`session-files/` 空壳、`logs/*.jsonl`(审计)、`crash.log`、`browser-cmd.log`、`diagnostics/`、孤儿 `app-data/{id}`、`artifacts/staging/*.tar.gz`

**红门（绝不碰）**〔④不可再生〕：`auth.json`、`provider-catalog.json`、`users.json`、`device-credentials.json`、`local-user-auth.json`、`pairing-sessions.json`、`web-sessions.json`、`security/`、`session-manifest.db(+wal/shm)`、`data-epoch.json`、`workspace-state/`、`usage-ledger.*`、`artifacts/{server,renderer,pointers,ota-state*}`、`provider-plugins/`、`agents/`、`attachments/`、`checkpoints/`、`data-epoch-checkpoints/`、`input-drafts-recovery/`、`input-drafts.v2/`、`user/`、`studios*/`、`studio-mounts.json`、`devices.json`、`opened-documents.json`、`subagent-*.json`、`workflow-activity.json`、`update-channel.json`、`last-update-version`、`added-models.yaml`、`models*.json`（`models-cache.json` 例外，regenerable）

**其余（只报告）**：`bridge/`（可恢复冷状态）、`channels/`（空）、`card-guide/`、`chat-card-state/`、`plugins/`、`plugin-data/`（除 image-gen 暂存）

## 附录 B · 平台自带清理机制（速查）

见 §4 绿门表。要点：**平台已自清约 80%**，用户真正要动手的只有 §5 那三项。

## 附录 C · 删错了怎么回

- **黄门的 regenerable 项**：下次自动长回来（市场缓存、模型目录、内置技能/配方、沙盒缓存重拷）。
- **红门**：C 无整体回滚；唯一后路是 `checkpoints/`、`data-epoch-checkpoints/`、`migration-backups/`（皆在红门内）。**红门 = 真丢失。**
- **回滚**：本体**没有清理型反悔窗口**；`trash/` 是资源删除桶（**无 TTL**），不是本体自管目录的回收站。要留退路只能**人工备份**（复制到安全位置 → 清单登记 → 关应用手删）。**不存在"跨法域移进工作区反悔区"这条路**（反悔区只认默认根、AI 对本体只读）。