# Pshell

[English](#english) · [简体中文](#简体中文)

## English

**Free for personal use · No sign-up for local use · Offline session management**

Pshell is a Windows x64 SSH client for saved connections and tabbed terminals. Manage sessions, make encrypted backups, and read the built-in guide offline. Connect directly to reachable SSH hosts, including on private networks. Cloud Sync is off by default.

[Download for Windows x64](https://download.dayanfeng.com/) · [Releases and SHA-256 checksums](https://github.com/pengdayou/Pshell-Releases/releases/latest) · [Self-hosted sync API](docs/PSHELL_SYNC_API.md) · [Email feedback](#support-and-license)

### Get started

Install Pshell and choose **New Session**. Enter the host, port, username, and authentication details. Select **Test Connection** to check the current settings, then save and open the session from the sidebar. For installer verification, run `Get-FileHash -Algorithm SHA256` in PowerShell, enter the installer path when prompted, and compare the result with the release's `SHA256SUMS.txt`.

### Basic features

- **Sessions and groups.** Create, edit, clone, and search saved sessions. Assign colors, drag sessions across groups, reorder groups, and restore or permanently remove sessions from the Recycle Bin.
- **SSH authentication.** Connect with a password or private key, with an optional key passphrase. Test SSH handshake and authentication in the new or edit session dialog before saving.
- **Tabbed terminals.** Open several connections in one window, open another tab for the same host, and rename or reorder tabs. Use **Ctrl+F** to search scrollback, and review multiline text before pasting.
- **Backup and migration.** Export session configurations as JSON with password encryption selected by default. Preview imports and choose to add, replace, or skip sessions with matching names.
- **Local data.** App data defaults to `%APPDATA%\Pshell`. The Windows uninstaller offers a choice to keep or remove it.
- **Display settings.** Choose light, dark, or eye comfort mode and one of eight terminal color schemes. Set terminal fonts, spacing, cursor style, and scrollback. Use zoom or full screen for more readable output.

### Advanced features

- **Connection settings.** UTF-8 terminal with configurable terminal type, remote locale, keepalive interval, and optional auto-reconnect.
- **Output and diagnostics.** Highlight log levels, IP addresses, and other common tokens. In Bash sessions, enable Command Navigation to jump between commands with **Ctrl+Alt+↑/↓** and mark failures. Keystroke echo timing reports how long Pshell waited for a reply, parsed the output, and waited for rendering.
- **Log Protection.** Temporarily suppress selected terminal query responses for the current connection when viewing replayed logs.
- **Window customization.** Adjust terminal contrast and background opacity. Drag an image to position it and choose a full-window image, frosted image, or desktop Acrylic. Acrylic requires Windows 11 22H2 or later with system transparency enabled.
- **Local credential protection.** Saved passwords, private keys, key passphrases, and login tokens use Windows secure storage. Copy connection details separately or set a clipboard timeout for copies that include a password.

### Optional Cloud Sync

- **Separate workspaces.** Keep a local workspace and, after signing in to a compatible server, copy selected sessions and groups to the Account Workspace.
- **Review before applying.** Check for differences manually or on a schedule. Review changes individually or in batches, resolve eligible conflicts field by field, or skip changes for later.
- **Credential encryption.** Where supported, enable end-to-end encryption for SSH passwords, private keys, and key passphrases. The master password stays on the client and can be changed. Lock credentials manually, after inactivity, or when Windows locks.
- **Device management.** View signed-in devices, use the device security key to revoke another device’s access, and review sync and sign-in notices. Signing out clears the account cache on this device.
- **Account preferences and self-hosting.** Publish, apply, or follow supported settings such as terminal font size and theme. To use your own compatible server, enter its URL in **Settings → Cloud Sync**. [Sync API and examples](docs/PSHELL_SYNC_API.md)

### Support and license

New profiles start in English. Switch to **简体中文** under **Settings → Appearance → Display Language**. **Help → Quick Start** opens the offline guide. **Help → Email Feedback** opens a draft with the Pshell version or lets you copy the address.

Pshell is free for personal use, including individual use at work. Commercial uses covered by the bundled license require written authorization. For feedback, cloud access inquiries, or licensing, contact Felix Hart (formerly Areom). [felixhart921227@gmail.com](mailto:felixhart921227@gmail.com?subject=Pshell%20Feedback)

## 简体中文

**个人免费 · 绿色本机运行 · 本机功能离线可用**

Pshell 是面向个人开发者和运维人员的 Windows x64 SSH 客户端。免注册即可管理本机会话、加密备份和查看帮助。SSH 直连可达的目标主机，在仅有内网的环境中也能使用。云同步默认关闭。

[下载 Windows x64 安装包](https://download.dayanfeng.com/) · [版本说明与 SHA-256 校验](https://github.com/pengdayou/Pshell-Releases/releases/latest) · [自建同步接口](docs/PSHELL_SYNC_API.md) · [邮件反馈](#支持与许可)

### 安装与上手

安装后打开「新建会话」，填写主机、端口、用户名和认证信息。先点「测试连接」，确认后保存，再从侧边栏打开会话。若要校验安装包，在 PowerShell 运行 `Get-FileHash -Algorithm SHA256`，按提示输入安装包路径，再与同版本的 `SHA256SUMS.txt` 核对。

### 基础功能

- **会话与分组。** 新建、编辑、克隆和搜索会话，设置颜色，拖动排序或跨组移动。回收站支持恢复和彻底删除。
- **SSH 认证。** 支持密码、私钥及私钥口令。新建和编辑时都可在保存前验证 SSH 握手和认证。
- **多标签终端。** 在一个窗口打开多个连接，同一主机可开多个终端。标签支持重命名和排序，**Ctrl+F** 搜索回滚输出，多行粘贴前可预览确认。
- **备份与换机。** 支持会话 JSON 导入与导出，导出默认勾选密码加密。导入时预览会话，同名项可选新建、覆盖或跳过。
- **本机资料。** 默认保存在 `%APPDATA%\Pshell`。使用卸载界面时，可选择保留或清理本机资料。
- **界面与终端。** 支持日间、夜间和护眼模式，内置八套终端配色。可调字体、间距、光标和回滚行数，也支持窗口缩放和全屏。

### 高级功能

- **连接调优。** 使用 UTF-8 终端，可设置终端类型、远端语言环境和心跳间隔，按需开启断线自动重连。
- **输出与诊断。** 高亮日志级别、IP 地址等内容。Bash 会话可开启命令导航，通过 **Ctrl+Alt+↑/↓** 跳转命令并标记失败。回显测量分段显示等待回复、解析输出和等待渲染的耗时。
- **日志保护。** 回看历史日志时，可为当前连接临时拦截部分终端查询响应。
- **窗口定制。** 可调终端对比度和底色不透明度，背景图可拖动取景，支持整窗铺图、同图磨砂和桌面亚克力。亚克力需要 Windows 11 22H2 或更新版本，并开启系统透明效果。
- **本机凭据保护。** 保存的密码、私钥、私钥口令和登录令牌使用 Windows 系统安全存储。可单独复制公开连接信息，含密码的复制支持定时清空。

### 可选云同步

- **工作区分离。** 本机会话保存在本机。登录兼容服务后，可选中会话和分组，将副本加入账号工作区。
- **同步前确认。** 可手动或定时检测变化，按项或批量处理。可合并的冲突支持按字段选择，也可跳过暂不处理的变更。
- **凭据加密。** 后端支持时可开启端到端加密，保护上传的 SSH 密码、私钥和私钥口令。主密码仅在客户端使用，可更换。凭据支持手动锁定、闲置锁定和 Windows 锁屏锁定。
- **设备管理。** 可查看登录设备、用安全匙撤销其他设备，并在通知中心查看同步与登录提醒。主动退出时会清理本机的账号缓存。
- **偏好与自建服务。** 可逐项发布、应用或跟随终端字号、主题等支持的偏好。在「设置 → 云同步」填写兼容服务器的地址即可接入。[接口与返回示例](docs/PSHELL_SYNC_API.md)

### 支持与许可

新配置默认英文。在 **Settings → Appearance → Display Language** 可切换简体中文。「帮助 → 快速上手」可离线查阅，「帮助 → 邮件反馈」可打开预填 Pshell 版本的草稿或复制邮箱。

个人使用免费，包括工作场景中的个人使用。许可协议所列商业用途须取得书面授权，详见安装包随附协议。问题反馈、云同步咨询和授权可联系作者 Felix Hart（旧称 Areom）。[felixhart921227@gmail.com](mailto:felixhart921227@gmail.com?subject=Pshell%20Feedback)
