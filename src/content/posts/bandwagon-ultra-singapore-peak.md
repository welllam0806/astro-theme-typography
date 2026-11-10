---
title: 山东联通｜中国联通｜家庭宽带｜Bandwagon Ultra Singapore｜晚高峰测试
pubDate: 2026-11-10
categories: ['VPS', '网络测试']
description: 山东联通千兆直连搬瓦工 Ultra Singapore 晚高峰实测：87ms / 0% 丢包，BBR 开启后三网单线程从 1.0M 回到 225.4M，下行 4 连接峰值 985.8M。
---


> 测试对象：Bandwagon Ultra Singapore（AS25820）。家用联通千兆直连，晚高峰 21:15–23:45，含 BBR 开启前后对照。

## 基本信息

| 项目 | 信息 |
| --- | --- |
| 所在地区 | 山东（出口 IP 数据库标注） |
| 网络运营商 | 中国联通 |
| 网络类型 | 家庭宽带（千兆接入，单流实测上限约 200 Mbps） |
| 测试机房 | Ultra Singapore（新加坡） |
| 测试时间 | 晚间 21:15–23:45（北京时间） |
| 服务端拥塞控制 | 首轮 21:15–22:10 为内核默认 cubic；23:00 后开启 BBR + fq 复测 |
| 测试路径 | 公网 IPv4，不经代理 / 不经 TUN，直连 VPS |
| 测试机规格 | KVM 2 vCPU（AMD EPYC-Genoa）/ 2 GB + 1 GB Swap / 38 GB / 无 IPv6 / AS25820 IT7 |
| 出口 IP | 27.197.*.*（已打码） |

注：文中路由与日志里的 IP 均已打码；<code>*</code> 表示探针未收到回应。

## 1 Ping 延迟与丢包

| 时段与时间 | 发送·收到 | 最低 / 平均 / 最高 ms | 抖动 mdev | 丢包 |
| --- | --- | --- | --- | --- |
| 晚间 21:52（cubic 期） | 300 / 300 | 86.245 / 87.232 / 90.885 | 0.911 | 0% |
| 晚间 23:12（开启 BBR 后） | 300 / 300 | 86.403 / 87.486 / 93.949 | 1.173 | 0% |
| 反向 23:20（VPS → 家宽，20 包） | 20 / 20 | — / 86.6 / — | — | 0% |

开启 BBR 前后延迟基本一致（87 ms 量级），说明拥塞控制不影响空载 RTT；差异体现在重传与吞吐上。

## 2 去程 / 回程路由

**去程（家宽 → 新加坡）**

```text
Start: 2026-11-10T09:12:23-0500
HOST: debian12                    Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 192.168.*.1               0.0%    20    0.2   0.2   0.2   0.3   0.0
  2.|-- 27.197.144.1               0.0%    20    3.8   4.3   2.7   9.6   2.0
  3.|-- 218.59.250.5              95.0%    20    5.9   5.9   5.9   5.9   0.0
  4.|-- ???                       100.0    20    0.0   0.0   0.0   0.0   0.0
  5.|-- ???                       100.0    20    0.0   0.0   0.0   0.0   0.0
  6.|-- 219.158.22.114            95.0%    20   43.6  43.6  43.6  43.6   0.0
  7.|-- 219.158.32.22              0.0%    20   42.4  42.5  41.7  44.9   0.7
  8.|-- 210.14.161.65              0.0%    20   40.5  40.4  39.7  41.0   0.3
  9.|-- 218.105.2.170              0.0%    20   42.5  42.9  42.4  43.4   0.3
 10.|-- 218.105.11.90              0.0%    20   50.2  50.0  49.5  51.0   0.4
 11.|-- 202.77.23.158             75.0%    20   50.5  50.5  49.9  51.3   0.6
 12.|-- 43.255.170.249             0.0%    20   85.9  86.5  85.7  88.1   0.6
 13.|-- ???                       100.0    20    0.0   0.0   0.0   0.0   0.0
 14.|-- 142.0.32.103              80.0%    20  135.8 140.5 135.8 146.3   5.2
 15.|-- 185.180.*.*             0.0%    20   86.7  87.0  86.5  88.1   0.4

```

**回程（新加坡 → 家宽）**

```text
Start: 2026-11-10T14:11:40+0000
HOST: cute-buzz-2.localdomain     Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 10.39.0.1                  0.0%    20   26.3  38.2  15.4 114.0  26.6
  2.|-- 10.39.1.32                 0.0%    20    0.3   0.3   0.2   0.5   0.1
  3.|-- 142.0.32.62                0.0%    20    0.7   2.5   0.7  31.4   6.9
  4.|-- 203.144.106.89             0.0%    20    1.1   4.2   0.9  24.8   6.3
  5.|-- 69.194.166.245             0.0%    20    1.0   1.0   0.8   1.2   0.1
  6.|-- 69.194.169.245             0.0%    20    2.2   4.3   1.9  17.7   3.9
  7.|-- 203.22.180.98             30.0%    20   39.1  39.0  38.9  39.1   0.1
  8.|-- 59.43.250.49              40.0%    20   42.0  42.2  41.9  43.7   0.5
  9.|-- 59.43.22.34               40.0%    20   44.0  44.0  43.9  44.0   0.0
 10.|-- 59.43.46.69               35.0%    20   78.2  78.2  78.1  78.3   0.1
 11.|-- 219.158.38.249             0.0%    20   83.5  77.7  76.3  83.9   2.8
 12.|-- 219.158.96.185            95.0%    20   77.9  77.9  77.9  77.9   0.0
 13.|-- ???                       100.0    20    0.0   0.0   0.0   0.0   0.0
 14.|-- ???                       100.0    20    0.0   0.0   0.0   0.0   0.0
 15.|-- 61.179.17.198              0.0%    20   83.8  84.2  83.8  85.1   0.3
 16.|-- 27.197.*.*             0.0%    20   86.6  86.6  86.2  87.0   0.2

```

回程以 59.43.*.*（CN2）与 CTGGIA 为主，山东本地落到 219.158.*.*（联通 169 骨干）后回家宽。

## 3 三网回程质量（TcpQuality 217 节点）

**小包（1.4 KB）**：零丢包 93 / 1–20% 0 / >20% 0。

