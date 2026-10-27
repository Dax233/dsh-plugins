# DSH Plugins

DeepSeek Harness（DSH）插件目录。每个插件维护在独立仓库中；本仓库只做索引、安装说明与设计笔记，不再包含插件源码。

## 插件

| 插件 | 说明 | 安装 |
| --- | --- | --- |
| [Outbound Proxy](https://github.com/Dax233/dsh-outbound-proxy) | 让 DSH 出站 HTTP(S) 走本机代理（Clash / mihomo 等），端口跟随启动环境，插件行可单独覆盖 | `dsh plugin --profile desktop add github:Dax233/dsh-outbound-proxy` |
| [Desktop Notify](https://github.com/Dax233/dsh-desktop-notify) | Windows 原生通知：任务完成 / 需要提问 / 出错时弹 toast 并播放内置 ciallo 提示音 | `dsh plugin --profile desktop add github:Dax233/dsh-desktop-notify` |
| [Sidebar Browser Control](https://github.com/Dax233/dsh-sidebar-browser-control) | 侧边栏 Agent 浏览器（0.2.9）：快照 / 点击 / 输入 / 滚动 / 截图，加 QA 验收面——`viewport` 真实 CSS 视口（含滚动条开关/contentWidth）、`emulate` 主题（含 `<picture>`/`<link media>` DOM 兜底）、`evaluate`/`metrics`/`logs`、整页/超长元素/clip 截图、`qa` 批量矩阵、`preview` 本地预览；稳定 tab + 旧黑标签清理、guest 容器迁移、attach 失败重建 webview；页面内虚拟光标不碰真实鼠标 | `dsh plugin --profile desktop add github:Dax233/dsh-sidebar-browser-control` |
| [Chrome Bridge](https://github.com/Dax233/dsh-chrome-bridge) | 控制真实 Chrome（0.1.30）：后台标签、DSH Agent 标签组 ACL、UIA refs、动作回执、console/network 日志、iframe 观测与点击；CDP 视口/配色仿真、fullPage/clip/元素截图、networkIdle best-effort（忽略长连接类型）、命令串行化 | `dsh plugin --profile desktop add github:Dax233/dsh-chrome-bridge` |
| [Codex Bridge](https://github.com/Dax233/dsh-codex-bridge) | 桥接本机 Codex：线程清单 / 转录取读 / 无头投递任务 / goal 与活性监测（可对停滞线程补"继续"）；仓库另带离线 watch 告警脚本 | `dsh plugin --profile desktop add github:Dax233/dsh-codex-bridge` |
| [Delete Guard](https://github.com/Dax233/dsh-delete-guard) | 删除指令守卫（0.2.0）：shell 命令里出现删除动词时必须是唯一且顶层的单条语句；串联、管道、嵌套、`cmd /c`、`iex` 拒绝；`run_code` 内文件删除与 `node -e`/`python -c` 等解释器包装同样拦截 | `dsh plugin --profile desktop add github:Dax233/dsh-delete-guard` |
| [Provider Aliases](https://github.com/Dax233/dsh-provider-aliases) | 模型目录别名：一个内置 provider 目录挂多条凭据路由（多 OpenCode Go 套餐等），附「模型别名」设置面板，每条路由独立 Key 与思维档位 | `dsh plugin --profile desktop add github:Dax233/dsh-provider-aliases` |
| [Session Bridge (fork)](https://github.com/Dax233/dsh-session-bridge) | 跨工作区会话桥 fork（上游 MIT，0.8.1）：按标题/工作区查找、读取任意会话，新增 `session_search_global` 全库正文检索；resume 会补齐模型路由（避免 `{{model}}` 装配错误，Cordis-safe）；工具面可切换（当前开放建会话/发消息/监控/归档等写操作），bundle patch 开启持久化全文索引 | `dsh plugin --profile desktop add github:Dax233/dsh-session-bridge` |
| [Preset Hotswitch](https://github.com/Dax233/dsh-preset-hotswitch) | 会话内热切换 Agent 预设：绕过官方「仅空白会话可切换」限制（`agentPresets.recompose` + `agent-preset/selected`），下一请求生效、重启保持 | `dsh plugin --profile desktop add github:Dax233/dsh-preset-hotswitch` |

## 安装

前置：DSH Desktop 带插件管理器（0.2.x），Windows 10/11。

```sh
dsh plugin --profile desktop add github:Dax233/dsh-outbound-proxy
dsh plugin --profile desktop add github:Dax233/dsh-desktop-notify
dsh plugin --profile desktop add github:Dax233/dsh-sidebar-browser-control
dsh plugin --profile desktop add github:Dax233/dsh-chrome-bridge
dsh plugin --profile desktop add github:Dax233/dsh-codex-bridge
dsh plugin --profile desktop add github:Dax233/dsh-provider-aliases
dsh plugin --profile desktop add github:Dax233/dsh-delete-guard
dsh plugin --profile desktop add github:Dax233/dsh-session-bridge
dsh plugin --profile desktop add github:Dax233/dsh-preset-hotswitch
```

也可以直接在插件页添加对应 GitHub 地址；安装后重启 DSH Desktop。

### Chrome Bridge 额外步骤

下载现成扩展包：[chrome-bridge 最新 Release](https://github.com/Dax233/dsh-chrome-bridge/releases/latest) 的 `dsh-chrome-bridge-v*.zip`，解压后在 `chrome://extensions` 开启开发者模式并 **加载已解压的扩展程序**。

> `.crx` 是自签名包，Chrome 稳定版不允许拖入安装（`CRX_REQUIRED_PROOF_MISSING`），仅用于企业策略/Chromium 开发版；日常安装请用 zip + Load unpacked。


1. 在 Chrome 打开 `chrome://extensions` → 开启开发者模式。
2. **加载已解压的扩展程序** → 选择 `dsh-chrome-bridge` 仓库的 `extensions/chrome-extension` 目录。
3. 打开扩展选项页，确认端口（默认 `47835`）与 token（默认 `dsh-chrome-bridge-local`）。

只有 **DSH Agent 标签组** 内的标签页可以被驱动；`newTab` 会自动入组，`attach` 是显式授权（把标签页拖出组即撤销）。

### 浏览器 QA（验收）能力

两套浏览器插件分工明确：

- **Sidebar Browser Control（0.2.9+）**：侧栏里用户可见的 QA renderer。`browser qa {url,widths,themes}`
  一次跑 width×theme 矩阵，输出每组截图路径 + metrics 表；`viewport` 改真实 guest CSS 视口
  （面板内等比缩放显示，`scrollbars:"hidden"` 可做像素级对齐并回执 contentWidth）、
  `emulate {colorScheme}` best-effort 主题（CSS 规则 + matchMedia shim + `<picture>`/`<link media>`
  DOM 兜底，含 open shadowRoot）、`evaluate`/`metrics`/`logs` 读布局/资源/console、
  `screenshot {fullPage,clip,ref,selector,format,path}` 出证据（fullPage 逐屏拼接、隐藏滚动条、
  阶段诊断与 timings、超长元素整幅拼接、真实像素回执）、`setContent`/`preview` 处理自包含 HTML 与本地目录。
  面板 tab 身份稳定（重载复用同一个 tab，启动清理旧版本残留的黑标签），guest attach 带 3 次
  re-append 重试 + 最后「关掉重开 tab」兜底。
- **Chrome Bridge（0.1.30+）**：真实 Chrome 标签页上的 CDP 仿真：`emulate
  {width,height,deviceScaleFactor,mobile,colorScheme}`、`screenshot {fullPage,clip,ref,format,quality}`
  （fullPage 临时拉高仿真视口再截，不用会挂死的 captureBeyondViewport）、`wait {networkIdle}`
  （忽略 WebSocket/EventSource/Ping 等长连接类型，超时 best-effort 返回 ok + warning）；
  扩展端命令串行化；会临时改变该标签显示，`reset`/`detach`/关标签自动复位，不是 CI/离屏方案。

**桌面壳边界（不是插件缺口）**：侧栏 webview 拿不到 CDP，所以真 DPR/媒体仿真与 HTTP 状态码不可用
（回执如实报真实 DPR/像素尺寸）；壳只放行 http(s) 导航并取消 `file:`，所以 `file://` 不可用
（用 `setContent` 或 `preview` 代替）；插件也没有 Electron 主进程权限，不做离屏/CI。需要真 DPR 的
验收走 Chrome Bridge 或外部 CDP。

### Codex Bridge 说明

需要本机已安装并登录 Codex（桌面版自带 CLI，或 `npm i -g @openai/codex`；登录态在 `CODEX_HOME` 下的 `auth.json`）。

插件只**读** Codex home（转录、goal 数据库、写者锁、automation 文件），不修改 Codex 会话、goal 或配置；驱动 turn 走 `codex exec` / `codex exec resume` 子进程，子进程不继承凭据类环境变量。无头执行没有人工审批通道，默认 `approval_policy=never`；`codex_liveness` 的自动补活默认关闭，需要配置 `allowNudge: true` 才生效。

仓库另带 `scripts/watch.mjs`：确定性的 relay 异常检查（rollout heartbeat / goals / automation / liveness），只输出告警，退出码 `0` 无告警 / `10` 有告警 / `2` 检查失败。它不持有定时器、不唤醒 DSH，需用 `--thread` 或 `watch.config.json` 指定线程（无内置默认值），再配合 DSH `schedule_create` 或任务计划实现"异常才介入"。

### Delete Guard 说明

插件挂在 `tools/pre-execute` 上：命令中出现删除动词（`Remove-Item`、`rm`、`del`、`rd`、
`[IO.File]::Delete` 等）时，只有**唯一且顶层的一条语句**才放行，其余组合在调用前被拒绝并返回原因。
0.2.0 起：`run_code` 代码文本里的文件删除 API 同样被拒绝（指引用独立 shell 删除），
`node -e` / `python -c` / `perl -e` / `ruby -e` / `php -r` / `deno eval` / `bun -e` 这类
「把删除藏在被求值字面量里」的包装也会被拒。

- 引号内字面量先掩码，路径里的 `a;b` 不会被误判成串联；但 `iex "..."`、解释器 `-c/-e` 这类会求值的场景会回查原文。
- 分析异常时 fail-open（放行），守卫自身出问题不会把会话卡死。
- `write`/`edit` 的覆盖写不在本插件范围（属编辑语义/文件观察策略层，没有「独立语句」等价物）。

### Provider Aliases 说明

一个包同时提供两半：同名扩展的内置 `@deepseek-ai/dsh-llm-pi-ai` 适配器（新增
`catalog` 别名能力）与「设置 → 模型别名」面板。安装后**重启 DSH Desktop**：

```sh
dsh plugin --profile desktop add github:Dax233/dsh-provider-aliases
```

它覆盖的适配器编译自 DSH 0.2.1-rc.3；DSH 升级后需要等待仓库更新或按仓库 README
重跑 `tools/sync.mjs` 重新生成。

### Preset Hotswitch 说明

插件注册一个 Host 工具 `switch_session_preset`：官方切换入口只允许**空白会话**使用，
已开始的会话会被 registry 以 `agent-preset/locked` 拒绝；本插件改走公开的
`agentPresets.recompose()` 并写入 `agent-preset/selected` 事件，下一请求生效、重启后保持。
只作用于**调用该工具的会话**，不创建/删除预设；依赖宿主的 `agentPresets` 服务。
`recompose` 的契约自述"由调用方负责空白检查"，本插件刻意跳过该检查，属**实验性**用法，
DSH 升级后可能失效。

## 安全说明

- 浏览器控制插件的本机桥只监听 `127.0.0.1`（默认端口 `47834` / `47835`）。
- 鉴权：侧栏桥使用 `Authorization: Bearer`；Chrome 桥使用 WebSocket `hello` 帧 token；两者都校验 `Host`/`Origin`。
- 默认 token 仅用于本机零配置配对，可在扩展选项 / 环境变量中覆盖；不要把本机桥端口暴露到公网。
- 本目录仓库不包含密钥、代理凭据或个人路径。

## 设计笔记

本目录的 `research/`（同类方案调研、反馈分级、根因与真机验证记录）**只保留在本地**，不纳入
版本控制、不推送远端：里面会引用设备诊断、会话 id 与本机路径，公开会泄露本机信息。
仓库只发布插件索引、安装说明、平台边界与操作规范。

## DSH 平台坑（本轮实测，建议上游修）

1. **`dsh plugin add link:<path>` 不更新 `dsh.profile.bundles`**：只写 dependencies，插件静默不加载。
   `file:` 安装会更新 bundle，`link:` 不会 → 装完必须核对 `dsh.profile.bundles`（本仓库脚本/文档已注明）。
2. **运行中对 Host 插件 `remove`+`add` 会重复注册**：旧实例未卸载、新实例热加载 →
   `Agent resolver 已经注册` 启动失败；需重启清状态。
3. **运行中编辑 profile 的 `package.json` 会触发热重载**：服务图重建期间可能出现
   `typert gateway: session/follow: active Service "sessionController" is unavailable`；需重启恢复。
4. 结论：profile/插件安装变更一律「先改好 → 一次性重启」，运行中不 remove/add Host 插件、不编辑 profile。

## 开发

每个插件仓库独立开发与测试：

```sh
git clone https://github.com/Dax233/dsh-<plugin>.git
cd dsh-<plugin>
npm test
```

### 本机 profile 更换本地插件版本

桌面 profile 里的 `@local` 插件是 `file:` **复制安装**（不是软链）：改了仓库文件不会自动进安装副本，
同路径重跑 `dsh plugin add file:F:/github/<repo>` 只会输出 `Already up to date`。刷新方式：

```powershell
$dsh = 'G:\programs\DSH\resources\runtime\cli\bin\dsh.cmd'
& $dsh plugin --profile desktop remove @local/<pkg>
& $dsh plugin --profile desktop add file:F:/github/<repo>
# 然后重启 DSH Desktop；Host 插件与 client bundle 都只在启动时加载
```

`file:` 依赖的「内容变了但路径没变」不会触发 pnpm 重装，这是它按目录路径标识包的行为，不是插件问题。

## License

MIT
