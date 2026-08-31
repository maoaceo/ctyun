# ctyun-autoheal Docker 部署教程

`ctyun-autoheal` 是一个带自动健康检查与重启能力的 Docker 容器镜像。本文说明如何通过单条 `docker run` 命令部署、首次短信绑定以及查看健康状态。

> 镜像内保存账号配置时会涉及手机号和密码。请务必限制 `ctyun-data/accounts.json` 的文件权限，并不要将该目录提交到 Git 仓库。

## 镜像信息

| 项目 | 内容 |
| --- | --- |
| Docker Hub | [`lusean23/ctyun-autoheal`](https://hub.docker.com/r/lusean23/ctyun-autoheal) |
| 固定版本 | `lusean23/ctyun-autoheal:1.2.0` |
| 最新标签 | `lusean23/ctyun-autoheal:latest` |
| 镜像摘要 | `sha256:baea559729c08cb184490f724e7a06bd2a643085fb4c780e30625b0ccf61f4b0` |

## 1. 直接 Docker 运行（推荐）

### 创建数据目录与账号配置

```bash
mkdir -p /opt/ctyun-autoheal/ctyun-data
nano /opt/ctyun-autoheal/ctyun-data/accounts.json
```

填入配置，将手机号、密码替换成自己的信息：

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

限制账号文件的读取权限：

```bash
chmod 700 /opt/ctyun-autoheal/ctyun-data
chmod 600 /opt/ctyun-autoheal/ctyun-data/accounts.json
```

### 首次短信绑定

首次运行先执行以下命令，并根据终端提示完成短信绑定：

```bash
docker run --rm -it \
  -v /opt/ctyun-autoheal/ctyun-data:/app/data \
  -e CTYUN_DATA_DIR=/app/data \
  lusean23/ctyun-autoheal:1.2.0
```

绑定结果会保存到主机的 `/opt/ctyun-autoheal/ctyun-data`，后续重建容器不会丢失。

### 后台持续运行

```bash
docker run -d \
  --name ctyun-autoheal \
  --restart unless-stopped \
  --init \
  -v /opt/ctyun-autoheal/ctyun-data:/app/data \
  -e CTYUN_DATA_DIR=/app/data \
  --health-cmd='/usr/local/bin/ctyun-healthcheck' \
  --health-interval=60s \
  --health-timeout=10s \
  --health-start-period=90s \
  --health-retries=3 \
  lusean23/ctyun-autoheal:1.2.0
```

## 2. 常用管理命令

实时查看运行日志：

```bash
docker logs -f ctyun-autoheal
```

查看容器健康状态：

```bash
docker inspect \
  --format '{{json .State.Health}}' \
  ctyun-autoheal
```

停止容器：

```bash
docker stop ctyun-autoheal
```

启动已停止的容器：

```bash
docker start ctyun-autoheal
```

删除容器（不会删除主机上的账号数据）：

```bash
docker rm -f ctyun-autoheal
```

## 3. 更新镜像

更新前建议先停止并移除旧容器；账号数据因挂载在主机目录中会被保留。

```bash
docker pull lusean23/ctyun-autoheal:1.2.0
docker rm -f ctyun-autoheal
```

然后重新执行上方的“后台持续运行”命令。

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
