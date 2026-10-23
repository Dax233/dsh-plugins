# 浏览器控制插件 P1 设计与 backlog（2026-10-23）

状态：P0（安全加固、侧栏快照/settle/回执、chrome Host 侧 refs/settle/回执/logs/upload/evaluate/wait）已实现并真机验证；本文件记录剩余 P1 的落地设计与顺序，下一轮按此实现。

## 1. iframe 支持（chrome）

现状：AX 快照只保留主 frame 节点（`Page.getFrameTree` + `DOM.describeNode.frameId` 过滤），子 frame 内容不可见、不可点。

设计：
1. `axSnapshot` 不再丢弃子 frame 节点：候选记录 `frameId`（`DOM.describeNode` 已返回）；输出元素附加 `frame` 字段（仅非主 frame 时）。
2. 构建 frame 偏移表：`Page.getFrameTree` 递归遍历 frame 树；对每个子 frame：
   - `DOM.getFrameOwner({frameId})` → owner backendNodeId；
   - `DOM.pushNodesByBackendIdsToFrontend` → nodeId；`DOM.getBoxModel` → iframe 元素 content quad；
   - 偏移 = 父 frame 偏移 + iframe content quad 左上角（嵌套递归）。
3. `axBox` 取节点自身 box（相对其 frame viewport）后加上该 frame 的偏移，得到主 frame 视口坐标再 `Input.dispatchMouseEvent`。
4. 同进程 iframe（same-process）可走上述路径；OOPIF（跨进程）下 `DOM.pushNodesByBackendIdsToFrontend` 在 tab target 可能失败：失败时把节点标记 `crossOrigin: true`，输出为不可点占位（role iframe / link 文本），提示模型用 `evaluate` 或切换到该 frame URL。
5. 测试：mock `Page.getFrameTree`、`DOM.getFrameOwner`、`DOM.getBoxModel`，断言子 frame 节点 ref/偏移/点击坐标；列一个 OOPIF 失败回退用例。
6. 真机：用 `https://www.w3schools.com/html/html_iframe.asp`（同源 iframe）验证 snapshot 含子 frame 元素且点击成功；再用跨域 iframe 页面验证占位与不报错。

## 2. tab group ACL（chrome）

现状：`newTab`/`attach` 会把 tab 放进 "DSH Agent" 组，但任何 tabId 都能直接操作，没有组约束。

设计：
1. `agentGroupId` 在 `ensureAgentGroup` 已维护；新增 `assertAgentTab(tabId)`：
   - `chrome.tabs.get(tabId)` 取 `groupId`；
   - 若 `groupId !== agentGroupId`，抛错 `tab <id> is not in the DSH Agent group; call attach first to opt it in`。
2. 在 `snapshot/click/type/key/scroll/logs/upload/evaluate/wait/navigate/captureScreenshot` 入口调用（`attach`/`newTab` 除外）。
3. `attach` 语义不变：显式把用户 tab 移入 DSH Agent 组（这是"同意被操作"的动作）；用户把 tab 拖出组即撤销。
4. `tabs` 列表保持全部可见，但每行标注 `(agent)` / `(not in agent group)`，并提示 attach 才能操作。
5. 测试：mock `tabs.get` 返回 groupId（agent / 其他 / -1）；断言未 attach 的操作报错、attach 后通过、detach 后回到拒绝。
6. 风险：现有测试大量直接操作 tab 1，需要补 `attach` 调用；live 用户若希望直接操作当前 tab，需先 attach（已在工具描述中说明）。

## 3. 下载（chrome）

现状：无下载动作；页面触发的下载无法被 agent 感知。

设计：
1. 新增 `downloads {limit?}`：用 `chrome.downloads.search`（manifest 加 `downloads` 权限）列出最近下载（filename/url/state/bytes）。
2. 新增 `downloadUrl {url, filename?}`：`chrome.downloads.download({url, filename})`，返回 downloadId；用于 agent 明确下载资源。
3. 页面自然下载：扩展 `chrome.downloads.onCreated` 事件记录到 controller 的 per-tab/全局环缓冲，并作为 `click` 回执附加 `downloads: [{id, filename, state}]`（需要在 click 后短轮询 500ms）。
4. 安全：只允许 http(s)；文件名限制在工作区/下载目录；不在 tool 结果里回传文件内容。
5. 权限变更需要重新加载扩展并可能出现 Chrome 权限提示；版本升级时在 README 说明。
6. 测试：mock `chrome.downloads` 的 search/download/onCreated；断言列表/下载/点击回执。

## 4. 随机配对 token（安全可选项）

现状：两个桥的 token 仍是默认常量（侧栏 Bearer / chrome hello），loopback + Host/Origin 校验已就位。

设计（chrome）：
1. Host 启动时生成随机 token（`crypto.randomBytes`），写入 `%TEMP%\dsh-chrome-bridge-token`（0600/隐藏），诊断状态只写 hash。
2. 扩展 options 增加"配对"流程：扩展 `GET http://127.0.0.1:47835/pair`（校验 Host + Origin `chrome-extension://…`）读取 token 存 `chrome.storage.local`；之后 hello 携带该 token。
3. 默认 token 保留为显式 fallback（环境变量 `DSH_CHROME_BRIDGE_TOKEN`），README 标注风险。
4. 侧栏同理：Host 生成 token → 渲染器 `GET /bootstrap`（校验 Host + Origin `dsh-app://app`）取 token，之后 Bearer；随窗口生命周期轮换。
5. 测试：配对端点 Origin/Host 校验、token 轮换后旧连接失效、无 Origin 的本地工具走 header token 仍可用。

## 顺序建议

1. iframe 支持（用户可见收益最大）
2. tab group ACL（安全语义）
3. 下载
4. 随机配对 token（安全增强，可选）

每项仍按测试先行：先写 controller/host 测试 → 实现 → `npm test` 全绿 → 扩展/Host 重装 → computer-use 刷新扩展 → 真机验证 → 文档/提交。