| 省份 | 电信 线路·延迟·丢包 | 联通 线路·延迟·丢包 | 移动 线路·延迟·丢包 |
| --- | --- | --- | --- |
| 河北 | CTGGIA · 86 ms · 0% | CTGGIA · 89 ms · 0% | CMI · 92 ms · 0% |
| 山西 | CN2GIA · 89 ms · 0% | CTGGIA · 86 ms · 0% | CTGGIA · 83 ms · 0% |
| 辽宁 | 163 · 93 ms · 0% | 4837 · 96 ms · 0% | CTGGIA · 95 ms · 0% |
| 吉林 | CTGGIA · 98 ms · 0% | CTGGIA · 96 ms · 0% | CTGGIA · 98 ms · 0% |
| 黑龙江 | CTGGIA · 103 ms · 0% | 4837 · 107 ms · 0% | CTGGIA · 99 ms · 0% |
| 江苏 | CTGGIA · 72 ms · 0% | CTGGIA · 76 ms · 0% | CTGGIA · 74 ms · 0% |
| 浙江 | CTGGIA · 72 ms · 0% | CTGGIA · 72 ms · 0% | CTGGIA · 67 ms · 0% |
| 安徽 | CTGGIA · 76 ms · 0% | CTGGIA · 77 ms · 0% | CTGGIA · 72 ms · 0% |
| 福建 | CTGGIA · 74 ms · 0% | CTGGIA · 62 ms · 0% | CTGGIA · 65 ms · 0% |
| 江西 | 163 · 67 ms · 0% | CTGGIA · 72 ms · 0% | CMI · 68 ms · 0% |
| 山东 | 163 · 93 ms · 0% | CTGGIA · 92 ms · 0% | CTGGIA · 91 ms · 0% |
| 河南 | CTGGIA · 86 ms · 0% | CTGGIA · 74 ms · 0% | CTGGIA · 76 ms · 0% |
| 湖北 | CTGGIA · 66 ms · 0% | CTGGIA · 67 ms · 0% | CTGGIA · 66 ms · 0% |
| 湖南 | CTGGIA · 69 ms · 0% | CTGGIA · 65 ms · 0% | CMIN2 · 64 ms · 0% |
| 广东 | CTGGIA · 49 ms · 0% | CTGGIA · 50 ms · 0% | CTGGIA · 60 ms · 0% |
| 海南 | CTGGIA · 50 ms · 0% | CTGGIA · 60 ms · 0% | CTGGIA · 61 ms · 0% |
| 四川 | CTGGIA · 76 ms · 0% | CTGGIA · 78 ms · 0% | CMIN2 · 76 ms · 0% |
| 贵州 | CTGGIA · 74 ms · 0% | CTGGIA · 77 ms · 0% | CTGGIA · 75 ms · 0% |
| 云南 | 163 · 80 ms · 0% | CTGGIA · 76 ms · 0% | 163 · 75 ms · 0% |
| 陕西 | CTGGIA · 74 ms · 0% | CTGGIA · 88 ms · 0% | CTGGIA · 89 ms · 0% |
| 甘肃 | CTGGIA · 75 ms · 0% | CTGGIA · 93 ms · 0% | CTGGIA · 84 ms · 0% |
| 青海 | CTGGIA · 87 ms · 0% | CTGGIA · 93 ms · 0% | CTGGIA · 96 ms · 0% |
| 内蒙古 | CTGGIA · 90 ms · 0% | CTGGIA · 88 ms · 0% | CTGGIA · 84 ms · 0% |
| 广西 | CTGGIA · 67 ms · 0% | CTGGIA · 69 ms · 0% | CTGGIA · 60 ms · 0% |
| 西藏 | CTGGIA · 78 ms · 0% | CTGGIA · 118 ms · 0% | CTGGIA · 73 ms · 0% |
| 宁夏 | CTGGIA · 94 ms · 0% | CTGGIA · 89 ms · 0% | CTGGIA · 82 ms · 0% |
| 新疆 | CTGGIA · 75 ms · 0% | CTGGIA · 116 ms · 0% | CTGGIA · 124 ms · 0% |
| 北京 | CTGGIA · 82 ms · 0% | CTGGIA · 98 ms · 0% | CTGGIA · 82 ms · 0% |
| 天津 | CTGGIA · 88 ms · 0% | CTGGIA · 82 ms · 0% | CTGGIA · 88 ms · 0% |
| 上海 | 163 · 73 ms · 0% | CTGGIA · 72 ms · 0% | CTGGIA · 69 ms · 0% |
| 重庆 | CTGGIA · 80 ms · 0% | CTGGIA · 73 ms · 0% | CTGGIA · 75 ms · 0% |

**大包（8 KB）**：零重传 91 / 1–20% 1 / >20% 1。

| 省份 | 电信 线路·延迟·丢包 | 联通 线路·延迟·丢包 | 移动 线路·延迟·丢包 |
| --- | --- | --- | --- |
| 河北 | CTGGIA · 84 ms · 0% | 4837 · 92 ms · 0% | CTGGIA · 93 ms · 0% |
| 山西 | 163 · 89 ms · 0% | CTGGIA · 87 ms · 0% | CTGGIA · 86 ms · 0% |
| 辽宁 | CTGGIA · 95 ms · 0% | 4837 · 94 ms · 0% | CTGGIA · 95 ms · 0% |
| 吉林 | CTGGIA · 100 ms · 0% | CTGGIA · 95 ms · 0% | CTGGIA · 97 ms · 0% |
| 黑龙江 | CTGGIA · 104 ms · 0% | 4837 · 108 ms · 0% | 163 · 100 ms · 0% |
| 江苏 | 163 · 73 ms · 0% | CTGGIA · 78 ms · 0% | CTGGIA · 75 ms · 0% |
| 浙江 | CTGGIA · 72 ms · 0% | 4837 · 72 ms · 0% | CTGGIA · 68 ms · 0% |
| 安徽 | CTGGIA · 75 ms · 0% | CTGGIA · 77 ms · 0% | CTGGIA · 73 ms · 0% |
| 福建 | 163 · 76 ms · 0% | 4837 · 65 ms · 0% | CMI · 65 ms · 0% |
| 江西 | 163 · 69 ms · 0% | CTGGIA · 72 ms · 0% | CMI · 65 ms · 0% |
| 山东 | CTGGIA · 93 ms · 0% | CTGGIA · 94 ms · 0% | CTGGIA · 90 ms · 0% |
| 河南 | 163 · 83 ms · 0% | CTGGIA · 72 ms · 0% | CTGGIA · 74 ms · 0% |
| 湖北 | CTGGIA · 65 ms · 0% | CTGGIA · 64 ms · 0% | CTGGIA · 67 ms · 0% |
| 湖南 | CTGGIA · 68 ms · 0% | 4837 · 64 ms · 0% | CMIN2 · 66 ms · 0% |
| 广东 | CTGGIA · 49 ms · 0% | CTGGIA · 64 ms · 0% | CTGGIA · 69 ms · 0% |
| 海南 | CTGGIA · 49 ms · 0% | 4837 · 60 ms · 0% | 163 · 71 ms · 0% |
| 四川 | CTGGIA · 78 ms · 0% | 4837 · 77 ms · 0% | CMIN2 · 78 ms · 0% |
| 贵州 | 163 · 76 ms · 0% | 4837 · 76 ms · 4% | 163 · 76 ms · 0% |
| 云南 | CTGGIA · 80 ms · 0% | 4837 · 74 ms · 0% | CTGGIA · 76 ms · 0% |
| 陕西 | CTGGIA · 74 ms · 0% | 4837 · 89 ms · 0% | CTGGIA · 91 ms · 0% |
| 甘肃 | CTGGIA · 75 ms · 0% | 4837 · 91 ms · 0% | CTGGIA · 83 ms · 0% |
| 青海 | 163 · 88 ms · 0% | CTGGIA · 97 ms · 0% | CTGGIA · 96 ms · 0% |
| 内蒙古 | 163 · 91 ms · 0% | CTGGIA · 88 ms · 0% | CTGGIA · 79 ms · 0% |
| 广西 | CTGGIA · 68 ms · 0% | 4837 · 71 ms · 0% | CTGGIA · 61 ms · 0% |
| 西藏 | CTGGIA · 78 ms · 0% | CTGGIA · 122 ms · 0% | 163 · 75 ms · 0% |
| 宁夏 | CTGGIA · 92 ms · 0% | 4837 · 90 ms · 0% | 163 · 75 ms · 0% |
| 新疆 | CTGGIA · 77 ms · 0% | CTGGIA · 115 ms · 0% | CMI · 121 ms · 0% |
| 北京 | CTGGIA · 83 ms · 0% | CTGGIA · 103 ms · 0% | CTGGIA · 84 ms · 0% |
| 天津 | 163 · 88 ms · 0% | CTGGIA · 81 ms · 0% | CTGGIA · 80 ms · 52% |
| 上海 | CTGGIA · 71 ms · 0% | CTGGIA · 71 ms · 0% | CTGGIA · 71 ms · 0% |
| 重庆 | CTGGIA · 79 ms · 0% | CTGGIA · 71 ms · 0% | CTGGIA · 80 ms · 0% |

**UDP 回程（按运营商统计线路条数）**

| 线路 | 电信 | 联通 | 移动 |
| --- | --- | --- | --- |
| CTGGIA | 29 | 30 | 23 |
| 4837 | 0 | 1 | 0 |
| 163 | 2 | 0 | 1 |
| CMI | 0 | 0 | 5 |
| CMIN2 | 0 | 0 | 2 |
| CN2GIA | 0 | 0 | 0 |

## 4 单线程测速与 iperf3

**iperf3（家宽 ↔ 新加坡，15 秒 + 3 秒预热，单位 Mbps）**

