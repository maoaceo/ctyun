# ctyun-autoheal — Docker Compose 部署

本仓库提供 **两个独立容器、每组两个账号** 的 Docker Compose 配置：

- `ctyun-autoheal-group1`：读取 `group1/accounts.json`
- `ctyun-autoheal-group2`：读取 `group2/accounts.json`

两个服务彼此独立；其中一组重启或异常时，不会影响另一组。

> `accounts.json` 含账号密码和设备码，已被 `.gitignore` 排除，**不会上传到 GitHub**。

## 1. 安装 Docker Compose（可选）

如果此命令已有版本输出，可跳过：

```bash
docker compose version
```

如需安装旧命令形式的 Compose：

```bash
sudo curl -fL "https://ghfast.top/https://github.com/docker/compose/releases/download/v2.4.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -sf /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose version
```

## 2. 下载配置并创建账号文件

```bash
git clone https://gh-proxy.org/https://github.com/maoaceo/ctyun.git
cd /opt/ctyun-autoheal
mkdir -p group1 group2
```

仓库已自带 `docker-compose.yml`。复制账号模板：

```bash
cp group1/accounts.json.example group1/accounts.json
cp group2/accounts.json.example group2/accounts.json
```

编辑两个账号文件，填写各自两个账号的手机号、密码和设备码：

```bash
nano group1/accounts.json
nano group2/accounts.json
```

账号文件格式如下：

```json
{
  "keepAliveSeconds": 60,
  "accounts": [
    {
      "name": "账号一",
      "user": "手机号",
      "password": "密码",
      "deviceCode": "设备码"
    },
    {
      "name": "账号二",
      "user": "手机号",
      "password": "密码",
      "deviceCode": "设备码"
    }
  ]
}
```

保护含敏感信息的文件：

```bash
chmod 700 group1 group2
chmod 600 group1/accounts.json group2/accounts.json
```

## 3. 启动

```bash
docker compose up -d
```

查看状态：

```bash
docker compose ps
```

查看日志：

```bash
docker compose logs -f
```

查看各组健康状态：

```bash
docker inspect --format '{{json .State.Health}}' ctyun-autoheal-group1
docker inspect --format '{{json .State.Health}}' ctyun-autoheal-group2
```

## 4. 常用管理命令

```bash
# 重启全部服务
docker compose restart

# 停止并删除容器（不会删除 group1/ 和 group2/ 中的数据）
docker compose down

# 拉取新镜像并重新创建容器
docker compose pull
docker compose up -d
```

## Compose 配置说明

`docker-compose.yml` 已包含：

- 镜像标签：`lusean23/ctyun-autoheal:latest`
- 容器自动重启：`unless-stopped`
- 独立数据目录：`./group1:/app/data`、`./group2:/app/data`
- 每 60 秒健康检查

## 安全提示

请勿把 Docker Hub Token、`accounts.json`、密码或设备码提交到 Git 仓库或公开发送。若 Token 泄露，请立即在 [Docker Hub Token 设置](https://app.docker.com/settings/personal-access-tokens)撤销并重新生成。
