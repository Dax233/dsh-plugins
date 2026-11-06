# DSH Desktop 0.2.1-rc.5 → 0.2.2-rc.1 升级指南

- 适用：Windows Desktop，nightly 更新通道；走完本机（主设备）真实升级全过程后整理
- 日期：2026-11-06
- 版本事实：0.2.2-rc.1 已全量发布（feed `https://download.deepseek.com/dsh-desk/feeds/win-x64/nightly.yml`，安装包 293,425,336 B / sha512 `swcK0J6of...`；GitHub tag `dsh-v0.2.2-rc.1`；npm dist-tag latest=0.2.2-rc.1）
- 结论：**0.2.2 对普通使用无感，但有三处会咬人**，先做 §1 再升级

---

## 0. 0.2.2 关键变化速览

| 变化 | 对谁有影响 |
| --- | --- |
| Agent 路由在首个请求前固定：`agent/request` 不再允许改路由 | 自带/引用了带 `installModelSelection(两参)` 旧 bundle 的插件（session-bridge 旧版即中招） |
| Session Projection 注册改异步（必须 `await` 并持有 handle） | 注册投影的自定义插件 |
| tools 执行器插件化：`tools.mode`/`DSH_TOOLS_MODE` 删除，改 `dsh-tool-executor-native/ptc` 行 | 只有显式写过 tools.mode/环境变量的部署（本生态无） |
| pi-ai 升 1.3.0；`llm-pi-ai` 适配器源码变了（rc.5 与 0.2.2 sha256 不同） | **provider-aliases 覆盖包必须重新 sync** |
| `fs-observation-policy` 默认改为 disabled（先读后写守卫默认关闭）+ 新增 `apply_patch` 工作 | 想保留守卫的部署需一行 patch |
| 独立「Git 工作树」bundle 合入「变更请求与工作树」（forge） | bundles 里同时开两个会 id 冲突；只开旧 bundle 暂时也能跑 |
| Web Fetch 不再拒绝非公网地址 | outbound-proxy 的初始动机（fake-IP DNS 被 SSRF 拦）消失 |
| schedule 存储 v1→v2（首次 v2 写入后 v1 程序拒开） | 想回滚必须提前备份 `schedule.json` |
| session 格式仍是 **v4**（无 v5） | 会话在 rc.5↔0.2.2 双向可读，升级/回滚都不丢 |
| 插件页支持「更新第三方插件 + 失败自动回滚」 | 台风后置项（见 §3-B） |

---

## 1. 升级前（不做完不要点更新）

1. **完全退出 DSH**（托盘 + 后台进程都确认关闭）。
2. **抢救 rc.5 安装包**（updater 缓存会被 0.2.2 覆盖，这是唯一可靠的回滚源）：
   ```powershell
   $kit = "$HOME\Desktop\dsh-rollback-pre-0.2.2"
   New-Item -ItemType Directory -Path "$kit\installers" -Force | Out-Null
   Copy-Item -LiteralPath "$env:LOCALAPPDATA\@deepseek-aidsh-desktop-updater\pending\deepseek-harness-0.2.1-rc.5-win-x64.exe" -Destination "$kit\installers\" -Force
   # 校验与 update-info.json 的 sha512 一致（该文件同样在 pending 目录里）
   Get-Content "$env:LOCALAPPDATA\@deepseek-aidsh-desktop-updater\pending\update-info.json"
   # 若 pending 里没有 rc.5 安装包，说明这台设备没有可回滚源——把风险记下来再继续
   ```
3. **快照 profile 与关键数据**（体积都很小）：
   ```powershell
   New-Item -ItemType Directory -Path "$kit\profile-config","$kit\dsh-home-config" -Force | Out-Null
   foreach ($f in 'package.json','pnpm-lock.yaml','cordis.patch.yml','cordis.yml') {
     Copy-Item -LiteralPath "$HOME\.dsh\profiles\desktop\$f" -Destination "$kit\profile-config\" -Force }
   Copy-Item -LiteralPath "$HOME\.dsh\.credentials.yaml"        -Destination "$kit\dsh-home-config\" -Force
   Copy-Item -LiteralPath "$HOME\.dsh\storages\schedule.json"   -Destination "$kit\dsh-home-config\" -Force   # v1，回滚关键
   Copy-Item -LiteralPath "$HOME\.dsh\storages\workspace.json"  -Destination "$kit\dsh-home-config\" -Force
   ```
4. **记录现状**：`dsh plugin --profile desktop list` 的输出存一份；对每个 file:/link: 插件记下版本（之后对账）。
5. **只属于该设备的覆盖包**（如同名 `@deepseek-ai/dsh-llm-pi-ai` fork）：确认其生成基座版本（看仓库 `package.json` 的 version）。**不要在 rc.5 期间预替换 0.2.2 版适配器**——rc.5 的 engines 预检会把 `engines.dsh ^0.2.2-rc.1` 的包在启动时禁用（app 能开但 pi-ai 模型全失效）。升级后重新生成（§3-A）。
6. 确认无 `DSH_TOOLS_MODE` 残留（系统/用户环境变量、`~/.dsh/.env`）；0.2.2 已删除该开关，PTC 会经 preset 自动迁移。

---

## 2. 升级

1. 启动 DSH → 检查更新 → 下载 0.2.2-rc.1 → 重启安装（或直接跑 feed 里的安装包）。
2. 装完启动，登录态/会话/工作区自动延续（session 格式 v4 双向通用）。