| 方向 | 单连接 | 4 连接 | 重传（单 / 4 连接，累计次） |
| --- | --- | --- | --- |
| 上行（家宽 → 新加坡） | 47.5 | 95.8 | 14,443 / 30,440 |
| 下行（新加坡 → 家宽） | 252.2 | 765.8（577–986） | 0 / 0 |

下行 4 连接在 BBR 下呈双峰分布，四次实测依次 576.7 / 597.6 / 934.0 / 985.8 Mbps，表中取中位 765.8。

**TcpQuality 三网单线程（23:30 后，BBR 生效）**

| 节点 | 回程重传 | 回程速度 Mbps | 去程速度 Mbps | 回程延迟 ms | 去程延迟 ms |
| --- | --- | --- | --- | --- | --- |
| 北京电信 | 0.00% | 248.9 | 126.3 | 86 | 91 |
| 北京联通 | 0.00% | 250.1 | 328.2 | 87 | 80 |
| 北京移动 | 0.00% | 225.4 | 337.2 | 94 | 93 |
| 上海电信 | 0.00% | 282.9 | 111.7 | 75 | 84 |
| 上海联通 | 0.00% | 269.6 | 292.9 | 80 | 85 |
| 上海移动 | 0.01% | 245.1 | 204.0 | 84 | 76 |
| 广东电信 | 0.00% | 419.5 | 447.6 | 129 | 51 |
| 广东联通 | 0.00% | 211.8 | 192.4 | 100 | 103 |
| 广东移动 | 0.03% | 181.1 | 138.5 | 108 | 101 |
| 国际方向 Apple | 0.00% | 2037.6 | 1263.4 | 15 | 16 |

对比开启 BBR 前：北京移动 21.97% 丢包 / 1.0 Mbps、上海移动 8.78% / 44.6 Mbps；开启后两者分别回到 225.4 / 245.1 Mbps，重传全部降至 0.00% 量级。

## 5 实际业务体验

**HTTP 首字节与整包（100 KB 随机数据，10 次）**：TTFB 0.168–0.189 s，整包 0.433–0.485 s。

```text
第 1 次  TTFB 0.170 s  总 0.441 s
第 2 次  TTFB 0.169 s  总 0.438 s
第 3 次  TTFB 0.184 s  总 0.475 s
第 4 次  TTFB 0.181 s  总 0.464 s
第 5 次  TTFB 0.186 s  总 0.485 s
第 6 次  TTFB 0.170 s  总 0.439 s
第 7 次  TTFB 0.168 s  总 0.433 s
第 8 次  TTFB 0.179 s  总 0.459 s
第 9 次  TTFB 0.189 s  总 0.485 s
第 10 次 TTFB 0.181 s  总 0.465 s
```

**流媒体与 AI 服务解锁（NodeQuality IP 质量）**

| 服务 | 状态 | 地区 | 方式 |
| --- | --- | --- | --- |
| TikTok | 解锁 | [SG] | 原生 |
| Disney+ | 解锁 | [SG] | 原生 |
| Netflix | 解锁 | [SG] | DNS |
| YouTube | 解锁 | [SG] | 原生 |
| Amazon Prime Video | 解锁 | [SG] | 原生 |
| Reddit | 屏蔽 | — | — |
| ChatGPT | 解锁 | [SG] | 原生 |

**常用网站可达性与 RTT（从新加坡发起）**

| 服务 | 域名 | RTT | 重传 |
| --- | --- | --- | --- |
| Adobe Assets | assets.adobe.com | 6.903ms | 0% |
| Amazon | www.amazon.com | 40.307ms | 0% |
| Apple iCloud | www.icloud.com | 9.303ms | 0% |
| AWS STS | sts.amazonaws.com | 212.233ms | 0% |
| ChatGPT | chatgpt.com | 8.486ms | 0% |
| Claude | claude.ai | 8.580ms | 0% |
| Cloudflare Dashboard | dash.cloudflare.com | 9.517ms | 0% |
| Discord Gateway | gateway.discord.gg | 12.844ms | 0% |
| Dropbox API | api.dropboxapi.com | 7.073ms | 0% |
| Facebook | www.facebook.com | 169.160ms | 0% |
| GitHub API | api.github.com | 8.540ms | 0% |
| GitLab | gitlab.com | 8.240ms | 0% |
| Gmail | mail.google.com | 224.947ms | 0% |
| Google Search | www.google.com | 6.270ms | 0% |
| Google Static | www.gstatic.com | 11.100ms | 0% |
| Instagram | www.instagram.com | 10.400ms | 0% |
| Microsoft Login | login.microsoftonline.com | 8.623ms | 0% |
| Netflix API | api-global.netflix.com | 178.197ms | 0% |
| NodeSeek | www.nodeseek.com | 13.554ms | 0% |
| Notion API | api.notion.com | 9.360ms | 0% |
| OpenAI API | api.openai.com | 8.104ms | 0% |
| PayPal API | api-m.paypal.com | 11.066ms | 0% |
| Reddit OAuth | oauth.reddit.com | 6.230ms | 0% |
| Slack App | app.slack.com | 11.563ms | 0% |
| Spotify Web | open.spotify.com | 5.700ms | 0% |
| Steam | store.steampowered.com | 8.833ms | 0% |
| Telegram | telegram.org | 158.380ms | 0% |
| Wikipedia | www.wikipedia.org | 14.467ms | 0% |
| X | x.com | 7.300ms | 0% |
| YouTube API | youtubei.googleapis.com | 10.893ms | 0% |
| Zoom API | api.zoom.us | 11.640ms | 0% |
| Telegram DC1-IPv4 | 149.154.175.50 | 220.087ms | 0% |
| Telegram DC2-IPv4 | 149.154.167.51 | 160.247ms | 0% |
| Telegram DC3-IPv4 | 149.154.175.100 | 220.780ms | 0% |
| Telegram DC4-IPv4 | 149.154.167.91 | 157.927ms | 0% |
| Telegram DC5-IPv4 | 149.154.171.5 | 7.120ms | 0% |
| Akamai Edge | www.akamai.com | 5.026ms | 0% |
| AWS CloudFront | d1.awsstatic.com | 7.873ms | 0% |
| CacheFly | cachefly.cachefly.net | 52.827ms | 0% |
| Cloudflare CDNJS | cdnjs.cloudflare.com | 4.466ms | 0% |
| Fastly Test | http-me.fastly.dev | 3.640ms | 0% |
| Google Hosted Libraries | ajax.googleapis.com | 6.320ms | 0% |
| jsDelivr Multi-CDN | cdn.jsdelivr.net | 5.817ms | 0% |
| UNPKG Cloudflare | unpkg.com | 7.893ms | 0% |

## 6 AI 站点页面首字节 TTFB

| 站点 | 域名 | HTTP 状态 | TTFB 秒（两次） |
| --- | --- | --- | --- |
| ChatGPT | chatgpt.com | 200 | 0.241 / 0.195 |
| Claude | claude.ai | 403（WAF） | 0.020 / 0.022 |
| Gemini | gemini.google.com | 200 | 0.087 / 0.062 |
| Google AI Studio | aistudio.google.com | 302 | 0.421 / 1.321 |
| Microsoft Copilot | copilot.microsoft.com | 301 | 0.081 / 0.022 |
| Perplexity | www.perplexity.ai | 403（WAF） | 0.027 / 0.023 |
| Grok | grok.com | 403（WAF） | 0.018 / 0.017 |
| DeepSeek | chat.deepseek.com | 202 | 0.043 / 0.016 |
| Poe | poe.com | 307 | 0.026 / 0.022 |

403 为 Cloudflare / 站点 WAF 对机房 IP 的拦截响应，302 / 301 / 307 为跳转，均非完整页面加载耗时；仅 200 的 ChatGPT / Gemini 可视为真实首字节。

## 7 原始报告

**NodeQuality · 23:00–23:40**

硬件信息

