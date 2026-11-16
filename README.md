# Antigravity 2.0 Desktop Plugin

把 Antigravity 2.0 桌面版自身的运维能力抽出来做成插件：**MCP 服务器（21 个工具）** + **技能（`ag2-desktop`）** + **本地控制面板**，并且**装完自动解锁、被更新还原后自动改回**。

## 功能

- **自动应用，无需手动操作**：安装时立刻检查 `language_server.exe` 的资格校验位，未解锁就当场写入；Google 更新把字节改回去后，下一轮触发自动改回。
- **三重触发**：安装瞬间跑一次 · 每次宿主拉起 MCP 服务器时后台再跑一次 · 系统定时（默认每 5 分钟）兜底。
- **免权限的定时兜底**：Windows 先试 `schtasks`，建任务被拒（无管理员权限）时自动退回登录启动的常驻守护（`Startup\ag2-heal.vbs`，wscript 静默、无窗口）；macOS 用 LaunchAgent，Linux 用 `systemd --user`，没有 systemd 退回 `crontab`。
- **安全写入链**：备份 `<exe>.agybak` → 定点写 6/8 字节 → 重新探测复验 → 失败用备份回滚；签名不认识一律不写（`unsupported`），文件被占用记为 `locked` 并在下一轮重试。
- **MCP 工具 21 个**：`ag2_status` · `ag2_doctor` · `ag2_binary_state` · `ag2_heal_status/now/config` · `ag2_settings_get/set` · `ag2_mcp_list/add/remove` · `ag2_skills_list` · `ag2_skill_read` · `ag2_projects_list` · `ag2_storage_get/set` · `ag2_logs_tail` · `ag2_devtools_port` · `ag2_launch` · `ag2_panel` · `ag2_paths`。
- **技能随装随用**：`ag2-desktop` 把上面这些工具整理成一份 agent 直接照着做的操作手册（状态 → 诊断 → 修复 → 日志的顺序）。
- **本地控制面板**：`http://127.0.0.1:47913`，令牌 + HttpOnly Cookie + `CSP: default-src 'none'`，只监听回环，不暴露任何远程接口。
- **零第三方依赖**：只用 Node 内置模块；系统没有 node 时自动用 App 自带的 Electron 当运行时（`ELECTRON_RUN_AS_NODE=1`）。
- **可回滚**：`ag2 heal --restore` 还原二进制；`ag2 uninstall` 只删自己创建的文件，`app.asar` 全程只读。

## 安装
0.使用agent帮助安装，下载Releases文件，解压放在一个文件夹，直接让agent安装到antigravity插件目录


1. 解压发布包（或克隆源码），在插件目录执行一键安装：

   ```powershell
   $env:ELECTRON_RUN_AS_NODE=1; & "$env:LOCALAPPDATA\Programs\antigravity\Antigravity.exe" bin\ag2.js install --interval=300
   ```

   - 复制自身到 `%USERPROFILE%\.gemini\ag2-plugin\app`
   - 生成 MCP 垫片 `run-mcp.cmd`（macOS / Linux 为 `run-mcp.sh`），把 `mcpServers["antigravity2-plugin"]` 写进 `%USERPROFILE%\.gemini\settings.json`（写前备份 `.bak`，不动其它服务器）
   - 安装技能到 `%USERPROFILE%\.gemini\skills\ag2-desktop`
   - 注册定时自动修复，并立刻执行一次修复

   macOS / Linux 把命令里的运行时换成 `node` 即可：`node bin/ag2.js install --interval=300`。

2. 重启 Antigravity 2.0（配置只在启动时读取）。MCP 子进程是懒启动，开第一段对话时才会拉起，进程列表里暂时看不到是正常的。

3. 验收：

   ```powershell
   & "$env:LOCALAPPDATA\Programs\antigravity\Antigravity.exe" test\verify-registered.js
   ```

   期望输出 `tools/list 返回 21 个工具`、`ag2_status` 显示版本与 `patched`、`ag2_doctor` 12 项全过。

常用命令：`ag2 status` · `ag2 doctor` · `ag2 heal --status` · `ag2 heal --off`（关自动修复）· `ag2 heal --restore`（还原二进制）· `ag2 autostart --remove`（移除定时）· `ag2 panel`（开面板）· `ag2 uninstall`。

## 声明

- 闸门签名与偏移是对 2.x 实测所得（x64 写入 `c64008019090` @ +0，arm64 写入 `2300805203200039` @ +0）；Google 改版后可能失效，失效时状态为 `unsupported` 并保持文件原样，需要按新二进制重新取签名。
- 自动修复只写 `language_server.exe` 的 6/8 字节，写前必备份，写完复验，失败回滚；`app.asar` 与用户数据不写入。
- 控制面板仅绑定 `127.0.0.1`，需要 24 字节令牌，页面带严格 CSP；不提供下载、注入、远程执行能力。
- 目前主要在 Windows + Antigravity 2.28.2 上实测；macOS / Linux 的定时触发路径按平台 API 实现，尚未长期跑过，欢迎反馈。

## License

GPL-3.0-or-later，见 [LICENSE](LICENSE)。更详细的说明与逆向依据见 [docs/USAGE.md](docs/USAGE.md)、[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)、[docs/REVERSE-ENGINEERING.md](docs/REVERSE-ENGINEERING.md)。
