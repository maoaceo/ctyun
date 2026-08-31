# ctyun-autoheal Docker 部署教程

`ctyun-autoheal` 是一个带自动健康检查与重启能力的 Docker 容器镜像。本文说明如何拉取、首次短信绑定、以 Docker Compose 持续运行，以及查看健康状态。

> 镜像内保存账号配置时会涉及手机号和密码。请务必限制 `ctyun-data/accounts.json` 的文件权限，并不要将该目录提交到 Git 仓库。

## 镜像信息

| 项目 | 内容 |
| --- | --- |
| Docker Hub | [`lusean23/ctyun-autoheal`](https://hub.docker.com/r/lusean23/ctyun-autoheal) |
| 固定版本 | `lusean23/ctyun-autoheal:1.2.0` |
| 最新标签 | `lusean23/ctyun-autoheal:latest` |
| 镜像摘要 | `sha256:baea559729c08cb184490f724e7a06bd2a643085fb4c780e30625b0ccf61f4b0` |

## 1. 拉取镜像

建议生产环境固定版本：

```bash
docker pull lusean23/ctyun-autoheal:1.2.0
```

也可以使用最新标签：

```bash
docker pull lusean23/ctyun-autoheal:latest
```

## 2. 使用 Docker Compose 部署

### 创建工作目录

```bash
mkdir -p /opt/ctyun-autoheal/ctyun-data
cd /opt/ctyun-autoheal
```

### 创建 `docker-compose.yml`

```yaml
services:
  ctyun:
    image: lusean23/ctyun-autoheal:1.2.0
    container_name: ctyun-autoheal
    restart: unless-stopped
    init: true
    volumes:
      - ./ctyun-data:/app/data
    environment:
      CTYUN_DATA_DIR: /app/data
    stdin_open: true
    tty: true
    healthcheck:
      test: ["CMD", "/usr/local/bin/ctyun-healthcheck"]
      interval: 60s
      timeout: 10s
      start_period: 90s
      retries: 3
```

### 创建账号配置

创建文件：

```bash
nano /opt/ctyun-autoheal/ctyun-data/accounts.json
```

填入配置（将手机号、密码替换成自己的信息）：

```json
{
  "keepAliveSeconds": 60,
  "accounts": [
    {
      "name": "账号一",
      "user": "手机号",
      "password": "***",
      "deviceCode": ""
    }
  ]
}
```

为避免其他本机用户读取配置，设置权限：

```bash
chmod 700 /opt/ctyun-autoheal/ctyun-data
chmod 600 /opt/ctyun-autoheal/ctyun-data/accounts.json
```

## 3. 首次短信绑定

首次运行时执行：

```bash
cd /opt/ctyun-autoheal
docker compose run --rm ctyun
```

按照终端提示完成短信绑定。绑定信息会保存到挂载的 `./ctyun-data` 目录中。

## 4. 启动和查看日志

绑定完成后启动后台服务：

```bash
docker compose up -d
```

实时查看日志：

```bash
docker compose logs -f
```

停止服务：

```bash
docker compose down
```

更新镜像后重新创建容器：

```bash
docker compose pull
docker compose up -d
```

## 5. 查看健康状态

```bash
docker inspect \
  --format '{{json .State.Health}}' \
  ctyun-autoheal
```

也可快速查看容器状态：

```bash
docker ps --filter name=ctyun-autoheal
```

## Docker Hub Token 安全说明

不要将 Docker Hub Personal Access Token 发到聊天、提交到 Git 仓库或写进镜像中。

如果 Token 已意外暴露，请立即到 Docker Hub 控制台撤销并重新生成：

<https://app.docker.com/settings/personal-access-tokens>

如不再需要从当前服务器推送 Docker 镜像，可清除本机登录状态：

```bash
docker logout
```

Docker 登录凭据通常位于：

```text
/root/.docker/config.json
```

请妥善保护该文件，且不要上传它。