```text

                         硬件质量体检报告：185.180.*.*

                    github.com/xykt/HardwareQuality

                    bash <(curl -sL Check.Place) -H

            报告时间：2026-11-10 22:43:07 CST  脚本版本：v2026-05-21

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

一、操作系统信息

容器/虚拟化：          KVM 虚拟机

架构：                 x86_64

操作系统/内核：        Debian GNU/Linux 12 (bookworm) 6.12.88+deb13-amd64

运行时间：             0 天 2 小时 16 分钟

负载：                 0.63, 0.35, 0.22

进程：                 1 用户，104 进程，12/162 活跃/总服务

区域设置：             C, UTF-8, Etc/UTC UTC +0000

二、主板信息

BIOS：    SeaBIOS, 版本1.16.3-5.el9.alma.1

芯片组：  Intel 82371SB PIIX3 ISA [Natoma/Triton II]

          Intel 440FX - 82441FX PMC [Natoma]

网卡：    Red Hat, Inc. Virtio network device

三、CPU测评

CPU：     AMD EPYC-Genoa Processor 步进0 (25代) 32/64-bit

           ╚═ 2核心, 2线程, 2944.468MHz, 利用率3%

缓存：    L1d 64 KiB, L1i 64 KiB, L2 2 MiB, L3 32 MiB

指令集：   ✘ VT-x/AMD-V    ✔ AES-NI    ✔ AVX2    ✔ BMI1/2    ✘ EPT/NPT

Sysbench：单线程 4240.39     多线程 8212.88

GB5基准： J1900 N5105 N100 6700K 9900K 5900X 12900K 14900K 7713 7995WX

GB5单核：       |1230

GB5多核：            |2316

详细结果：browser.geekbench.com/v5/cpu/24801871

四、显卡测评

显卡：    [集显]AMD Cirrus Logic GD 5446

五、内存测评

内存：    总容量 2.0 GB,  已用 0.3 GB(15%),  可用 1.7 GB(85%)

交换：    总容量 1024 MB,  已用 18 MB(2%),  可用 1006 MB(98%)

超开指标： ✔ 气球回收     ✘ KSM 复用

Sysbench：读取 54704.0 MB/s    写入 32138.0 MB/s    延迟 135 ns

六、硬盘测评

硬盘：    数量 1,  总容量 38G,  已用容量 4.4G(11%),  可用容量 32G(89%)

测试设备：sda2(/r**t) -> DISK

Fio测试： RND4K/Q1    IOPS||RND4K/Q32   IOPS||SEQ1M/Q1    IOPS||SEQ1M/Q8    IOPS

读取：    26.9MB/s    6.9k||453MB/s     116k||1717MB/s    1.7k||11027MB/s    11k

写入：    38.5MB/s    9.9k||386MB/s      99k||2926MB/s    2.9k||9267MB/s    9.3k

七、HQ硬件加权评分

项目：      总 分          CPU           GPU          内 存         硬 盘

分数：      47973    =    27055    +     N/A     +    16257    +    4661

排名：      17.3%         27.5%          N/A          12.9%         19.4%

================================================================================

报告链接：Report.Check.Place/hardware/2B0R830OL.svg
```

IP 质量

```text

                      IP质量体检报告：185.180.*.*

                   github.com/xykt/IPQuality

                bash <(curl -sL Check.Place) -I

        报告时间：2026-11-10 22:46:41 CST  脚本版本：v2026-11-07

一、基础信息（Maxmind 数据库）

自治系统号：            AS25820

组织：                  IT7 Networks Inc

坐标：                  118°16′15″W, 34°3′5″N

地图：                  check.place/34.0515,-118.2707,15,cn

城市：                  加州, 洛杉矶, 90017

使用地：                [SG]新加坡, [NA]北美洲

注册地：                [CA]加拿大

时区：                  America/Los_Angeles

IP类型：                 广播IP

二、IP类型属性

数据库：      IPinfo    ipregistry    ipapi    IP2Location   AbuseIPDB

使用类型：     机房        机房        机房        机房        机房

公司类型：     机房        商业        机房        机房

三、风险评分

风险等级：      极低         低       中等       高         极高

IP2Location：                                                99|高风险

Scamalytics：  0|低风险

ipapi：                      1.17%|较高风险

AbuseIPDB：    0|低风险

四、风险因子

库： IP2Location ipapi ipregistry IPQS Scamalytics ipdata IPinfo DB-IP

地区：    [SG]    [SG]    [SG]     无     [SG]    [SG]    [SG]     无

代理：     是      否      否      无      否      否      否      无

Tor：      否      否      否      无      否      否      否      无

VPN：      是      否      是      无      否      无      否      无

服务器：   否      是      是      无      否      是      是      无

滥用：     否      否      否      无      否      否      无      无

机器人：   否      否      无      无      否      无      无      无

五、流媒体及AI服务解锁检测

服务商：  TikTok   Disney+  Netflix Youtube  AmazonPV  Reddit   ChatGPT

状态：     解锁     解锁     解锁     解锁     解锁     屏蔽     解锁

地区：     [SG]     [SG]     [SG]     [SG]     [SG]              [SG]

方式：     原生     原生      DNS     原生     原生              原生

六、邮局连通性及黑名单检测

本地25端口出站：可用

通信：+Gmail+Outlook+Yahoo+Apple-QQ+MailRU+AOL+GMX+MailCOM+163+Sohu-Sina

IP地址黑名单数据库：  有效 423   正常 387   已标记 36   黑名单 0

========================================================================

报告链接：Report.Check.Place/ip/MOLV7J9N6.svg
```

网络质量

