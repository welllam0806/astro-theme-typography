---
title: 搭建RuskDesk服务器
pubDate: 2024-10-18
categories: ['RustDesk']
description: ''
slug: tolerance-and-freedom
---
# RustDesk Server 自建服务器部署教程

本教程详细介绍了如何在 Linux 环境下手动安装并配置 RustDesk 服务端（hbbs/hbbr）。

## 一、 环境准备与下载

首先，创建一个专门的目录用于存放 RustDesk 相关文件。

**Bash**

```
# 创建安装目录
mkdir -p /root/myApplication

# 进入该目录
cd /root/myApplication
```

使用 `wget` 下载指定版本的服务端程序（此处以 1.1.15 版本为例）：

**Bash**

```
# 下载压缩包
wget https://github.com/rustdesk/rustdesk-server/releases/download/1.1.15/rustdesk-server-linux-amd64.zip

# 解压文件
unzip rustdesk-server-linux-amd64.zip

# 重命名文件夹以便管理
mv amd64 RustDesk
```

---

## 二、 配置 Systemd 服务管理

为了确保服务稳定运行并在后台执行，我们需要将其注册为系统服务。

### 1. 创建 hbbs 服务 (ID 服务器)

**Bash**

```
nano /usr/lib/systemd/system/RustDeskHbbs.service
```

**写入以下内容：**

**Ini, TOML**

```
[Unit]
Description=RustDesk Hbbs
After=network.target

[Service]
User=root
Type=simple
WorkingDirectory=/root/myApplication/RustDesk
ExecStart=/root/myApplication/RustDesk/hbbs -k _
ExecStop=/bin/kill -TERM $MAINPID

[Install]
WantedBy=multi-user.target
```

### 2. 创建 hbbr 服务 (中继服务器)

**Bash**

nano /usr/lib/systemd/system/RustDeskHbbr.service

**写入以下内容：**

**Ini, TOML**

```
[Unit]
Description=RustDesk Hbbr
After=network.target

[Service]
User=root
Type=simple
WorkingDirectory=/root/myApplication/RustDesk
ExecStart=/root/myApplication/RustDesk/hbbs -k _
ExecStop=/bin/kill -TERM $MAINPID

[Install]
WantedBy=multi-user.target
```

---

## 三、 启动与验证

### 1. 重新加载配置并启动

**Bash**

```
# 重新加载 systemd 守护进程
systemctl daemon-reload

# 启动服务
systemctl start RustDeskHbbs
systemctl start RustDeskHbbr

# 设置开机自启 (可选)
systemctl enable RustDeskHbbs
systemctl enable RustDeskHbbr
```

### 2. 检查运行状态

**Bash**

```
systemctl status RustDeskHbbs
systemctl status RustDeskHbbr
```

---

## 四、 获取配置密钥 (Key)

RustDesk 客户端连接时需要填写 **公钥 (Key)**。该文件在服务首次启动后会自动生成在工作目录下。

**Bash**

```
# 进入程序目录
cd /root/myApplication/RustDesk

# 查看公钥内容
cat id_ed25519.pub
```

> [!TIP]
>
> **防火墙提示**：请确保您的防火墙已开放以下端口：
>
> * **TCP**: 21115, 21116, 21117, 21118, 21119
> * **UDP**: 21116
