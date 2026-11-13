# 五平台适配点

每个平台一个适配器：`{ id, name, files[], status(), activate(), restore() }`。
激活 = 备份原始配置（仅原始态）→ 修改接入本地代理；还原 = 干净备份回滚或反向改写。

| 平台 | 配置文件 | 激活改动 | 还原方式 |
|---|---|---|---|
| ZCode | `.zcode/v2/provider_config.json` | `config.api.baseUrl → 127.0.0.1:8443` | 备份回滚 / baseUrl 反向改写 |
| DSH | `.dsh/profiles/{desktop,headless}/cordis.patch.yml` | `baseURL: → 本地代理` | 备份回滚 / baseURL 反向改写 |
| Codex | `.codex/config.toml` | `[model_providers.*].base_url` + `model_provider` + instructions 指向人格文件 + developer_instructions 全文 | 备份回滚 / base_url 反向改写（人格段需备份才可完整撤销） |
| WorkBuddy | `.workbuddy/MEMORY.md` | 追加/替换人格段 | 备份回滚 / 截断人格段 |
| Hermes | `AppData/Local/hermes/config.yaml` + `SOUL.md` | `model.provider/base_url/default` + providers 新增本地条目 + SOUL 人格 | 备份回滚 |

## 备份语义（关键设计）

- **只有"未注入"的文件才允许生成备份**。如果文件里已含本地代理地址，说明它已被改过——
  此刻备份下来的是注入态快照，"还原"会还原回注入态，等于没有还原。这种文件一律不备份，
  还原走反向改写（把地址改回官方值）。
- 还原时再次校验备份"干净度"（不含本地代理地址），脏备份不参与还原。

## 还原尊重用户选择

- 用户点过「还原」的平台 → `etc/disabled/<id>` 写标记 → 后台守护（guard）不再自动改回该平台的配置；
- 再次「激活」→ 删除标记，守护恢复看护。

## 客户端重装自愈

| 场景 | 检测 | 行为 |
|---|---|---|
| 配置文件被删（卸载重装带走配置） | 文件不存在 | 从快照还原（快照定期更新，始终是最近一次健康注入态） |
| 配置被客户端重置成默认 | ENI 标记缺失 | 只补 ENI 相关键，保留用户其它改动（MCP/插件/approval 等） |
| baseUrl 被改回官方 | 字符串检测 | 改回本地代理 |
