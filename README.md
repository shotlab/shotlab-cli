# Shotlab CLI：把下面这段提示词交给 Agent

> 帮我在这台电脑上配置 Shotlab CLI 正式版，让当前 Agent 可以使用 Shotlab 创作图片和视频。
>
> 1. 阅读长期有效的安装说明：
> https://raw.githubusercontent.com/shotlab/shotlab-cli/main/INSTALL.md
> 2. 按说明读取公开的 `distribution.json`，下载并校验当前安装引导包。已有安装时沿用当前渠道，首次使用正式渠道，安装该渠道推荐的最新版 CLI 和完整 Skill，选择当前 Agent 对应的宿主。除非我明确指定版本，否则不要固定版本号。
> 3. 检查本机依赖；缺少 ffmpeg/ffprobe 时，先检查已有安装和 PATH，再由你按系统安装 FFmpeg、验证两个工具可运行，不要只把安装命令交给我。实际需要管理员密码、授权弹窗或遇到权限限制时，再请我完成必要步骤。执行 `shotlab login`，展示并打开本次登录的授权链接，等待我在网页完成授权，再执行 `shotlab login --complete`。
> 4. 验证 CLI 与 Skill 版本、登录状态及模型查询。就绪后告诉我可以开始哪些创作；如果还有缺失配置，请明确说明。不要自动提交付费生成任务。

支持 Codex、Claude Code。正式版连接正式环境，账号与可用模型以登录后实际查询结果为准。
下载安装无需登录 GitHub；Shotlab 创作需要用户在授权网页自行登录。

## 终端一条命令

macOS/Linux（Codex，正式渠道）：

```sh
curl -fsSL https://raw.githubusercontent.com/shotlab/shotlab-cli/main/install.sh | sh -s -- --channel stable --agent codex
```

Windows PowerShell（x64）：

```powershell
& ([scriptblock]::Create((Invoke-RestMethod 'https://raw.githubusercontent.com/shotlab/shotlab-cli/main/install.ps1'))) -Channel stable -Agent codex
```

Claude Code 将 `codex` 改为 `claude-code`。固定版本可用 `--version` / `-Version` 指定仓库中已发布的正式版。
同一命令可重复运行来更新；省略参数会沿用安装记录。首次不指定渠道默认 stable，当前没有正式版时会提示停止。
每次运行先读取 `distribution.json`，校验并使用其中推荐的安装器；CLI 最新版也只由该文件决定。配置缺失或无效时停止，不猜测其他最新版。
安装脚本不会自动登录。完成后打开新终端或重启 Agent，再按[安装说明](INSTALL.md)完成网页授权。

## 更新提示词

> 帮我将 Shotlab CLI 和配套 Skill 更新到当前渠道的最新版本，保留账号配置，并验证更新结果。
> 从 https://raw.githubusercontent.com/shotlab/shotlab-cli/main/INSTALL.md 获取当前安装入口，先按 `distribution.json` 更新安装器，再省略版本和渠道以沿用安装记录。不要自动提交付费生成任务。

## 下载、校验与平台状态

- [统一版本发现与引导包 SHA-256](distribution.json)
- [当前安装入口](INSTALL.md)
- [详细安装、依赖、迁移与恢复说明](INSTALL.md)

五种发行目标：macOS arm64/x64、Linux arm64/x64、Windows x64。
具体实机验证结果见安装器 Release 说明；未实机验证的目标不宣称已可用。
CLI 与完整 Skill 随版本一起安装。安装器自动检测 ffmpeg/ffprobe，同版本重跑也会复查。缺失或异常时显示当前系统的安装命令与 PATH 提示；由 Agent 配置时，Agent 继续检查已有安装、补齐依赖并验证，实际需要管理员密码、授权弹窗或遇到权限限制时才请用户参与。直接在终端安装的用户可自行按提示处理。Linux 凭据服务需单独配置。
不提供后台自动升级，也不要单独用 `skills update` 更新 Shotlab，以免 CLI 和 Skill 版本不一致。

本仓库只保存公开安装入口、用户说明与发行附件。源码与发布凭据不在本仓库维护。
