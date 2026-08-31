# ctyun-autoheal Docker Compose 部署教程

`ctyun-autoheal` 是一个带自动健康检查与重启能力的 Docker 容器镜像。Docker Compose 更适合长期运行：配置可保存、更新和重启命令也更简洁。

## 镜像信息

| 项目 | 内容 |
| --- | --- |
| Docker Hub | [`lusean23/ctyun-autoheal`](https://hub.docker.com/r/lusean23/ctyun-autoheal) |
| 固定版本 | `lusean23/ctyun-autoheal:1.2.0` |
| 最新标签 | `lusean23/ctyun-autoheal:latest` |
| 镜像摘要 | `sha256:baea559729c08cb184490f724e7a06bd2a643085fb4c780e30625b0ccf61f4b0` |

## 1. 安装 Docker Compose（可选）

如果系统执行 `docker compose version` 已能输出版本号，说明已安装 Compose V2，可直接跳到下一节。

若需要安装旧命令形式的 Compose 二进制，可执行以下命令。命令会根据当前系统和 CPU 架构自动选择对应文件：

```bash
sudo curl -fL "https://ghfast.top/https://github.com/docker/compose/releases/download/v2.4.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -sf /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose version
```

> 此处使用 `ghfast.top` GitHub 加速地址；已实测可访问 Docker Compose v2.4.1 的 Linux x86_64 文件。下载失败时可稍后重试，或优先安装 Docker 官方的 Compose V2 插件并使用 `docker compose`（中间有空格）命令。

## 2. 创建 Compose 配置

```bash
mkdir -p /opt/ctyun-autoheal
cd /opt/ctyun-autoheal
nano docker-compose.yml
```

粘贴以下内容，将 `手机号` 和 `密码` 替换为自己的账号：

```yaml
services:
  ctyun:
    image: lusean23/ctyun-autoheal:1.2.0
    container_name: ctyun-autoheal
    restart: unless-stopped
    init: true
    environment:
      APP_USER: "手机号"
      APP_PASSWORD: "密码"
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

> 这个方式**不需要创建 `accounts.json`**。账号由 `APP_USER` 与 `APP_PASSWORD` 环境变量提供。

保护配置文件，避免其他本机用户查看密码：

```bash
chmod 600 /opt/ctyun-autoheal/docker-compose.yml
```

## 2. 启动与首次绑定

启动服务（Docker 会自动拉取镜像）：

```bash
cd /opt/ctyun-autoheal
docker compose up -d
```

查看首次运行日志；如需要短信绑定，按终端日志提示完成：

```bash
docker compose logs -f
```

## 3. 常用命令

后台启动：

```bash
docker compose up -d
```

查看日志：

```bash
docker compose logs -f
```

停止并删除容器：

```bash
docker compose down
```

查看健康状态：

```bash
docker inspect --format '{{json .State.Health}}' ctyun-autoheal
```

## 4. 更新镜像

```bash
cd /opt/ctyun-autoheal
docker compose pull
docker compose up -d
```

## 可选：使用 `accounts.json` 配置多个账号

仅在配置多个账号，或不希望密码写在 Compose 文件中时使用。

```bash
mkdir -p /opt/ctyun-autoheal/ctyun-data
nano /opt/ctyun-autoheal/ctyun-data/accounts.json
```

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

然后删除 Compose 中的 `APP_USER` 与 `APP_PASSWORD` 两行，并在 `init: true` 下加入：

```yaml
    volumes:
      - ./ctyun-data:/app/data
```

## Docker Hub Token 安全说明

不要把 Docker Hub Personal Access Token 发到聊天、提交到 Git 仓库或写进镜像中。若已泄露，请立即在 [Docker Hub Token 页面](https://app.docker.com/settings/personal-access-tokens) 撤销并重新生成。

若不再需要从当前机器推送镜像：

```bash
docker logout
```
