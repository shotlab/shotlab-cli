# Shotlab 安装引导（installer 1.5.1）

本包负责安装 Shotlab CLI 和同版本完整 Skill。目前支持 Codex、Claude Code。
安装器不会登录，不会提交生成任务，不会安装 LibTV。

## 长期有效的入口与引导包发现

每次安装或更新先读取：

https://raw.githubusercontent.com/shotlab/shotlab-cli/main/distribution.json

仅接受 `schemaVersion: 2`，从 `installer.assets["shotlab-setup.zip"]` 获取 `url` 和 `sha256`。
下载 ZIP 并核对 SHA-256，再安全解压到临时目录；拒绝绝对路径、`..`、符号链接及重复路径。
引导包地址必须是本公开仓库对应 `installer-v<installer.version>` Release 的 `shotlab-setup.zip`，
不能执行配置指向的其他来源。配置读取、格式或校验失败时停止，不能回退到旧 `latest` 或自行扫描 Release 猜测最新版。
也可直接使用 README 中长期有效的 Shell/PowerShell 在线入口。

引导包和在线入口会先校验其固定核心，再读取同一份配置，校验并加载推荐的新版安装逻辑。
一次安装使用同一配置快照完成安装器与 CLI 选择；不在用户机器上修改引导包或建立后台更新任务。
`installer-v1.0.0` 的旧 ZIP 不具备此功能：首次迁移须从上述长期入口重新获取引导包，不能继续使用保存的旧 ZIP。

GitHub 安装、更新和指定版本安装均直接读取此清单中的附件地址与 SHA-256，不调用 GitHub Release API。
无需 GitHub Token。指定版本必须已收录在清单中；未收录或校验失败时停止，请联系发布维护者。

## 给 Agent 的执行步骤

1. 识别当前宿主：Codex 使用 `--agent codex`，Claude Code 使用 `--agent claude-code`。
   其他宿主说明目前的支持范围，不猜测 Skill 安装位置。
2. 用户明确指定版本时传入 `--version`；否则不要传版本号。
   已有安装且用户未要求切换渠道时省略 `--channel`，沿用安装记录。
   此公开入口仅支持 stable；首次默认 stable，尚无正式版时停止。
   其他渠道的安装记录不能通过此入口更新；切换到正式版须明确指定 `--channel stable`。
3. 在临时目录解压本包，按系统执行下列命令。macOS/Linux 不需要 sudo：

   ```sh
   sh ./install.sh --agent codex
   ```

   Windows PowerShell：

   ```powershell
   & ./install.ps1 -Agent codex
   ```

   已有正式版安装可保持省略渠道。
   Claude Code 将 `codex` 改为 `claude-code`。同时安装两个宿主可重复 `--agent`，
   PowerShell 用 `-Agent codex,claude-code`。`--no-modify-path` / `-NoModifyPath` 禁止修改 PATH。
   如果系统执行策略阻止本地脚本，请按组织政策处理；不要修改系统执行策略。
4. CLI 安装位置为 macOS/Linux 的 `~/.local/bin/shotlab`，Windows 的
   `%LOCALAPPDATA%\Shotlab\bin\shotlab.exe`。当前进程尚未刷新 PATH 时使用完整路径。
   检查 `shotlab --version`，读取下述安装记录中的 Skill `VERSION`、`SKILL.md`。
   如当前 Agent 尚未发现新 Skill，提示重启 Agent；当前会话可先直接读取完整 Skill。
5. 安装器会在安装、更新或同版本重跑完成后，自动执行 `ffmpeg -version`、`ffprobe -version`，
   分别报告可用、未找到或执行异常；每项检查最多 5 秒。缺失或异常时显示当前系统的安装命令和 PATH
   配置提示，不影响 CLI/Skill 安装成功，也不会自动安装 FFmpeg。按提示处理后打开新终端并重启 Agent，
   再执行两个版本命令，或重跑安装器复查。只有两个工具均可运行，媒体依赖检查才通过。
   Linux 登录还需要运行中的 D-Bus 用户会话与 Secret Service（例如 GNOME Keyring/KWallet）；
   无桌面的服务器需用户配置凭据服务。缺失时说明相应能力尚未就绪，不自动修改系统服务。
