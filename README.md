# Pshell

[English](#english) · [简体中文](#简体中文)

## English

Pshell is a Windows SSH client for organizing sessions and working in tabbed terminals. Use Local Sessions without an account, or connect to a compatible Pshell server for optional Cloud Sync.

### Download and connect

1. Download the Windows x64 installer (`Pshell.Setup.<version>.exe`) from [GitHub Releases](https://github.com/pengdayou/Pshell-Releases/releases/latest) or the [official download page](https://download.dayanfeng.com/).
2. Run the installer and open Pshell.
3. Choose **New Session** (**新建会话** in Simplified Chinese), enter the host, port, and username, then select password or private key authentication. Create the session and select it in the sidebar to connect.

To verify the installer, compare its SHA-256 hash with the matching entry in `SHA256SUMS.txt` attached to the release. Run `Get-FileHash -Algorithm SHA256` in PowerShell and supply the installer path when prompted.

### What you can do

- **Organize connections.** Group SSH sessions, search by name, and work in multiple terminal tabs.
- **Configure the terminal.** Search scrollback, highlight log output, and choose fonts, color schemes, and background images.
- **Back up or sync sessions.** Import and export session configurations, encrypt backups with a password, or review local and cloud changes before applying them. Optional end-to-end encryption (E2EE) protects passwords, private keys, and passphrases before upload.

### Display language

In Pshell 2.7.4 and later, new profiles use English by default. Select **Settings → Appearance → Display Language → 简体中文**, then choose **Save**. Pshell applies the change immediately and remembers it on this device.

### Support and license

Send feedback or licensing inquiries to Areom at [felixhart921227@gmail.com](mailto:felixhart921227@gmail.com). For a bug report, include your Pshell version, Windows version, and steps to reproduce the issue.

Pshell is free for personal use. Commercial use as defined in the included software license requires written authorization.

## 简体中文

Pshell 是一款 Windows SSH 客户端，用于整理连接和管理多标签终端。无需账号即可使用本机会话，也可以连接兼容的 Pshell 服务器，按需启用云同步。

### 下载与连接

1. 从 [GitHub Releases](https://github.com/pengdayou/Pshell-Releases/releases/latest) 或[官网下载页](https://download.dayanfeng.com/)下载 Windows x64 安装包（`Pshell.Setup.<版本号>.exe`）。
2. 运行安装包，安装完成后打开 Pshell。
3. 选择「新建会话」，填写主机、端口和用户名，再选择密码或私钥认证。创建后，在侧边栏点选会话即可连接。

如需校验安装包，在 PowerShell 中运行 `Get-FileHash -Algorithm SHA256`，按提示输入安装包路径，再与同一版本发布附件 `SHA256SUMS.txt` 中的对应记录比对。

### 主要功能

- **管理 SSH 连接。** 按分组整理会话，搜索连接，在多个终端标签间切换。
- **配置终端。** 搜索回滚记录、高亮日志输出，调整字体、配色和背景图。
- **备份或同步会话。** 导入、导出会话配置，可给备份设置密码，也可在确认本机和云端的差异后同步。启用端到端加密（E2EE）后，上传前会在本机加密密码、私钥和私钥口令。

### 界面语言

从 Pshell 2.7.4 起，新安装默认使用英文。打开 **Settings → Appearance → Display Language**，选择 **简体中文** 后点击 **Save**。语言即时切换并保存在本机。切回英文时，打开「设置 → 外观 → 界面语言」，选择 **English** 并保存。

### 联系与许可

问题反馈和授权咨询请联系作者 Areom，邮箱为 [felixhart921227@gmail.com](mailto:felixhart921227@gmail.com)。反馈问题时，可附上 Pshell 版本、Windows 版本和复现步骤。

Pshell 可供个人免费使用。许可协议所列的商业用途须取得书面授权，详见安装包随附的软件许可协议。
