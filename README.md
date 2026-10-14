# Rocky Chess — 公开安装引导

本仓库仅放置经审核的安装入口、说明和 SHA256 校验值；**不存放网站源码、数据库、Token、Dropbox 授权或 Restic 密码**。网站源码始终保存在私有仓库 `ansinlia/Rocky-Chess`。

## 运行条件与安全边界

- 仅限事先批准的**全新 Debian 12 amd64 / x86_64、支持 AVX2** 的测试 VPS；使用具有交互终端的 `root` 账户。
- 私仓 `main` 必须先合并经过审核的五模块安装向导，否则脚本会拒绝安装旧版本。
- 从 GitHub 克隆私仓时，会提示输入 GitHub 用户名和 **Contents: Read** 权限仅覆盖 `Rocky-Chess` 的 Fine-grained Token。不要把 Token 粘进命令行、下载 URL、日志、文件或聊天。
- **本次测试须新建独立 Dropbox Scoped App Folder，并在其中初始化全新 Restic 仓库**。禁止把旧备份仓库所在的 App Folder 授权给本次全新安装。备份清理策略可能删除旧快照，因此必须隔离。
- **不允许 `curl | bash`**：交互式 Git 认证和五模块菜单依赖真实终端。应先下载、校验，再使用 `bash` 执行。
- 不会自动删除历史备份、已有网站、密钥、Docker 镜像或其他项目；首次配置会在独立测试仓库创建数据和备份。

## 在全新测试 VPS 上运行

在 VPS 的 **root SSH 终端**中运行以下完整命令。安装器临时文件会在本段命令退出后自动清除：

```bash
apt-get update && apt-get install -y --no-install-recommends curl ca-certificates
(
  set -e
  file="$(mktemp /tmp/rocky-public-start.XXXXXXXX)"
  trap 'rm -f -- "$file"' EXIT
  curl -fsSL https://raw.githubusercontent.com/ansinlia/Rocky-Chess-Bootstrap/main/rocky-chess-start.sh -o "$file"
  printf '%s  %s\n' '9ba456beb3e2b9d20469721ae8193393ae6fcbab59366604f3d66be9b847bf6d' "$file" | sha256sum -c -
  bash "$file" --fresh-staging
)
```

恢复旧 Staging 使用 `--restore-staging`；恢复独立 Production 使用 `--restore-production`，需要原 Restic 密码及 Dropbox App Folder 授权。**测试场景不应使用任何旧仓库恢复模式**。

## 维护约束

公开的 `rocky-chess-start.sh` 必须与私仓正式源 `deploy/rocky-chess-start.sh` **逐字节相同**；每次更新都要核对 SHA256、同步 `SHA256SUMS` 和上述启动命令的预期 SHA，并重新做隔离端到端测试。

**上线门槛：**本仓库公开发布、私有 `main` 合并、全新 VPS 真实安装和独立 Dropbox 首备恢复测试都通过之后，才能宣布正式安装就绪。
