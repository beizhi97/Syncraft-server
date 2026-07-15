# Syncraft 服务端

这是 Syncraft 的**独立服务端部署仓库**。克隆此仓库即可部署管理控制台、VLESS/REALITY 隧道和 MinIO 对象存储；它不包含 Windows 客户端构建文件。

Windows 客户端请到 [Syncraft-client](https://github.com/beizhi97/Syncraft-client) 的 Releases 下载。开发源码、协议合同与完整测试在 [Syncraft](https://github.com/beizhi97/Syncraft)。

## 快速部署

部署主机需要 Docker Engine 与 Docker Compose v2，并且公网 DNS 已指向此主机，80、443 和 `VLESS_PORT` 已放行。

```bash
git clone https://github.com/beizhi97/Syncraft-server.git
cd Syncraft-server
cp .env.example .env
```

在 `.env` 中至少替换下面的值，不能保留示例密码：

```text
SFSC_PUBLIC_HOST=sync.example.com
PUBLIC_SERVER_URL=https://sync.example.com
VLESS_PUBLIC_HOST=sync.example.com
SFSC_ADMIN_TOKEN=请替换为高强度随机值
MINIO_ROOT_PASSWORD=请替换为高强度随机值
```

启动：

```bash
docker compose up --build -d
docker compose ps
```

打开 `https://你的域名/`，用 `SFSC_ADMIN_TOKEN` 进入管理控制台，生成新的 SyncKey 后交给对应 Windows 用户导入。不要把 SyncKey、管理 Token、VLESS URI 或 `.env` 上传到 Issue、聊天记录或截图。

## 升级与备份

升级前先运行 `sh ./sfscctl backup`，备份文件与 `.env`、`data/`、`sing-box/config.json` 应保存在受控位置。更新源码后执行：

```bash
git pull --ff-only
docker compose up --build -d
sh ./sfscctl health
```

N1/host-network 部署请使用 `docker compose -f docker-compose.n1.yml up --build -d`。该模板的 MinIO 数据路径为 `/ssd/sfsc-test/minio`；确认该 SSD 挂载存在后再启动，禁止把生产对象数据迁回系统盘。

## 安全说明

- `data/`、`.env`、`sing-box/config.json` 均为运行态或敏感数据，已被 Git 忽略。
- 默认不把管理 API 直接暴露为未认证服务；启动时缺少或使用示例管理 Token 会失败。
- 镜像版本固定在 `.env.example` 中，升级前请先在测试环境验证。

当前交付版本：`v0.3.0-rc.1`。服务端与客户端以协议 v1 为兼容基线；升级任意一端前请阅读对应 Release 说明。
