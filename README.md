# DSH Plugins

DeepSeek Harness（DSH）插件目录。每个插件维护在独立仓库中；本仓库只做索引、安装说明与设计笔记，不再包含插件源码。

## 插件

| 插件 | 说明 | 安装 |
| --- | --- | --- |
| [Outbound Proxy](https://github.com/Dax233/dsh-outbound-proxy) | 让 DSH 出站 HTTP(S) 走本机代理（Clash / mihomo 等），端口跟随启动环境，插件行可单独覆盖 | `dsh plugin --profile desktop add github:Dax233/dsh-outbound-proxy` |
| [Desktop Notify](https://github.com/Dax233/dsh-desktop-notify) | Windows 原生通知：任务完成 / 需要提问 / 出错时弹 toast 并播放内置 ciallo 提示音 | `dsh plugin --profile desktop add github:Dax233/dsh-desktop-notify` |
| [Sidebar Browser Control](https://github.com/Dax233/dsh-sidebar-browser-control) | 侧边栏 Agent 浏览器：快照 / 点击 / 输入 / 滚动 / 截图，页面内虚拟光标不碰真实鼠标 | `dsh plugin --profile desktop add github:Dax233/dsh-sidebar-browser-control` |
| [Chrome Bridge](https://github.com/Dax233/dsh-chrome-bridge) | 控制真实 Chrome：后台标签、DSH Agent 标签组 ACL、UIA refs、动作回执、console/network 日志、iframe 观测与点击 | `dsh plugin --profile desktop add github:Dax233/dsh-chrome-bridge` |
| [Codex Bridge](https://github.com/Dax233/dsh-codex-bridge) | 桥接本机 Codex：线程清单 / 转录取读 / 无头投递任务 / goal 与活性监测（可对停滞线程补"继续"） | `dsh plugin --profile desktop add github:Dax233/dsh-codex-bridge` |

## 安装

前置：DSH Desktop 带插件管理器（0.2.x），Windows 10/11。

```sh
dsh plugin --profile desktop add github:Dax233/dsh-outbound-proxy
dsh plugin --profile desktop add github:Dax233/dsh-desktop-notify
dsh plugin --profile desktop add github:Dax233/dsh-sidebar-browser-control
dsh plugin --profile desktop add github:Dax233/dsh-chrome-bridge
dsh plugin --profile desktop add github:Dax233/dsh-codex-bridge
```

也可以直接在插件页添加对应 GitHub 地址；安装后重启 DSH Desktop。

### Chrome Bridge 额外步骤

下载现成扩展包：[chrome-bridge 最新 Release](https://github.com/Dax233/dsh-chrome-bridge/releases/latest) 的 `dsh-chrome-bridge-v*.zip`，解压后在 `chrome://extensions` 开启开发者模式并 **加载已解压的扩展程序**。

> `.crx` 是自签名包，Chrome 稳定版不允许拖入安装（`CRX_REQUIRED_PROOF_MISSING`），仅用于企业策略/Chromium 开发版；日常安装请用 zip + Load unpacked。


1. 在 Chrome 打开 `chrome://extensions` → 开启开发者模式。
2. **加载已解压的扩展程序** → 选择 `dsh-chrome-bridge` 仓库的 `extensions/chrome-extension` 目录。
3. 打开扩展选项页，确认端口（默认 `47835`）与 token（默认 `dsh-chrome-bridge-local`）。

只有 **DSH Agent 标签组** 内的标签页可以被驱动；`newTab` 会自动入组，`attach` 是显式授权（把标签页拖出组即撤销）。

### Codex Bridge 说明

需要本机已安装并登录 Codex（桌面版自带 CLI，或 `npm i -g @openai/codex`；登录态在 `CODEX_HOME` 下的 `auth.json`）。

插件只**读** Codex home（转录、goal 数据库、写者锁、automation 文件），不修改 Codex 会话、goal 或配置；驱动 turn 走 `codex exec` / `codex exec resume` 子进程，子进程不继承凭据类环境变量。无头执行没有人工审批通道，默认 `approval_policy=never`；`codex_liveness` 的自动补活默认关闭，需要配置 `allowNudge: true` 才生效。

## 安全说明

- 浏览器控制插件的本机桥只监听 `127.0.0.1`（默认端口 `47834` / `47835`）。
- 鉴权：侧栏桥使用 `Authorization: Bearer`；Chrome 桥使用 WebSocket `hello` 帧 token；两者都校验 `Host`/`Origin`。
- 默认 token 仅用于本机零配置配对，可在扩展选项 / 环境变量中覆盖；不要把本机桥端口暴露到公网。
- 本目录仓库不包含密钥、代理凭据或个人路径。

## 设计笔记

- `research/browser-control-survey.md`：同类浏览器控制方案调研。
- `research/browser-control-backlog.md`：下载、随机配对 token、逐 frame target 等未做项。

## 开发

每个插件仓库独立开发与测试：

```sh
git clone https://github.com/Dax233/dsh-<plugin>.git
cd dsh-<plugin>
npm test
```

## License

MIT
