# 本地 API 契约

两个本地服务，均**只监听 127.0.0.1**，且校验 Host 头（防 DNS rebinding）。

## eni-launcher（127.0.0.1:8444）—— 客户端界面服务

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/` | 按账号状态返回 登录页 / 激活页 / 主界面（主界面仅已激活账号可得） |
| GET | `/api/alive` | SSE：窗口存活通道 + 服务端状态推送（`event: state`） |
| GET | `/api/status` | `{state, activated, status?, proxy?, platforms?}`（未激活只返回 state） |
| POST | `/api/login` | `{email,password}` → 转发服务端 account_login，本地存 token |
| POST | `/api/register` | `{email,password}` → 转发 register |
| POST | `/api/activate` | `{key}` → 服务端 laok_activate（token + 卡 + 机器码） |
| POST | `/api/logout` | 清本地凭据 |
| POST | `/api/platform/activate` | `{id}` → 平台注入（id: zcode/dsh/codex/workbuddy/hermes） |
| POST | `/api/platform/restore` | `{id}` → 从干净备份还原 + 写 disabled 标记 |
| POST | `/api/restart` | 重启本地代理 |
| POST | `/api/quit` | 托盘退出用：杀代理并退出（= 功能失效） |
| POST | `/api/shop` | 返回购买链接 |

## eni-proxy（127.0.0.1:8443）—— 协议代理

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/healthz` | `{ok, version, eni, pid, ts}` |
| POST | `/v1/chat/completions` | OpenAI chat 协议透传（流式/非流式），前置闸门校验 |
| POST | `/v1/responses` | OpenAI responses 协议透传（Codex 形态） |

**闸门语义**：代理每 5 分钟向服务端心跳验卡；闸门未通过时所有 AI 请求返回
`402` + `eni_license_required`（含购买引导），`healthz` 不受影响。

## 服务端 API（HTTPS，响应带 Ed25519 签名包络）

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | `/register` | `{email,password}` → 注册（限速 5/时/IP） |
| POST | `/account_login` | 登录；同账号新登录会作废旧会话（一号一机） |
| POST | `/laok_account_status` | `{token}` → 该账号授权状态（activated/days_left/expires_at） |
| POST | `/laok_activate` | `{token,key,hwid}` → 卡密绑定到账号（机器码只记录不参与判定） |
| POST | `/laok_heartbeat` | `{token,key,hwid}` → 持续心跳；session-invalid = 该账号已在其他设备登录 |
| POST | `/account_logout` | 作废会话 |
| GET | `/pubkey` | 签名公钥（客户端首次锁定） |

### 签名包络

```
sig = Ed25519-Sign( priv, json.dumps(body, sort_keys=True,
                                      separators=(',',':'), ensure_ascii=True) )
```

客户端按同一规则重建规范 JSON 后验签（数字必须为整数，浮点会导致 JS/Python 序列化差异）。

### 状态机（客户端闸门）

```
无 token / 无卡          → 402 引导
activated / active       → 放行（缓存 last_ok）
session-invalid          → 402「账号已在其他设备登录」
expired                  → 402「已到期续费」
key-mismatch             → 402「本地卡与账号绑定不一致，重新激活」
网络故障 & last_ok<24h    → 宽限放行
网络故障 & 超过 24h       → 402
```
