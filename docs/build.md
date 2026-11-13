# 构建与打包

## 发行物

| 文件 | 说明 |
|---|---|
| `老K破甲-Setup-vX.Y.Z.exe` | 自解压安装包（约 26MB），双击安装，零控制台窗口 |
| 桌面 `老K破甲执行器.lnk` | 启动器快捷方式（带图标） |

## payload 结构

```
payload/
├── bin/
│   ├── node.exe            # 便携运行时（零系统依赖）
│   ├── EniLauncher.exe     # 托盘宿主（C# WinForms，编译时嵌入图标）
│   ├── eni-launcher.js     # 客户端界面服务（8444）
│   ├── eni-proxy.js        # 协议代理（8443）
│   ├── eni-verify.js       # 通信安全层（验签/锁定/防重放/机器码）
│   ├── eni-license.js      # 持续心跳验卡
│   ├── eni-platforms.js    # 五平台激活/还原
│   ├── eni-guard.js        # 配置自愈守护
│   └── eni-core.js         # 上下文注入核心（闭源）
├── etc/
│   ├── ENI.md              # 人格规范（闭源，不随本仓库分发）
│   ├── skills/             # 技能库（闭源，不随本仓库分发）
│   └── ...
├── install.ps1             # 安装逻辑（部署文件 → 注入五平台 → 清单 → 启动）
└── uninstall.ps1           # 卸载（平台还原 → 激活码备份到桌面 → 清理）
```

## 安装流程（install.ps1）

```
0. 识别全新安装 / 升级
1. 停旧服务（guard/proxy/launcher）
2. 部署 bin/etc 到 %LOCALAPPDATA%\ENI
3. 五平台配置注入（已有配置改地址；缺失则生成骨架）
4. 注册开机自启（HKCU Run，零窗口）
4.5 激活码输入（有卡自动保留，绝不阻塞流程）
4.8 生成完整性清单 etc/manifest.sha256（bin/*.js 的 SHA-256）
5. 启动托盘宿主 EniLauncher.exe（代理由它拉起）
6. 自检：客户端 8444 + 代理 8443 双探活
```

## 打包链

```
payload/*  ──7z──▶  payload.7z
payload.7z + 7z.exe + EniLauncher.exe(--install 模式)
           ──IExpress(SED)──▶  ENI-Patch-Setup.exe
```

- 安装器入口是 `EniLauncher.exe --install`（winexe，无控制台），它负责解压 → 调 install.ps1 → 弹窗报结果 → 启动托盘
- IExpress 的 SED 必须把所有文件列入 FILEn 段（漏一个 = 运行时静默失败）
- `install.cmd` 若走 cmd 执行，编码必须是 GBK（UTF-8 中文会把 echo 吃掉）

## 版本升级

重跑安装包即升级：version.json 记录 installedAt/mode；本地凭据（激活码/账号）保留；
manifest 重新生成。平台配置缺失/被重置由守护进程自动修复。