6. 读取配套 Skill 的 `commands/auth.md`，先执行 `shotlab auth status --json`。
   已登录且环境正确时继续验证；否则执行 `shotlab login --json`。
   展示本次返回的完整 `authorization_url`、到期时间及 `qr_code_path`，并使用当前宿主的浏览器
   打开授权链接。等待用户明确表示网页授权完成，再执行 `shotlab login --complete --json`。
   这会恢复本机保存的同一会话，不传会话 ID，不重复发起 login，不读取、输出或索取任何 token、
   密码或系统凭据。会话过期时说明原因后重新发起一次授权。
7. 执行 `shotlab auth status --json`、`shotlab account me --json`、`shotlab model list --json`。
   以实际返回的可用模型和 Skill 支持的能力说明可开始的创作。不要为了验证而提交付费生成任务。
   分别报告“CLI/Skill 安装完成”“登录完成”“创作依赖齐全”；某项失败不要笼统声称全部就绪。
8. 安装结束后删除本次下载/解压的临时引导包目录；保留运行时、安装记录、升级备份。

## 更新和回退

从长期入口重新获取安装说明或当前引导包，就是更新安装器和 CLI/Skill。同版本且文件完整时跳过。已有宿主始终随 CLI 同步更新。
省略渠道沿用原记录，此入口只支持正式渠道；显式 `--channel stable` 可将其他渠道安装切换到正式版。
显式 `--version v…` 可回退，同时恢复该版本的完整 Skill；账号配置保持不变。

```sh
sh ./install.sh
sh ./install.sh --channel stable --agent codex --agent claude-code
sh ./install.sh --version <已发布正式版标签>
```

CLI/Skill 文件被修改或缺失时会停止。请保存自己的修改，再恢复安装记录对应 Release 的原始文件。
已有手动安装而没有安装记录时，请先备份并移出提示的 CLI/同名 Skill 路径，再执行安装。
不要删除账号目录或系统凭据。其他 Skill 不受影响。

## 安装记录与恢复

macOS/Linux：`~/.local/share/shotlab-installer/`；Windows：`%LOCALAPPDATA%\Shotlab\installer\`。
`install.json` 保存版本、渠道、宿主与文件校验记录；`backups/` 为每次升级独立保存备份。
`runtime/` 是需要时安装的 Node 24.22.0，系统 Node 不会被替换；`npm-cache/` 供固定
`skills@1.7.0` 使用。安装工具只在隔离 HOME 中安装完整 Skill，然后以事务复制到宿主目录。
Codex 使用标准工具的 `~/.agents/skills/shotlab`，Claude Code 使用
`${CLAUDE_CONFIG_DIR:-~/.claude}/skills/shotlab`。Codex 旧 `.codex/skills/shotlab` 存在时停止，避免重复加载。
自定义宿主目录必须仍在用户目录中且不包含链接。

失败时自动恢复本次修改。进程被强行终止时可能留下 `lock/` 与 `transaction.json`：
先检查 `lock/owner.json` 的 PID，确认安装进程已退出，再仅删除 `lock/`，重新运行同一命令自动恢复。
不要删除 `transaction.json` 或备份。若恢复时检测到新的本地修改，会停止并指出需要人工处理的路径。
运行时下载的锁是 `runtime/lock/`；确认没有安装进程后才可以清理。

所有下载使用 HTTPS 并验证 SHA-256。引导脚本固定共享安装逻辑校验值，Node 校验值随安装器固定。
每个引导包的校验文件位于其对应的 `installer-v…` Release；推荐版本和 SHA-256 由 `distribution.json` 给出。只从公开发行仓库下载，不需要仓库账号或发布令牌。

渠道记录为 `null` 时表示尚无该渠道版本，不会回退到另一渠道。渠道配置还固定 CLI 的 `checksums.txt` SHA-256，
安装器核对它与 Release 一致后，继续校验各包、压缩路径、CLI 版本和完整 Skill。
显式 `--version` 查询该标签 Release，并执行原有完整校验；不受推荐 CLI 版本限制。
默认更新拒绝降级 CLI 或安装器；只有 CLI 的显式版本参数允许回退。
