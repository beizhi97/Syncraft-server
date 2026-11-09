# Syncraft 服务端部署仓库（Syncraft-server）

这是 Syncraft 的**服务端独立部署交付仓库**，由主仓库 [beizhi97/Syncraft](https://github.com/beizhi97/Syncraft) 在发布时自动生成并同步。Windows 客户端用户请前往 [Syncraft-client](https://github.com/beizhi97/Syncraft-client/releases) 下载桌面客户端。

> 请勿直接在此仓库修改业务代码；所有代码与协议变更均在主仓库验证、打标后再同步至此仓库。

## 当前发布版本信息

- **发布标签（`releaseTag`）**：`v0.3.0-rc.3-sfsc-4765c02d125a`
- **构建批次（`buildId`）**：`sfsc-4765c02d125a`
- **主仓源码指纹（`sourceTreeHash`）**：`4765c02d125a54861f7e467e127b401a22548ad9ecb10583f44cffc3116c8822`
- **主仓提交（`sourceCommit`）**：`1c5029fe244ee814efb5c4dfe4a598951c0a9090`

---

## 1. N1 / Linux 普通目录模式部署（默认推荐）

普通目录模式（Ordinary-Directory Appliance）直接将 N1/Linux 主机上的既有目录通过受 `sfsc-root-guard` 与 `sfsc-relation-supervisor` 隔离的 Go 原生 SFTP 服务开放给获授权的 Windows 客户端，无需部署或依赖 MinIO。完整要求与排障步骤详见 [DEPLOYMENT.md](DEPLOYMENT.md)。

### 第一步：准备主机共享目录与环境文件

```bash
sudo install -d -m 0750 /srv/syncraft-shares
sudo cp .env.directory.example .env.directory
sudoedit .env.directory
```

在 `.env.directory` 中设置强随机 `SFSC_ADMIN_TOKEN` 以及客户端可访问的外部地址（切勿使用默认占位值）。

### 第二步：执行安装脚本

可先运行只读检查确认路径与配置无误，再执行正式安装：

```bash
sudo env SFSC_DIRECTORY_CHECK_ONLY=1 sh ./install-n1-directory-appliance.sh /srv/syncraft-shares
sudo sh ./install-n1-directory-appliance.sh /srv/syncraft-shares
```

安装完成后核对服务状态与统一健康检查端点：

```bash
curl --fail http://127.0.0.1:5177/healthz
systemctl status sfsc-appliance-directory.service
```

### 第三步：管理控制台首次配置与双码签发

浏览器打开管理控制台并输入 `SFSC_ADMIN_TOKEN` 后，按“首次配置”向导顺序操作：

1. **开放目录**：在“目录、配对与修复”面板登记主机共享目录（系统会自动写入并保护 `.sfsc-directory-root.json` 标记文件）。
2. **生成通道配置码**：配置并应用外部连接地址后，生成**通道配置码**发给 Windows 客户端用户。
3. **审批设备身份申请**：客户端导入通道配置码并提交申请后，在控制台“设备申请”列表中为该设备勾选一个或多个获授权目录，签发**一次性身份验证码**。
4. **一键重配与权限吊销**：
   - 如需更换通道端口或密钥，可使用“一键重设连接配置”并签发新通道配置码，已授权客户端导入新通道配置码后可直接沿用既有授权关系与同步基准。
   - 如需收回某个目录授权或停用设备，可在控制台直接撤销关系或禁用设备，服务端会实时终止（`SIGTERM`）对应在途传输会期。

---

## 2. S3 / MinIO 兼容模式与运维文档

- **S3 / MinIO 兼容模式**：存量 S3/MinIO 部署仍可使用 `./install-n1-appliance.sh` 或 Docker Compose 兼容配置运行，但普通目录模式（`sfsc-appliance-directory.service`）与 S3 兼容模式（`sfsc-appliance.service`）共享端口与状态目录，**不能同时启用**。
- **相关文档索引**：
  - [DEPLOYMENT.md](DEPLOYMENT.md)：N1/Linux 普通目录模式完整部署说明。
  - [docs/DEPLOYMENT_AND_MIGRATION_GUIDE.md](docs/DEPLOYMENT_AND_MIGRATION_GUIDE.md)：升级、备份恢复、状态自检与迁移指南。
  - [docs/INTERFACE_CONTRACT.md](docs/INTERFACE_CONTRACT.md) 与 [docs/API_MATRIX.md](docs/API_MATRIX.md)：服务端与客户端共享接口契约。
  - [docs/DIRECTORY_ROOT_CONTRACT.md](docs/DIRECTORY_ROOT_CONTRACT.md)：目录根安全标记与边界规范。

> 运行时敏感文件（`.env`、`.env.directory`、`data/`、`sing-box/config.json` 及任何密钥文件）均已由 `.gitignore` 排除，切勿提交至版本库。
