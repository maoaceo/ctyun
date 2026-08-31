# ctyun-autoheal Docker 部署教程

`ctyun-autoheal` 是一个带自动健康检查与重启能力的 Docker 容器镜像。

镜像启动时必须得到账号信息：可通过环境变量直接传入，也可以挂载 `accounts.json`。如果不提供任一种，镜像会提示“未读取到账号配置”。

> 下方的环境变量写法不需要提前创建配置文件；但密码会出现在 Shell 历史记录中。更注重隐私时，请使用文末的 `accounts.json` 挂载方式。

## 镜像信息

| 项目 | 内容 |
| --- | --- |
| Docker Hub | [`lusean23/ctyun-autoheal`](https://hub.docker.com/r/lusean23/ctyun-autoheal) |
| 固定版本 | `lusean23/ctyun-autoheal:1.2.0` |
| 最新标签 | `lusean23/ctyun-autoheal:latest` |
| 镜像摘要 | `sha256:baea559729c08cb184490f724e7a06bd2a643085fb4c780e30625b0ccf61f4b0` |

## 一条命令直接运行

将 `手机号` 与 `密码` 换成自己的账号信息：

```bash
docker run -d --name ctyun-autoheal --restart unless-stopped --init -e APP_USER='手机号' -e APP_PASSWORD='密码' -e CTYUN_DATA_DIR=/app/data --health-cmd='/usr/local/bin/ctyun-healthcheck' --health-interval=60s --health-timeout=10s --health-start-period=90s --health-retries=3 lusean23/ctyun-autoheal:1.2.0
```

Docker 会自动拉取镜像。若应用要求首次短信绑定，请查看容器日志并按提示操作：

```bash
docker logs -f ctyun-autoheal
```

## 常用管理命令

查看容器健康状态：

```bash
docker inspect --format '{{json .State.Health}}' ctyun-autoheal
```

停止容器：

```bash
docker stop ctyun-autoheal
```

启动已停止的容器：

```bash
docker start ctyun-autoheal
```

删除容器：

```bash
docker rm -f ctyun-autoheal
```

更新镜像后重新创建：

```bash
docker pull lusean23/ctyun-autoheal:1.2.0
docker rm -f ctyun-autoheal
```

随后重新执行上方的“一条命令直接运行”。

## 可选：使用配置文件保存多个账号

多个账号或不希望将密码写进命令时，才需要创建配置文件：

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

然后将上方命令中的 `-e APP_USER=... -e APP_PASSWORD=...` 替换为：

```bash
-v /opt/ctyun-autoheal/ctyun-data:/app/data
```

并建议保护文件权限：

```bash
chmod 700 /opt/ctyun-autoheal/ctyun-data
chmod 600 /opt/ctyun-autoheal/ctyun-data/accounts.json
```

## Docker Hub Token 安全说明

不要将 Docker Hub Personal Access Token 发到聊天、提交到 Git 仓库或写进镜像中。

如果 Token 已意外暴露，请立即到 Docker Hub 控制台撤销并重新生成：

<https://app.docker.com/settings/personal-access-tokens>

如不再需要从当前服务器推送 Docker 镜像，可清除本机登录状态：

```bash
docker logout
```

Docker 登录凭据通常位于 `/root/.docker/config.json`，请勿上传该文件。