```text

                         网络质量体检报告：185.180.*.*

                       github.com/xykt/NetQuality

                    bash <(curl -sL Check.Place) -N

            报告时间：2026-11-10 22:47:48 CST  脚本版本：v2026-09-01

一、BGP信息（BGP.TOOLS & HE.NET）

注册信息：          ARIN, AS25820 IT7 Networks Inc, Prefix/23

注册/修改日期：     2019-05-06 / 2019-05-06

地区：              [US]United States of America, Northern America, Americas

地址：              600 W 7TH ST STE 330

活跃邻居：          Subnet/24  159 / 256     Prefix/23  262 / 512

二、本地策略

NAT类型：         开放网络无NAT

TCP拥塞控制算法：bbr          TCP接收缓冲区（rmem）：4096 131072 6291456

队列调度算法：   fq           TCP发送缓冲区（wmem）：4096 16384 4194304

三、接入信息（*=Tier1 *=非Tier1 *=上游）

互联网交换点接入数：0         上游数量：0         对等互联数量：1

四、三网TCP大包延迟（依次为电信|联通|移动 Step=80ms） 京⣀⣀⣀⣀⣀76 ⣤⣤⣤⣤⣤105⣀⣀⣀⣀⣀77

津⣤⣤⣤⣤⣤80 ⣀⣀⣀⣀⣀79 ⣤⣤⣤⣤⣤81  冀⣀⣤⣀⣠⣤80 ⣤⣤⣤⣤⣤87 ⣤⣤⣤⣤⣤82  晋⣶⣦⣦⣦⣦148⣤⢰⢰⢰⢰142⠀⠀⠀⠀⠀0

蒙⣤⣤⣤⣤⣤85 ⣤⣤⣤⣤⣤81 ⣄⣤⣠⣠⣤81  辽⣤⣤⣤⣤⣤98 ⣤⣤⣤⣤⣤93 ⣤⣤⣤⣤⣤93  吉⣤⣤⣤⣤⣤103⣤⣤⣤⣤⣤94 ⣤⣤⣤⣤⣤92

黑⣤⣤⣤⣤⣤102⣤⣤⣤⣤⣤103⣤⣤⣤⣤⣤93  沪⣀⣀⣀⣀⣀68 ⣀⣀⢀⣀⣀68 ⢀⠀⡀⣀⣀68  苏⣀⣀⣀⣀⣀66 ⣀⣀⣀⣀⣀72 ⣀⣀⣀⣀⣀69

浙⣀⣀⣀⣀⣀68 ⣀⣀⣀⣀⣀67 ⣀⣀⣀⣀⣀63  皖⣠⣀⣀⣀⣀76 ⣀⣀⣀⣀⣀73 ⣀⣀⣀⣀⣀67  闽⣀⣀⣀⣀⣀67 ⣀⣀⣀⣀⣀58 ⣀⣀⣀⣀⣀67

赣⣀⣀⣀⣀⣀62 ⣀⣀⣀⣀⣀65 ⣀⣀⣀⣀⣀64  鲁⣤⣤⣤⣤⣤93 ⣤⣤⢠⣤⣤89 ⢠⣤⣤⣤⣤89  豫⣤⣤⣤⣤⣤83 ⣀⣀⣀⣀⣀66 ⣀⣀⣀⣀⣀70

鄂⣀⣀⣀⣀⣀63 ⣀⣀⣀⣀⣀63 ⣀⣀⣀⣀⣀60  湘⣀⣀⣀⣀⣀58 ⣀⣀⣀⣀⣀58 ⣀⣀⣀⣀⣀59  粤⣀⣀⣀⣀⣀47 ⣀⣀⣀⣀⣀55 ⣀⣀⣀⣀⣀64

桂⣀⣀⣀⣀⣀60 ⣀⣀⣀⣀⣀66 ⣀⣀⣀⣀⣀59  琼⣀⣀⣀⣀⣀42 ⣀⣀⣀⣀⣀57 ⣀⣀⣀⣀⣀72  渝⣀⣄⣀⣀⣀78 ⣀⣀⣀⣀⣀67 ⣀⣀⣀⣀⣀72

川⡀⣀⣀⣀⣀74 ⣀⡀⣀⣀⣀74 ⣀⣀⣀⣀⣀70  贵⣀⣀⣀⣀⣀71 ⣀⣀⣀⣀⣀72 ⣀⣀⣀⣀⣀73  云⣀⣀⣄⣀⣀76 ⣀⣀⣀⣀⠀68 ⣠⣀⣀⣀⣀73

藏⣀⣀⣀⣀⣀72 ⣤⣤⣤⣤⣤115⣀⣀⣀⣀⣀68  陕⣀⣀⣀⣀⣀71 ⣤⣤⣤⣄⣤84 ⣤⣤⣤⣤⣤95  甘⣀⣀⣀⣀⣀69 ⣤⣤⣤⣤⣤86 ⣤⣤⣤⣤⣤81

青⣤⣤⣤⣤⣤80 ⣤⣤⣤⣤⣤88 ⣤⣤⣤⣤⣤85  宁⣤⣤⣤⣤⣤90 ⣤⣤⣤⣤⣤86 ⣀⣀⣀⣀⣀67  新⣀⣀⣀⣀⣀70 ⣤⣤⣤⣤⣤111⣤⣤⣤⣤⣤133

五、三网回程路由（NextTrace API）

北京TCP：电信      IT7->CTGGIA || 联通      IT7->CTGGIA || 移动      IT7->CTGGIA

北京UDP：电信      IT7->CTGGIA || 联通      IT7->CTGGIA || 移动      IT7->CTGGIA

上海TCP：电信      IT7->CTGGIA || 联通      IT7->163    || 移动      IT7->CTGGIA

上海UDP：电信      IT7->CTGGIA || 联通      IT7->CTGGIA || 移动      IT7->CTGGIA

广州TCP：电信      IT7->CN2GIA || 联通      IT7->CN2GIA || 移动      IT7->163

广州UDP：电信      IT7->CTGGIA || 联通      IT7->CTGGIA || 移动      IT7->CTGGIA

六、国内测速   发送  延迟    接收  延迟||单位：Mbps ms  发送  延迟    接收  延迟

杭州电信          ERROR         ERROR  ||宁波电信          ERROR         ERROR

上海联通          ERROR         ERROR  ||香港联通          ERROR         ERROR

苏州移动          ERROR         ERROR  ||深圳移动          ERROR         ERROR

七、国际互连   延迟 发送 重传 接收 重传||单位：ms Mbps  延迟 发送 重传 接收 重传

香港      ⣀⣀⣀⣀⣀⣀ 35   723   0   854   0||东京      ⣀⣀⣀⣀⣀⣀ 65   398  10   454   0

新加坡    ⣀⣀⣀⣀⣀⣀  0  6502  1k  5924   0||悉尼      ⣤⣤⣤⣤⣤⣤ 92   237  1k   320   0

洛杉矶    ⣶⣶⣶⣶⣶⣶168   131   5   156   4||纽约      ⣶⣶⣶⣶⣶⣶204     ERROR   118   0

法兰克福  ⣤⣤⣤⣤⣤⣤147   125   0   171   0||伦敦      ⣤⣤⣤⣤⣤⣤156   145   0   173   0

阿姆斯特丹⣤⣤⣤⣤⣤⣤156   140   0   170   0||圣保罗    ⣿⣿⣿⣿⣿⣿332     ERROR     ERROR

================================================================================

报告链接：Report.Check.Place/net/20BXEVOSS.svg
```

回程路由追踪

