# 浏览器控制插件对标调研（2026-10-23）

范围：`sidebar-browser-control`（DSH 侧栏 Agent 浏览器）与 `chrome-bridge` + MV3 扩展（真实 Chrome 控制）。
方法：GitHub raw README/源码、npm registry 数据；`web_search` 不可用，未 clone/安装任何对标项目。结论是**不 fork**，只吸收设计与自研实现。

## 1. 侧栏 Agent 浏览器

| 项目 | 许可 | 观测 | 动作 | 可吸收点 |
|---|---|---|---|---|
| [browser-use](https://github.com/browser-use/browser-use) | MIT | DOM + a11y + DOMSnapshot 合并树，数字 refs，稳定哈希 | CDP 元素事件 | `data-browser-use-exclude` 排除自身 UI、新标签检测、空 DOM 重试、文本截断 100 |
| [Playwright MCP](https://github.com/microsoft/playwright-mcp) | Apache-2.0 | `ariaSnapshot({mode:'ai'})`，refs `e12`/`f1e5` | Locator/ref | 动作后 settle 500ms、模态框/文件选择器竞态、高亮/标注、iframe 前缀 refs |
| [Nanobrowser](https://github.com/nanobrowser/nanobrowser) | Apache-2.0 | DOM + 截图 | 真实输入 | 动作前可见/遮挡/可点击检查、ref 绑定元素、Ask/Auto + 站点权限 |
| [Stagehand](https://github.com/browserbase/stagehand) | MIT | a11y 裁剪 | `act/observe/extract` | 深层 iframe / closed shadow DOM locator、自愈选择器 |
| BrowserOS / Skyvern | AGPL-3.0 | 各异 | — | 仅借鉴 UX（看板/回放/validate），不复制代码 |

落地优先级（sidebar P0/P1）：

1. 缩进树快照（已完成，commit `487ca8c`）
2. 稳定 refs + 明确 stale 错误（已完成）
3. 自身 overlay 排除 + 截断提示（已完成）
4. 动作 settle：导航等待 + MutationObserver 静默（约 100ms quiet，3s 上限）（待做）
5. 动作回执：URL/title/scroll/console 错误增量/期望文本（待做）
6. P1：点击后新标签检测、对话框/文件选择器/下载状态化、iframe refs、光标高亮/缓动、按站点权限、密码框脱敏、大页面 find/search

## 2. 真实 Chrome 控制

| 项目 | 许可 | 传输 | 亮点 |
|---|---|---|---|
| [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) | Apache-2.0 | Puppeteer/CDP，可连运行中 Chrome | a11y uid 由 Host 侧 `(loaderId, backendNodeId)` 维护；network/console/perf trace；settle 协议 |
| [Playwright MCP](https://github.com/microsoft/playwright-mcp) + Extension | Apache-2.0 | 扩展 relay / CDP | 每客户端一个 tab group；审批 + per-profile token；origin 白名单；文件上传 |
| [Browser MCP](https://github.com/BrowserMCP/mcp) | Apache-2.0 | 固定端口 WS | 每个动作后自动返回新快照；**无鉴权/Origin 校验**（反面教材） |
| Chrome MCP Server | MIT | MV3 nativeMessaging | webRequest + debugger body；语义化标签搜索 |
| [browser-harness](https://github.com/browser-use/browser-harness) | MIT | CDP + `chrome://inspect` 授权 | 无扩展接入；健康检查 |
| BrowserTools MCP v2 | MIT | loopback + 扩展 | loopback/Origin/per-run bearer/脱敏加固模板；1.2.x 曾有 CVSS 9.8 WS RCE |

落地优先级（chrome P0/P1）：

1. 桥安全：Host/Origin 校验、token 移到 hello、timingSafeEqual（已完成代码，扩展待刷新复验）
2. Host 侧 refs：`(loaderId, backendNodeId)`，不把 ref 放页面；缺失 ref 明确报错（待做）
3. settle 协议：预挂 `Page.frameStartedNavigating` + MutationObserver 静默 + dialog/popup 捕获（待做）
4. 动作后返回新快照/回执（待做）
5. P1：tab group ACL、审批/接管 UX、console/network 环形缓冲 + 脱敏、iframe、上传/下载、`select_option`/`drag`/`pdf`/`evaluate`、截图预算

## 3. 决策记录

- **不 fork**：架构（Python/独立 MCP/Chromium fork）与 DSH 侧栏/扩展集成不匹配；依赖与维护成本高；现有实现已有 DSH 专属能力与测试基线。
- **可选并行**：network/console/perf 可另装 chrome-devtools-mcp 作为 MCP 服务，不替换自有插件。
- **许可**：参考实现为 MIT/Apache-2.0，可吸收思想；若复制代码需保留版权/NOTICE 并兼容本仓库 MIT。