---

## 3. 升级后（必做）

### A. 同名覆盖适配包（provider-aliases 一类）——放置在升级完成之后

```powershell
cd <provider-aliases 仓库>
git pull                      # 本仓库 main 已有 0.2.2 适配器（commit 88b9e8a 起）
node tools\sync.mjs           # 从“已安装的 0.2.2 asar”重新生成 lib/index.js + package.json
node --test tests\manifest.test.mjs   # 7/7 应通过
```
- node 用 DSH 自带的：`<DSH 安装目录>\resources\runtime\primary-runtime\dependencies\node\bin\node.exe`
- 11 个 alias patch 应全部命中；若报 "expected exactly one anchor"，说明 Desktop asar 与已同步版本又漂了——把报错片段发回主设备。
- link: 安装无需重装；跑完 sync 重启一次 DSH 即权威化。

### B. 第三方插件更新（必须后置）

在 插件页 → 已安装插件 → 各插件详情 → 「更新到 x.y.z」（0.2.2 新功能，失败自动回滚旧版）：
- `dsh-edit-turn` → 0.5.18（rc.5 上 0.5.14+ 也能跑，但统一升）
- `dsh-rerun-turn` → 0.2.11（**必须**：旧版 0.2.0 在 0.2.2 有已知缺陷；而 0.2.11 依赖 0.2.2 新 API，不能在 rc.5 上预先升）
- `dsh-computer-use-win` → 最新（≥0.5.4）

### C. profile 微调（二选一的事，都只改 `~/.dsh/profiles/desktop/cordis.patch.yml`）

1. 保留“先读后写”守卫（推荐，主设备已用并验证）：
   ```yaml
   - id: fs-observation-policy
     disabled: false
   ```
2. 「Git 工作树」bundle：未启用 forge 时**单独保留能跑**；要新面板就删 `@deepseek-ai/dsh-experimental-tool-worktree` 并加 `@deepseek-ai/dsh-experimental-forge-bundle`（两者同开会插入重复 id）。

### D. 这台设备若无任何自定义/覆盖插件

A 略过；B 照做；C.1 可选。到此即可正常使用。

---

## 4. 验证清单

- ``dsh plugin --profile <名> list``：包数与你记录的一致，无报错。
- 发一句话能走模型（适配器存活）。
- 守卫探针：对任意未读文件执行 write/edit → 应被 `file has not been read / FS_NOT_OBSERVED` 拒绝（若按 C.1 加了 patch）。
- 插件页「已安装插件」显示各包新版本；`schedule` 存储自动升 v2（正常现象）。
- （装了 session-bridge 0.9.0 的设备）空闲 30s 后全文检索可用；有活跃队友写作时可能偶发 `SESSION_QUERY_PERSISTENCE_FAILED`——重试即成功，属已知限制，非故障。

---

## 5. 回滚

1. 完全退出 DSH。
2. 运行 `$kit\installers\deepseek-harness-0.2.1-rc.5-win-x64.exe` 装回 rc.5。
3. 需要时把 `$kit\profile-config\*` 覆盖回 `~/.dsh/profiles/desktop/`；`$kit\dsh-home-config\schedule.json` 覆盖回 `~/.dsh/storages/`（**v2→v1 必须靠它**，否则 rc.5 打不开任务库）。
4. 覆盖包若已升到 0.2.2 生成版本：`git checkout` 或备份恢复 rc.5 生成件（本仓库 `rollback-rc5` 说明；rc.5 包在 0.2.2 下能加载，反之受伤的是 engines 预检——所以包一旦升了 0.2.2 版，回滚后必须恢复 rc.5 生成件）。
5. 启动；session v4 双向通用，聊天记录无恙。

---

## 6. 顺带的行为差异（不影响正确性，知道即可）

- 思考内容展示重构：slot `conversation.chat.reasoning.body` 的 props 变 `{rawText, text(display), running}`（本生态无插件注册该 slot）。
- MCP 资源能力守卫：未声明 `resources` capability 的 server 不再返回资源列表（tools/list 不受影响）。
- 统一右键菜单（新 `ctx.contextMenus` + `contextMenu.icon/content`）、文件预览目录树、聊天引用插入等新能力，插件可选择性接入。
- 插件市场更名「自定义插件索引」；0.2.2 尚不能自动更新本地 file:/link: 包。
- schedule 模型工具改为 RRULE+target 四件套（`schedule_create/list/view/edit`，ogs 更新过的脚本无需变更——主设备旧调度照跑）。

---

## 7. 证据索引（主设备验证记录，可对照）

- `research/dsh-022-delta-2026-11-06.md`（rc.5→0.2.2 全量契约 delta，npm 316 包逐文件 sha256）
- `research/plugin-install-audit-2026-10-30.md` / `dsh-rc5-plugin-audit-2026-10-30.md`（升级前基线审计）
- `research/session-bridge-022-compat-2026-11-06.md` / `session-bridge-022-rebuild-2026-11-06.md`（0.9.0 迁移与真机验收）
- `research/provider-aliases-022-sync-2026-11-06.md`（适配器预生成/权威化）
- `research/update-022-playbook-2026-11-06.md`（本指南的操作底稿）
- GitHub：`dsh-provider-aliases@88b9e8a`（0.2.2 适配器）、`dsh-session-bridge@be5e120`（0.9.0）、索引 `dsh-plugins` 全程同步
