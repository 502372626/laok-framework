# 授权服务端 API 契约

授权服务端为闭源组件，此处只公开接口契约（供集成方/客户端参考）。
所有响应带 Ed25519 签名包络（见 docs/api-contracts.md）。

## 账号体系

| 端点 | 请求 | 响应要点 |
|---|---|---|
| `POST /register` | `{email, password}` | `{ok, email, token, status}`；限速 5/时/IP |
| `POST /account_login` | `{email, password}` | `{ok, token, status}`；**同账号新登录作废旧会话（一号一机）** |
| `POST /account_logout` | `{token}` | 作废会话 |

## 卡密（老K产品线）

| 端点 | 请求 | 响应要点 |
|---|---|---|
| `POST /laok_activate` | `{token, key, hwid}` | 卡密绑定到**账号**；机器码只记录不参与判定；一张卡只绑一个账号 |
| `POST /laok_account_status` | `{token}` | `{activated, status(active/no-license/expired), expires_at, days_left}` |
| `POST /laok_heartbeat` | `{token, key, hwid}` | 持续心跳；`session-invalid` = 该账号已在其他设备登录 |

### 状态语义

```
activated   首次激活成功（绑账号 + 计时开始）
active      心跳正常
no-license  账号未激活（登录了但没绑卡）
expired     已到期
session-invalid  一号一机：新设备登录后旧设备会话失效
key-mismatch    本机使用的卡与账号绑定不一致（重新激活即可）
```

### 授权口径（v1.0.3 起）

1. **卡密绑账号，不绑机器**——换设备直接登录同一账号，旧设备自动下线，无需换绑。
2. **单点由会话承担**：服务端会话表，新登录踢旧会话；旧客户端下次心跳拿到 `session-invalid` 自动下线。
3. 机器码仅用于排障记录。

## 签名与安全

- 响应签名：Ed25519，覆盖排序紧凑 JSON（`sort_keys + separators=(',',':') + ensure_ascii`）
- `/pubkey`：返回签名公钥（客户端首次获取后本地锁定，变更即拒绝通信）
- 管理端（用户/卡密/审计）独立鉴权，不在本契约范围