```text

                         网络质量体检报告：185.180.*.*

                       github.com/xykt/NetQuality

                    bash <(curl -sL Check.Place) -N

            报告时间：2026-11-10 22:52:47 CST  脚本版本：v2026-09-01

五、三网回程路由（NextTrace API）

  北京 电信    IT7 -> CTGGIA

地理路径：荷兰 -> 新加坡 -> 广东 -> 北京    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS4134 -> AS4847

 2       0.57ms  10.39.*.*                                * RFC1918

 3       0.96ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net

 4       1.04ms  203.144.*.*                              * 新加坡 电信

 5       0.98ms  69.194.*.*   AS23764                     * 日本 东京都 东京 电信/CTGNet

 9      43.03ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信

11      83.91ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 北京 en.chinatelecom.com.cn 电信

13-15   79.20ms  219.141.*.*  AS4847    [CHINATELECOM-BJ] 中国 北京 中国国家网络交换中心

16      77.83ms  106.37.*.*   AS4847    [CHINANET-HN]     中国 北京 中国国家网络交换中心

  北京 联通    IT7 -> CN2GIA

地理路径：荷兰 -> 新加坡 -> 香港 -> 广东 -> 北京    自治系统路径：AS25820 -> AS4809 -> AS4134 -> AS4808

 2       0.51ms  10.39.*.*                                * RFC1918

 3       0.93ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net

 4-5     1.28ms  203.144.*.*                              * 新加坡 电信

 6       9.05ms  183.91.*.*             [APNIC-AP]        * 中国 香港 电信

 9      44.30ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信

10      46.19ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 广东 广州 en.chinatelecom.com.cn 电信

15     118.92ms  125.33.*.*   AS4808    [UNICOM-BJ]       中国 北京 中国联通 联通

16     105.01ms  61.51.*.*    AS4808                      中国 北京市 延庆 中国联通 联通

19     101.29ms  221.222.*.*  AS4808    [UNICOM-BJ]       中国 北京 海淀 联通

  北京 移动    IT7 -> CTGGIA

地理路径：荷兰 -> 新加坡 -> 广东 -> 北京    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS4134 -> AS56048

 2       0.44ms  10.39.*.*                                * RFC1918

 3       0.82ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net

 4       1.15ms  203.144.*.*                              * 新加坡 电信

 5       1.33ms  69.194.*.*   AS23764                     * 日本 东京都 东京 电信/CTGNet

 7      39.02ms  203.22.*.*   AS23764   [CTG-CN]          * 新加坡 CTGNet

 9      42.42ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信

10      44.84ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 广东 广州 en.chinatelecom.com.cn 电信

16-19   84.89ms  211.136.*.*  AS56048   [CMNET]           中国 北京 bj.10086.cn 移动

21      78.73ms  211.136.*.*  AS56048   [CMNET]           中国 北京 bj.10086.cn 移动

  上海 电信    IT7 -> CN2GIA

地理路径：荷兰 -> 新加坡 -> 广东 -> 上海    自治系统路径：AS25820 -> AS4809 -> AS4812

 2       0.36ms  10.39.*.*                                * RFC1918

 3       0.90ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net

 4-6     1.16ms  203.144.*.*                              * 新加坡 电信

 9      51.11ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信

12      69.89ms  101.95.*.*   AS4812    [CHINANET-SH]     中国 上海 chinatelecom.cn 电信

18      67.59ms  101.226.*.*  AS4812    [CHINANET-SH]     中国 上海市 chinatelecom.cn 电信

  上海 联通    IT7 -> CTGGIA

地理路径：荷兰 -> 新加坡 -> 广东 -> 上海    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS4134 -> AS140979

 2       0.46ms  10.39.*.*                                * RFC1918

 3       0.87ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net

 4-5     1.17ms  203.144.*.*                              * 新加坡 电信

 6       2.14ms  183.91.*.*             [APNIC-AP]        * 中国 香港 电信

 7      39.13ms  203.22.*.*   AS23764   [CTG-CN]          * 新加坡 CTGNet

 9      45.07ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信

10      46.31ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 广东 广州 en.chinatelecom.com.cn 电信

13      68.52ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 上海 en.chinatelecom.com.cn 电信

21      69.70ms  58.246.*.*   AS140979                    中国 上海市 chinaunicom.cn 联通

  上海 移动    IT7 -> CN2GIA

地理路径：荷兰 -> 新加坡 -> 香港 -> 广东 -> 上海    自治系统路径：AS25820 -> AS4809 -> AS4134 -> AS9808

 2       0.79ms  10.39.*.*                                * RFC1918

 3       4.25ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net

 4-5     1.24ms  203.144.*.*                              * 新加坡 电信

 6       4.51ms  183.91.*.*             [APNIC-AP]        * 中国 香港 电信

 9      41.57ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信

10      45.71ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 广东 广州 en.chinatelecom.com.cn 电信

14      68.53ms  221.183.*.*  AS9808    [CMNET]           中国 上海 chinamobileltd.com 移动

17      73.00ms  117.185.*.*  AS9808    [CMNET]           中国 上海市 chinamobileltd.com 移动

  广东 电信    IT7 -> CTGGIA

地理路径：荷兰 -> 新加坡 -> 香港 -> 广东    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS4134 -> AS58466

 2       0.29ms  10.39.*.*                                * RFC1918

 3       1.08ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net

 4       1.35ms  203.144.*.*                              * 新加坡 电信

 5       2.08ms  69.194.*.*   AS23764                     * 新加坡 电信/CTGNet

 6       2.31ms  183.91.*.*             [APNIC-AP]        * 中国 香港 电信

 9      44.97ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信

10      44.11ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 广东 广州 en.chinatelecom.com.cn 电信

13      47.53ms  113.96.*.*   AS4134    [CHINANET-GD]     中国 广东 广州 en.chinatelecom.com.cn

17      46.07ms  14.22.*.*    AS58466                     中国 广东省 广州市 chinatelecom.com.cn 电信

  广东 联通    IT7 -> CTGGIA

地理路径：荷兰 -> 新加坡 -> 广东    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS4134 -> AS4837 -> AS17816 -> AS136958

 2       0.71ms  10.39.*.*                                * RFC1918

 3       1.37ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net

 4       1.31ms  203.144.*.*                              * 新加坡 电信

 5       1.25ms  69.194.*.*   AS23764                     * 日本 东京都 东京 电信/CTGNet

 9      41.18ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信

10      43.16ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 广东 广州 en.chinatelecom.com.cn 电信

12      46.21ms  118.84.*.*   AS4134    [CHINANET-BB]     中国 北京市 en.chinatelecom.com.cn 电信

13      46.56ms  219.158.*.*  AS4837    [CU169-BACKBONE]  中国 北京市 chinaunicom.cn 联通

14      47.59ms  221.4.*.*    AS17816   [UNICOM-GD]       中国 广东 广州 China Unicom IP network China169 Guangdong province

15      49.21ms  120.80.*.*   AS17816   [APNIC-AP]        中国 广东省 广州市 中国联通/基站WiFi

17      53.33ms  122.13.*.*   AS136958                    中国 广东省 广州市 chinaunicom.cn 联通

  广东 移动    IT7 -> CTGGIA

地理路径：荷兰 -> 新加坡 -> 广东    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS4134 -> AS56040

 2       0.42ms  10.39.*.*                                * RFC1918

 3       0.93ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net

 4       1.13ms  203.144.*.*                              * 新加坡 电信

 5       2.12ms  69.194.*.*                               * 中国 香港 电信

 6       2.25ms  183.91.*.*             [APNIC-AP]        * 中国 香港 电信

 7      41.72ms  203.22.*.*   AS23764   [CTG-CN]          * 新加坡 CTGNet

 9      45.61ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信

10      44.66ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 广东 广州 en.chinatelecom.com.cn 电信

15-17   74.19ms  183.235.*.*  AS56040   [APNIC-AP]        中国 广东 广州 gd.10086.cn 移动

20      53.89ms  211.139.*.*  AS56040   [CMNET]           中国 广东 广州 gd.10086.cn 移动

================================================================================

报告链接：Report.Check.Place/net/1XMUNZVB0.svg
```

**TcpQuality · 22:55–23:45**

