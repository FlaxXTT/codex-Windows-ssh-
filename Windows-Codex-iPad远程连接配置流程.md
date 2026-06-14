# Windows Codex 连接 iPad ChatGPT 配置流程

本文记录在 Windows 上配置 Codex CLI，并让 iPad 上的 ChatGPT 发现并连接这台 Windows 电脑的完整流程。

## 适用前提

- Windows 已安装 Node.js 和 npm。
- Windows 已安装 Codex 桌面版或 Codex CLI。
- iPad 上已安装 ChatGPT App。
- Windows 和 iPad 登录的是同一个 ChatGPT/OpenAI 账号。
- 账号所在的 workspace 支持 Codex / remote connection 功能。

示例验证环境：

```powershell
codex --version
# codex-cli 0.133.0

npm --version
# 11.15.0
```

## 1. 检查终端是否已有 Codex CLI

在 PowerShell 中运行：

```powershell
Get-Command codex
codex --version
```

如果能看到版本号，例如：

```text
codex-cli 0.133.0
```

说明终端版 Codex CLI 已可用。

也可以查看实际路径：

```powershell
where.exe codex
```

常见路径包括：

```text
C:\Users\<你的用户名>\AppData\Roaming\npm\codex.cmd
C:\Program Files\WindowsApps\OpenAI.Codex_...\app\resources\codex.exe
```

## 2. 安装或更新 Codex CLI

如果没有安装，或想更新到最新版：

```powershell
npm install -g @openai/codex@latest
```

如果也想更新 npm：

```powershell
npm install -g npm@latest
```

更新后验证：

```powershell
codex --version
npm --version
```

## 3. 登录 ChatGPT 账号

运行：

```powershell
codex login
```

PowerShell 可能会打开浏览器或显示登录链接。按提示登录 ChatGPT/OpenAI 账号。

登录完成后，网页会显示类似：

```text
Signed in to Codex
You may now close this page
```

验证登录状态：

```powershell
codex login status
```

正常结果：

```text
Logged in using ChatGPT
```

## 4. 在 config.toml 中启用远程连接功能

Codex 的用户配置文件通常在：

```text
C:\Users\<你的用户名>\.codex\config.toml
```

可以用记事本打开：

```powershell
notepad "$env:USERPROFILE\.codex\config.toml"
```

在文件中加入或确认存在下面配置：

```toml
[features]
remote_connections = true
remote_control = true
```

如果文件里已经有 `[features]` 段，只需要把这两行加到原来的 `[features]` 段下面，不要重复写多个 `[features]`。

修改完成后保存文件。建议不要把自己的完整 `config.toml` 发布到网上，因为里面可能包含本机路径、项目目录、MCP 服务或环境变量配置。

**重要提醒：保存后建议把 `config.toml` 设置为只读。**

操作路径：

```text
C:\Users\<你的用户名>\.codex\config.toml
```

在文件资源管理器中打开：

1. 进入 `C:\Users\<你的用户名>\.codex`
2. 找到 `config.toml`
3. 右键文件，选择“属性”
4. 勾选“只读”
5. 点击“应用”，再点击“确定”

这样可以减少配置被自动改动或覆盖的情况。如果以后需要修改 Codex 配置、添加 MCP、切换插件或重新保存设置，需要先取消“只读”，改完后再重新勾选。

也可以用 PowerShell 检查是否已经只读：

```powershell
(Get-Item "$env:USERPROFILE\.codex\config.toml").IsReadOnly
```

如果返回 `True`，说明已经是只读。

## 5. 启动 Windows 远程连接服务

这是关键步骤。

在 PowerShell 中运行：

```powershell
codex app-server --listen off --remote-control
```

这个命令会在当前 PowerShell 会话里运行。只要它不退出，就表示服务还在。

如果你想让它在后台运行，可以新开一个 PowerShell 窗口专门执行这条命令，不要关闭窗口。

## 6. 这个命令的实际作用

```powershell
codex app-server
```

启动 Codex 的 app-server，也就是 Codex App、移动端或其他客户端可以使用的后台服务。

```powershell
--listen off
```

不开放本地 WebSocket/HTTP 监听端口。iPad 不是通过局域网 IP 直连 Windows，而是通过 Codex/ChatGPT 的远程连接通道发现和连接。

```powershell
--remote-control
```

开启远程控制注册。Codex 会使用 `codex login` 保存的 ChatGPT 登录态，把这台 Windows 电脑注册到当前账号/workspace 下。

整体链路是：

```text
Windows Codex CLI app-server
-> 使用 ChatGPT 登录态连接 OpenAI/Codex 远程服务
-> iPad ChatGPT 在同一账号/workspace 下发现这台电脑
-> 建立远程控制会话
```

## 7. 在 iPad 上连接

1. 打开 iPad 上的 ChatGPT App。
2. 进入 Codex / connection / remote desktop 相关页面。
3. 如果出现登录提示，先登录同一个 ChatGPT 账号。
4. 等待设备列表刷新。
5. 选择 Windows 设备。

连接成功后，iPad 上会显示你的 Windows 设备名。例如：

```text
YOUR-PC-NAME
```

可以在 Windows 上运行下面命令查看自己的设备名：

```powershell
hostname
```

## 8. 验证和排查

### 检查 Codex 是否登录

```powershell
codex login status
```

如果不是 `Logged in using ChatGPT`，重新运行：

```powershell
codex login
```

### 检查 Codex 版本

```powershell
codex --version
```

建议使用最新版：

```powershell
npm install -g @openai/codex@latest
```

### 检查诊断信息

```powershell
codex doctor
```

重点看：

- `auth is configured`
- `stored auth mode chatgpt`
- `websocket connected`
- `network reachable`

### iPad 一直 Waiting for desktop

按顺序处理：

1. 确认 PowerShell 里正在运行：

   ```powershell
   codex app-server --listen off --remote-control
   ```

2. 确认 iPad 和 Windows 登录的是同一个账号。
3. 完全关闭 iPad 上的 ChatGPT App，再重新打开。
4. 如果刚登录过账号，等待十几秒再刷新 connection 页面。
5. 重新运行启动命令。

### iPad 进入 connection 页面但没有设备

通常是下面几类原因：

- Windows 端没有运行 remote-control app-server。
- Windows 和 iPad 不是同一个 ChatGPT 账号。
- 账号相同但 workspace 不同。
- iPad App 页面状态没有刷新，需要完全退出后重开。
- Codex CLI 版本太旧，需要更新。

## 9. 重启电脑后怎么办

重启电脑后，之前的 PowerShell 进程会消失，需要重新运行：

```powershell
codex app-server --listen off --remote-control
```

然后再打开 iPad ChatGPT 的 connection 页面。

## 10. 不推荐的命令

下面这个命令在 Windows 上可能不可用：

```powershell
codex remote-control start
```

Windows 上可能出现的结果：

```text
Error: codex app-server daemon lifecycle is only supported on Unix platforms
```

本流程在 Windows 上实际可用的是：

```powershell
codex app-server --listen off --remote-control
```

## 11. 最小可复现流程

如果从零开始，只需要按这个顺序执行：

```powershell
npm install -g @openai/codex@latest
codex --version
codex login
codex login status
# 编辑 C:\Users\<你的用户名>\.codex\config.toml，加入 [features] 配置
codex app-server --listen off --remote-control
```
