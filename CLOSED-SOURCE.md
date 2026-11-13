# 闭源说明

本仓库是 **ENI Framework 的架构与契约层**。以下部分为闭源发行内容，不在本仓库：

| 闭源组件 | 原因 |
|---|---|
| `eni-core.js`（上下文注入核心） | 核心引擎 |
| `ENI.md`（人格规范） | 核心资产 |
| `etc/skills/`（技能库） | 核心资产 |
| `eni-proxy.js` / `eni-launcher.js` / `eni-license.js` / `eni-verify.js` / `eni-platforms.js` / `eni-guard.js` 实现 | 完整实现（本仓库提供接口契约） |
| `EniLauncher.cs`（托盘宿主源码） | 完整实现 |
| 授权服务端（license server） | 完整实现（本仓库提供 API 契约） |
| 安装 payload 与构建产物 | 发行内容 |

接口契约与行为语义在 `docs/` 与 `server/` 下完整公开：
任何语言都可以按契约重新实现等价组件。

获取官方发行版：QQ 群 **1124698128**