```text

  IPv4回程 统计摘要  零丢包: 93    1-20%:  0    >20%:  0

  三网概览                  电信        /               联通        /               移动
  河北             CTGGIA   86ms     0% /      CTGGIA   89ms     0% /         CMI   92ms     0%
  山西             CN2GIA   89ms     0% /      CTGGIA   86ms     0% /      CTGGIA   83ms     0%
  辽宁                163   93ms     0% /        4837   96ms     0% /      CTGGIA   95ms     0%
  吉林             CTGGIA   98ms     0% /      CTGGIA   96ms     0% /      CTGGIA   98ms     0%
  黑龙江           CTGGIA  103ms     0% /        4837  107ms     0% /      CTGGIA   99ms     0%
  江苏             CTGGIA   72ms     0% /      CTGGIA   76ms     0% /      CTGGIA   74ms     0%
  浙江             CTGGIA   72ms     0% /      CTGGIA   72ms     0% /      CTGGIA   67ms     0%
  安徽             CTGGIA   76ms     0% /      CTGGIA   77ms     0% /      CTGGIA   72ms     0%
  福建             CTGGIA   74ms     0% /      CTGGIA   62ms     0% /      CTGGIA   65ms     0%
  江西                163   67ms     0% /      CTGGIA   72ms     0% /         CMI   68ms     0%
  山东                163   93ms     0% /      CTGGIA   92ms     0% /      CTGGIA   91ms     0%
  河南             CTGGIA   86ms     0% /      CTGGIA   74ms     0% /      CTGGIA   76ms     0%
  湖北             CTGGIA   66ms     0% /      CTGGIA   67ms     0% /      CTGGIA   66ms     0%
  湖南             CTGGIA   69ms     0% /      CTGGIA   65ms     0% /       CMIN2   64ms     0%
  广东             CTGGIA   49ms     0% /      CTGGIA   50ms     0% /      CTGGIA   60ms     0%
  海南             CTGGIA   50ms     0% /      CTGGIA   60ms     0% /      CTGGIA   61ms     0%
  四川             CTGGIA   76ms     0% /      CTGGIA   78ms     0% /       CMIN2   76ms     0%
  贵州             CTGGIA   74ms     0% /      CTGGIA   77ms     0% /      CTGGIA   75ms     0%
  云南                163   80ms     0% /      CTGGIA   76ms     0% /         163   75ms     0%
  陕西             CTGGIA   74ms     0% /      CTGGIA   88ms     0% /      CTGGIA   89ms     0%
  甘肃             CTGGIA   75ms     0% /      CTGGIA   93ms     0% /      CTGGIA   84ms     0%
  青海             CTGGIA   87ms     0% /      CTGGIA   93ms     0% /      CTGGIA   96ms     0%
  内蒙古           CTGGIA   90ms     0% /      CTGGIA   88ms     0% /      CTGGIA   84ms     0%
  广西             CTGGIA   67ms     0% /      CTGGIA   69ms     0% /      CTGGIA   60ms     0%
  西藏             CTGGIA   78ms     0% /      CTGGIA  118ms     0% /      CTGGIA   73ms     0%
  宁夏             CTGGIA   94ms     0% /      CTGGIA   89ms     0% /      CTGGIA   82ms     0%
  新疆             CTGGIA   75ms     0% /      CTGGIA  116ms     0% /      CTGGIA  124ms     0%
  北京             CTGGIA   82ms     0% /      CTGGIA   98ms     0% /      CTGGIA   82ms     0%
  天津             CTGGIA   88ms     0% /      CTGGIA   82ms     0% /      CTGGIA   88ms     0%
  上海                163   73ms     0% /      CTGGIA   72ms     0% /      CTGGIA   69ms     0%
  重庆             CTGGIA   80ms     0% /      CTGGIA   73ms     0% /      CTGGIA   75ms     0%
  颜色: 正常  延迟151-240ms或1-20%重传  延迟>240ms或>20%重传，或失败

  IPv4大包回程 统计摘要
  零重传: 91    1-20%:  1    >20%:  1

  三网概览                  电信        /               联通        /               移动
  河北             CTGGIA   84ms     0% /        4837   92ms     0% /      CTGGIA   93ms     0%
  山西                163   89ms     0% /      CTGGIA   87ms     0% /      CTGGIA   86ms     0%
  辽宁             CTGGIA   95ms     0% /        4837   94ms     0% /      CTGGIA   95ms     0%
  吉林             CTGGIA  100ms     0% /      CTGGIA   95ms     0% /      CTGGIA   97ms     0%
  黑龙江           CTGGIA  104ms     0% /        4837  108ms     0% /         163  100ms     0%
  江苏                163   73ms     0% /      CTGGIA   78ms     0% /      CTGGIA   75ms     0%
  浙江             CTGGIA   72ms     0% /        4837   72ms     0% /      CTGGIA   68ms     0%
  安徽             CTGGIA   75ms     0% /      CTGGIA   77ms     0% /      CTGGIA   73ms     0%
  福建                163   76ms     0% /        4837   65ms     0% /         CMI   65ms     0%
  江西                163   69ms     0% /      CTGGIA   72ms     0% /         CMI   65ms     0%
  山东             CTGGIA   93ms     0% /      CTGGIA   94ms     0% /      CTGGIA   90ms     0%
  河南                163   83ms     0% /      CTGGIA   72ms     0% /      CTGGIA   74ms     0%
  湖北             CTGGIA   65ms     0% /      CTGGIA   64ms     0% /      CTGGIA   67ms     0%
  湖南             CTGGIA   68ms     0% /        4837   64ms     0% /       CMIN2   66ms     0%
  广东             CTGGIA   49ms     0% /      CTGGIA   64ms     0% /      CTGGIA   69ms     0%
  海南             CTGGIA   49ms     0% /        4837   60ms     0% /         163   71ms     0%
  四川             CTGGIA   78ms     0% /        4837   77ms     0% /       CMIN2   78ms     0%
  贵州                163   76ms     0% /        4837   76ms     4% /         163   76ms     0%
  云南             CTGGIA   80ms     0% /        4837   74ms     0% /      CTGGIA   76ms     0%
  陕西             CTGGIA   74ms     0% /        4837   89ms     0% /      CTGGIA   91ms     0%
  甘肃             CTGGIA   75ms     0% /        4837   91ms     0% /      CTGGIA   83ms     0%
  青海                163   88ms     0% /      CTGGIA   97ms     0% /      CTGGIA   96ms     0%
  内蒙古              163   91ms     0% /      CTGGIA   88ms     0% /      CTGGIA   79ms     0%
  广西             CTGGIA   68ms     0% /        4837   71ms     0% /      CTGGIA   61ms     0%
  西藏             CTGGIA   78ms     0% /      CTGGIA  122ms     0% /         163   75ms     0%
  宁夏             CTGGIA   92ms     0% /        4837   90ms     0% /         163   75ms     0%
  新疆             CTGGIA   77ms     0% /      CTGGIA  115ms     0% /         CMI  121ms     0%
  北京             CTGGIA   83ms     0% /      CTGGIA  103ms     0% /      CTGGIA   84ms     0%
  天津                163   88ms     0% /      CTGGIA   81ms     0% /      CTGGIA   80ms    52%
  上海             CTGGIA   71ms     0% /      CTGGIA   71ms     0% /      CTGGIA   71ms     0%
  重庆             CTGGIA   79ms     0% /      CTGGIA   71ms     0% /      CTGGIA   80ms     0%
  颜色: 正常  延迟151-240ms或1-20%重传  延迟>240ms或>20%重传，或失败

  UDP回程
                             IPv4
  三网概览         电信 /        联通 /        移动
  河北           CTGGIA /      CTGGIA /      CTGGIA
  山西           CTGGIA /      CTGGIA /         CMI
  辽宁           CTGGIA /      CTGGIA /      CTGGIA
  吉林           CTGGIA /      CTGGIA /      CTGGIA
  黑龙江         CTGGIA /        4837 /      CTGGIA
  江苏           CTGGIA /      CTGGIA /         163
  浙江           CTGGIA /      CTGGIA /         CMI
  安徽           CTGGIA /      CTGGIA /      CTGGIA
  福建              163 /      CTGGIA /         CMI
  江西           CTGGIA /      CTGGIA /         CMI
  山东              163 /      CTGGIA /      CTGGIA
  河南           CTGGIA /      CTGGIA /      CTGGIA
  湖北           CTGGIA /      CTGGIA /         CMI
  湖南           CTGGIA /      CTGGIA /       CMIN2
  广东           CTGGIA /      CTGGIA /      CTGGIA
  海南           CTGGIA /      CTGGIA /      CTGGIA
  四川           CTGGIA /      CTGGIA /       CMIN2
  贵州           CTGGIA /      CTGGIA /      CTGGIA
  云南           CTGGIA /      CTGGIA /      CTGGIA
  陕西           CTGGIA /      CTGGIA /      CTGGIA
  甘肃           CTGGIA /      CTGGIA /      CTGGIA
  青海           CTGGIA /      CTGGIA /      CTGGIA
  内蒙古         CTGGIA /      CTGGIA /      CTGGIA
  广西           CTGGIA /      CTGGIA /      CTGGIA
  西藏           CTGGIA /      CTGGIA /      CTGGIA
  宁夏           CTGGIA /      CTGGIA /      CTGGIA
  新疆           CTGGIA /      CTGGIA /      CTGGIA
  北京           CTGGIA /      CTGGIA /      CTGGIA
  天津           CTGGIA /      CTGGIA /      CTGGIA
  上海           CTGGIA /      CTGGIA /      CTGGIA
  重庆           CTGGIA /      CTGGIA /      CTGGIA

  CERNET-IPv4 统计摘要  零丢包: 30    1-20%:  0    >20%:  1

  省份概览
  河北         CTGGIA->163   91ms     0%
  山西         CTGGIA->163   87ms     0%
  辽宁              CTGGIA   96ms     0%
  吉林              CTGGIA   94ms     0%
  黑龙江       CTGGIA->163  101ms     0%
  江苏         10099->9929   71ms     0%
  浙江         10099->9929   78ms     0%
  安徽         10099->9929   66ms     0%
  福建         10099->9929   62ms     0%
  江西         CTGGIA->163   84ms     0%
  山东         10099->9929   93ms     0%
  河南         CTGGIA->163   70ms     0%
  湖北         CTGGIA->163   61ms     0%
  湖南         CTGGIA->163   62ms     0%
  广东         10099->9929   53ms     0%
  海南         10099->9929   54ms     0%
  四川         CTGGIA->163   85ms     0%
  贵州         CTGGIA->163   84ms     0%
  云南         CTGGIA->163   99ms     0%
  陕西              CTGGIA   91ms     0%
  甘肃              CTGGIA   94ms     0%
  青海              CTGGIA   96ms    36%
  内蒙古            CTGGIA   88ms     0%
  广西         10099->9929   58ms     0%
  西藏         CTGGIA->163   85ms     0%
  宁夏         10099->9929  108ms     0%
  新疆         CTGGIA->163  124ms     0%
  北京         CTGGIA->163   93ms     0%
  天津              CTGGIA   91ms     0%
  上海         10099->9929   72ms     0%
  重庆         CTGGIA->163   80ms     0%
  颜色: 正常  延迟151-240ms或1-20%重传  延迟>240ms或>20%重传，或失败

  国际节点TCP互联测试
  区域      节点-IPv4              下载延迟  下载重传   上传延迟  上传重传
  亚洲      香港                     37ms         0       38ms         0
            日本                     64ms         0       66ms         0
            新加坡                    0ms         0        0ms         0
  美洲      美国西部-洛杉矶         169ms         0      168ms         0
            美国中部-达拉斯         191ms         0      193ms         0
            美国东部-芝加哥         191ms         0      191ms         0
            加拿大-蒙特利尔         208ms         0      206ms         0
            巴西-里约热内卢         220ms         0      220ms         0
  欧洲      德国-法兰克福           149ms         0      151ms         0
            英国-伦敦               156ms         0      154ms         0
            荷兰-阿姆斯特丹         154ms         0      152ms         0
  大洋洲    澳大利亚-悉尼            92ms         0       92ms         0

  测试消耗流量<100MB，颜色: 0-100ms 正常  101-200ms 一般  >200ms 较高

  常用网站 国际互联
  服务                      域名                              可达        延迟      重传
  ------------------------  --------------------------------  ----  ----------  --------
  Adobe Assets              assets.adobe.com                  ✓        6.903ms        0%
  Amazon                    www.amazon.com                    ✓       40.307ms        0%
  Apple iCloud              www.icloud.com                    ✓        9.303ms        0%
  AWS STS                   sts.amazonaws.com                 ✓      212.233ms        0%
  ChatGPT                   chatgpt.com                       ✓        8.486ms        0%
  Claude                    claude.ai                         ✓        8.580ms        0%
  Cloudflare Dashboard      dash.cloudflare.com               ✓        9.517ms        0%
  Discord Gateway           gateway.discord.gg                ✓       12.844ms        0%
  Dropbox API               api.dropboxapi.com                ✓        7.073ms        0%
  Facebook                  www.facebook.com                  ✓      169.160ms        0%
  GitHub API                api.github.com                    ✓        8.540ms        0%
  GitLab                    gitlab.com                        ✓        8.240ms        0%
  Gmail                     mail.google.com                   ✓      224.947ms        0%
  Google Search             www.google.com                    ✓        6.270ms        0%
  Google Static             www.gstatic.com                   ✓       11.100ms        0%
  Instagram                 www.instagram.com                 ✓       10.400ms        0%
  Microsoft Login           login.microsoftonline.com         ✓        8.623ms        0%
  Netflix API               api-global.netflix.com            ✓      178.197ms        0%
  NodeSeek                  www.nodeseek.com                  ✓       13.554ms        0%
  Notion API                api.notion.com                    ✓        9.360ms        0%
  OpenAI API                api.openai.com                    ✓        8.104ms        0%
  PayPal API                api-m.paypal.com                  ✓       11.066ms        0%
  Reddit OAuth              oauth.reddit.com                  ✓        6.230ms        0%
  Slack App                 app.slack.com                     ✓       11.563ms        0%
  Spotify Web               open.spotify.com                  ✓        5.700ms        0%
  Steam                     store.steampowered.com            ✓        8.833ms        0%
  Telegram                  telegram.org                      ✓      158.380ms        0%
  Wikipedia                 www.wikipedia.org                 ✓       14.467ms        0%
  X                         x.com                             ✓        7.300ms        0%
  YouTube API               youtubei.googleapis.com           ✓       10.893ms        0%
  Zoom API                  api.zoom.us                       ✓       11.640ms        0%
  Telegram DC1-IPv4         149.154.175.50                    ✓      220.087ms        0%
  Telegram DC2-IPv4         149.154.167.51                    ✓      160.247ms        0%
  Telegram DC3-IPv4         149.154.175.100                   ✓      220.780ms        0%
  Telegram DC4-IPv4         149.154.167.91                    ✓      157.927ms        0%
  Telegram DC5-IPv4         149.154.171.5                     ✓        7.120ms        0%
  颜色: 0-100ms 正常  101-200ms 一般  >200ms 较高

  常用 CDN 国际互联
  服务                      域名                              可达        延迟      重传
  ------------------------  --------------------------------  ----  ----------  --------
  Akamai Edge               www.akamai.com                    ✓        5.026ms        0%
  AWS CloudFront            d1.awsstatic.com                  ✓        7.873ms        0%
  CacheFly                  cachefly.cachefly.net             ✓       52.827ms        0%
  Cloudflare CDNJS          cdnjs.cloudflare.com              ✓        4.466ms        0%
  Fastly Test               http-me.fastly.dev                ✓        3.640ms        0%
  Google Hosted Libraries   ajax.googleapis.com               ✓        6.320ms        0%
  jsDelivr Multi-CDN        cdn.jsdelivr.net                  ✓        5.817ms        0%
  UNPKG Cloudflare          unpkg.com                         ✓        7.893ms        0%
  颜色: 0-2ms 正常  2-10ms 一般  >10ms 较高

单线程测速

          IPv4    回程重传      回程速度      去程速度    回程延迟    去程延迟
      北京电信       0.00%     248.9Mbps     126.3Mbps        86ms        91ms
      北京联通       0.00%     250.1Mbps     328.2Mbps        87ms        80ms
      北京移动       0.00%     225.4Mbps     337.2Mbps        94ms        93ms

          IPv4    回程重传      回程速度      去程速度    回程延迟    去程延迟
      上海电信       0.00%     282.9Mbps     111.7Mbps        75ms        84ms
      上海联通       0.00%     269.6Mbps     292.9Mbps        80ms        85ms
      上海移动       0.01%     245.1Mbps     204.0Mbps        84ms        76ms

          IPv4    回程重传      回程速度      去程速度    回程延迟    去程延迟
      广东电信       0.00%     419.5Mbps     447.6Mbps       129ms        51ms
      广东联通       0.00%     211.8Mbps     192.4Mbps       100ms       103ms
      广东移动       0.03%     181.1Mbps     138.5Mbps       108ms       101ms

      国际方向    下载重传      下载速度      上传速度    下载延迟    上传延迟
    Apple IPv4       0.00%    2037.6Mbps    1263.4Mbps        15ms        16ms

  注：代理速度由回程速度+下载速度的短板决定。

  报告链接：tcpquality.ibsgss.uk/r/L2lBcsszDz

```

