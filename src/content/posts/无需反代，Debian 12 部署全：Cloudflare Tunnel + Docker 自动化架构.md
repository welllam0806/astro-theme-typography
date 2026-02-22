```
title: 无需反代，Debian 12 部署全：Cloudflare Tunnel + Docker 自动化架构
pubDate: 2026-02-22
categories: ['docker']
description: 'No reverse proxy needed, complete Debian 12 deployment: Cloudflare Tunnel + Docker automated architecture'
slug: Debian 12 deployment: Cloudflare Tunnel + Docker automated architecture
```

这份教程集成了 **Cloudflare Tunnel（安全穿透）**、**Docker 整合网络（高效通信）** 以及 **Watchtower（自动运维）**。它将确保您的服务在“全隐身”状态下安全运行，且无需手动维护镜像更新。

---
## 一、 获取 Cloudflare Tunnel Token

在操作服务器前，请先获取连接凭证：

1. 登录 [Cloudflare Zero Trust 控制台](https://one.dash.cloudflare.com/)。
2. 进入 **Networks** -> **Tunnels**，点击 **Create a Tunnel**。
![](https://tuku-blog.0bcd6be8976b1c167d72be2963555c8b.r2.cloudflarestorage.com/2026-02-22)
3. 选择 **Cloudflared** 并命名（如 `my-debian`）。
![](https://tuku-blog.0bcd6be8976b1c167d72be2963555c8b.r2.cloudflarestorage.com/PixPin_2026-02-22_20-14-35)
![](https://tuku-blog.0bcd6be8976b1c167d72be2963555c8b.r2.cloudflarestorage.com/PixPin_2026-02-22_20-15-10)
![](https://tuku-blog.0bcd6be8976b1c167d72be2963555c8b.r2.cloudflarestorage.com/PixPin_2026-02-22_20-16-27)
4. 在 **Install and run a connector** 页面选择 **Docker**。
5. 复制展示的代码末尾 `--token` 之后的那一串长字符串，这就是你的 **CLOUDFLARE_TOKEN**。
![](https://tuku-blog.0bcd6be8976b1c167d72be2963555c8b.r2.cloudflarestorage.com/PixPin_2026-02-22_20-17-22..)

---

## 二、 服务器端部署步骤

### 1. 创建项目及持久化目录

我们统一在 `/opt/cloudflared-tunnel` 下管理，并以容器名创建数据文件夹：

```bash
# 创建主项目目录
sudo mkdir -p /opt/cloudflared-tunnel && cd /opt/cloudflared-tunnel

# 创建以容器命名的持久化文件夹
sudo mkdir -p clipcascade ms-ra-forwarder

# 赋予权限确保容器可读写
sudo chmod -R 777 clipcascade ms-ra-forwarder
```

### 2. 编写整合版 `docker-compose.yml`

执行 `sudo nano docker-compose.yml`，粘贴以下内容（**请替换 Token 和密码**）：

```yaml
version: '3.8'

services:
  # 1. Cloudflare 隧道：安全入站流量网关
  tunnel:
    container_name: cloudflared-tunnel
    image: cloudflare/cloudflared:latest
    restart: always
    environment:
      - TUNNEL_TOKEN=你的_CLOUDFLARE_TOKEN_在此
    command: tunnel --no-autoupdate run
    networks:
      - cloudflare_net

  # 2. ClipCascade：剪贴板同步服务
  clipcascade:
    image: sathvikrao/clipcascade:latest
    container_name: clipcascade
    restart: always
    environment:
      - CC_USERNAME=admin
      - CC_PASSWORD=你的密码
      - CC_MAX_MESSAGE_SIZE_IN_MiB=25 # 设置消息大小限制，防止滥用
      - CC_P2P_ENABLED=true # 启用 P2P 模式，提升性能
      - CC_SIGNUP_ENABLED=false # 禁止注册，保持安全
    volumes:
      - /opt/cloudflared-tunnel/clipcascade:/app/data # 数据持久化
    networks:
      - cloudflare_net

  # 3. MS-RA-Forwarder：TTS 转发服务
  ms-ra-forwarder:
    container_name: ms-ra-forwarder
    image: wxxxcxx/ms-ra-forwarder:latest
    restart: unless-stopped
    environment:
      - TOKEN=你的安全令牌
    volumes:
      - /opt/cloudflared-tunnel/ms-ra-forwarder:/app/storage # 配置持久化
    networks:
      - cloudflare_net

  # 4. Watchtower：每天凌晨 4 点自动更新所有镜像
  watchtower:
    container_name: watchtower
    image: containrrr/watchtower:latest
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_SCHEDULE=0 0 4 * * *
      - TZ=Asia/Shanghai
      - DOCKER_API_VERSION=1.44
     

networks:
  # 显式命名网络，防止 Docker 自动添加文件夹前缀
  cloudflare_net:
    name: cloudflare_net 
    driver: bridge
```

### 3. 启动服务

```bash
sudo docker compose up -d
```

---

## 三、 Cloudflare 路由填表指南

在 Cloudflare Zero Trust 控制台的 **Public Hostname** 页面点击 **Add a public hostname**：

### 映射 1：ClipCascade


|             |               |                       |
| ---------------- | ------------------ | -------------------------- |
| **字段**           | **填写值**            | **说明**                     |
| **Subdomain**    | `clip`             | 最终通过 `clip.example.com` 访问 |
| **Domain**       | `example.com`      | 选择你的主域名                    |
| **Path**         | **留空**             | 请确保此处没有任何字符                |
| **Service Type** | `HTTP`             | 协议类型                       |
| **URL**          | `clipcascade:8080` | **必须填写容器名 + 内部端口**         |


### 映射 2：MS-RA-Forwarder


|             |                   |                      |
| ---------------- | ---------------------- | ------------------------- |
| **字段**           | **填写值**                | **说明**                    |
| **Subdomain**    | `tts`                  | 最终通过 `tts.example.com` 访问 |
| **Domain**       | `example.com`          | 选择你的主域名                   |
| **Path**         | **留空**                 | 请保持为空                     |
| **Service Type** | `HTTP`                 | 协议类型                      |
| **URL**          | `ms-ra-forwarder:3000` | **必须填写容器名 + 内部端口**        |


## ![](https://tuku-blog.0bcd6be8976b1c167d72be2963555c8b.r2.cloudflarestorage.com/PixPin_2026-02-22_20-51-09)

---

**以后添加新的docker容器，可以先执行**`docker compose down`**，让AI把新的docker容器融入到新的docker-compose.yml里面，再执行**`docker compose up -d`

**添加新的tunnel看一下下面的图片**

## ![](https://tuku-blog.0bcd6be8976b1c167d72be2963555c8b.r2.cloudflarestorage.com/PixPin_2026-02-22_20-52-56.)

---
## 四、 维护说明与安全策略

1. **验证网络状态**： 执行 `docker network inspect cloudflare_net`，确保 `Containers` 列表中包含 `tunnel`、`clipcascade` 和 `ms-ra-forwarder` 三项。
2. **验证持久化**：

进入 `/opt/cloudflared-tunnel/clipcascade` 目录，若看到生成了数据库或配置文件，说明持久化已生效。

3. **真正的全隐身**：

您现在可以安全地关闭 Debian 上的所有 Web 端口映射，因为流量是通过隧道由内向外主动建立连接的。

```bash
sudo ufw deny 80/tcp
sudo ufw deny 443/tcp
sudo ufw allow 22/tcp  # 务必保留 SSH
```

4. **日志排查**：

- 检查连接状态：`sudo docker logs -f cloudflared-tunnel`
- 检查更新记录：`sudo docker logs watchtower`

 