## 8 总体评价

延迟稳定在 87 ms 量级，家宽 → 新加坡这一段没有绕路；217 个探测点小包零丢包 93 个、大包仅 2 个点出现重传，回程线路以 CTGGIA / CN2 为主，对山东联通属于友好路径。

性能上最大的变量不是线路而是拥塞控制：默认 cubic 下，TcpQuality 测到北京移动 21.97% 丢包、单线程仅 1.0 Mbps，上海移动 8.78% / 44.6 Mbps；开启 BBR + fq 后这些高延迟路径分别回到 225.4 / 245.1 Mbps，重传几乎归零，iperf3 下行 4 连接最好到 985.8 Mbps。

上行仍受家宽本身限制（单线程 47.5 Mbps、4 线程 95.8 Mbps），与 VPS 无关；这也解释了为什么这类三网优化 VPS 的“单线程 160–170 Mbps”属于正常水平。

本次 NodeQuality 的国内测速 6 个节点全部返回 ERROR（对端 iperf3 未响应），因此国内带宽以 TcpQuality 单线程结果为准。

## 9 公开平台发布链接

| 平台 | 链接 |
| --- | --- |
| md5.pw | md5.pw/index.php?title=山东联通｜中国联通｜家庭宽带｜Bandwagon Ultra Singapore｜晚高峰测试 |
| 个人博客 | blog.wikihub.ch/posts/bandwagon-ultra-singapore-peak/ |
