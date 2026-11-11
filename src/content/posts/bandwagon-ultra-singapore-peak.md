---
title: 山东联通｜中国联通｜家庭宽带｜Bandwagon Ultra Singapore｜白天与晚高峰测试
pubDate: 2026-11-11
categories: ['VPS', '网络测试']
description: 山东联通千兆直连搬瓦工 Ultra Singapore 白天与晚高峰双时段实测：87ms / 0% 丢包；晚高峰 cubic 下北京移动单线程仅 1.0M，开 BBR 后回到 225.4M，白天三网单线程 180–420M。
---


> 测试对象：Bandwagon Ultra Singapore（AS25820）。家用联通千兆直连，白天 11:40 与晚高峰 21:15 两轮对照，含 BBR 开启前后差异。

## 基本信息

| 项目 | 信息 |
| --- | --- |
| 所在地区 | 山东（出口 IP 数据库标注） |
| 网络运营商 | 中国联通 |
| 网络类型 | 家庭宽带（千兆接入，单流实测上限约 200 Mbps） |
| 测试机房 | Ultra Singapore（新加坡），AS25820 IT7 |
| 测试时间 | 白天 11:40–13:00 / 晚间 21:15–23:45（北京时间） |
| 服务端拥塞控制 | 两轮均为 BBR + fq；晚间首轮曾用内核默认 cubic，用于对照 |
| 测试路径 | 公网 IPv4，不经代理 / 不经 TUN，直连 VPS |
| 测试机规格 | KVM 2 vCPU（AMD EPYC-Genoa）/ 2 GB + 1 GB Swap / 38 GB / 无 IPv6 |
| 出口 IP | 27.197.*.*（已打码） |

注：文中路由与日志里的 IP 均已打码；<code>*</code> 表示探针未收到回应。两轮测试使用同一台 VPS、同一条家宽、同一出口 IP，唯一差别是时段与拥塞控制。

## 1 Ping 延迟与丢包

| 时段与时间 | 发送·收到 | 最低 / 平均 / 最高 ms | 抖动 mdev | 丢包 |
| --- | --- | --- | --- | --- |
| 白天 12:50 | 300 / 300 | 86.354 / 87.394 / 91.415 | 0.985 | 0% |
| 晚间 21:52（cubic 期） | 300 / 300 | 86.245 / 87.232 / 90.885 | 0.911 | 0% |
| 晚间 23:12（开启 BBR 后） | 300 / 300 | 86.403 / 87.486 / 93.949 | 1.173 | 0% |
| 反向（VPS → 家宽，20 包） | 20 / 20 | — / 86.6 / — | — | 0% |

白天与晚间延迟都在 87 ms 量级、零丢包，说明这条路径的空载 RTT 与时段无关；开启 BBR 前后也基本一致，差异只体现在重传与吞吐上。

## 2 去程 / 回程路由（白天复测）

**去程（家宽 → 新加坡）**

```text
Start: 2026-11-10T23:09:25-0500
HOST: debian12                    Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- _gateway                     0.0%    10    0.3   0.3   0.2   0.3   0.0
  2.|-- 27.197.144.1                 0.0%    10    3.6   7.4   2.6  35.3  10.1
  3.|-- ???                         100.0    10    0.0   0.0   0.0   0.0   0.0
  4.|-- ???                         100.0    10    0.0   0.0   0.0   0.0   0.0
  5.|-- ???                         100.0    10    0.0   0.0   0.0   0.0   0.0
  6.|-- ???                         100.0    10    0.0   0.0   0.0   0.0   0.0
  7.|-- 219.158.32.22                0.0%    10   42.3  42.2  41.7  42.8   0.3
  8.|-- 210.14.161.65                0.0%    10   41.2  43.7  40.3  47.4   2.5
  9.|-- 218.105.2.170                0.0%    10   42.6  43.1  42.6  43.8   0.3
 10.|-- 218.105.11.90                0.0%    10   50.3  50.2  49.9  50.3   0.1
 11.|-- 202.77.23.158               60.0%    10   61.3  56.6  50.6  63.7   7.0
 12.|-- 43.255.170.249               0.0%    10   85.7  86.1  85.7  86.5   0.3
 13.|-- ???                         100.0    10    0.0   0.0   0.0   0.0   0.0
 14.|-- verge-103.as25820.net       50.0%    10   83.3  82.9  82.5  83.3   0.3
 15.|-- 185.180.*.*.16clouds.com  0.0%    10   86.7  87.0  86.7  87.4   0.3

```

**回程（新加坡 → 家宽）**

```text
Start: 2026-11-11T04:09:52+0000
HOST: cute-buzz-2.localdomain    Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 10.39.0.1                   0.0%    10   35.3  29.0  15.5  38.3   7.2
  2.|-- 10.39.1.32                  0.0%    10    0.4   1.8   0.3  14.8   4.6
  3.|-- sg-eqsg1-e1-1701-r.it7.net  0.0%    10    0.9   1.3   0.8   4.4   1.1
  4.|-- 203.144.106.89              0.0%    10    1.2   2.8   1.1  11.3   3.6
  5.|-- 69.194.166.245              0.0%    10    1.1   1.0   0.9   1.1   0.1
  6.|-- 69.194.169.245              0.0%    10    1.9   3.7   1.8   8.1   2.7
  7.|-- 203.22.180.98              60.0%    10   38.9  39.0  38.9  39.1   0.1
  8.|-- 59.43.250.49               80.0%    10   41.9  42.0  41.9  42.1   0.1
  9.|-- ???                        100.0    10    0.0   0.0   0.0   0.0   0.0
 10.|-- 59.43.46.69                40.0%    10   78.1  79.5  78.0  84.1   2.3
 11.|-- 219.158.38.249              0.0%    10   77.6  76.5  76.2  77.6   0.4
 12.|-- 219.158.96.185             20.0%    10   77.8  80.4  77.7  85.8   3.4
 13.|-- ???                        100.0    10    0.0   0.0   0.0   0.0   0.0
 14.|-- ???                        100.0    10    0.0   0.0   0.0   0.0   0.0
 15.|-- 61.179.17.198               0.0%    10   85.2  85.8  83.8  89.9   2.1
 16.|-- 27.197.*.*              0.0%    10   86.7  86.6  86.2  87.0   0.3

```

回程以 59.43.*.*（CN2）与 CTGGIA 为主，山东本地落到 219.158.*.*（联通 169 骨干）后回家宽。

## 3 三网回程质量（TcpQuality 217 节点，白天 vs 晚间）

**小包（1.4 KB）统计**：白天 零丢包 91 / 1–20% 2 / >20% 0；晚间 零丢包 93 / 1–20% 0 / >20% 0。

| 省份 | 白天 电信 | 白天 联通 | 白天 移动 | 晚间 电信 | 晚间 联通 | 晚间 移动 |
| --- | --- | --- | --- | --- | --- | --- |
| 河北 | CTGGIA · 86 ms · 0% | CTGGIA · 89 ms · 0% | CTGGIA · 101 ms · 0% | CTGGIA · 86 ms · 0% | CTGGIA · 89 ms · 0% | CMI · 92 ms · 0% |
| 山西 | 163 · 90 ms · 0% | 4837 · 90 ms · 0% | CTGGIA · 83 ms · 0% | CN2GIA · 89 ms · 0% | CTGGIA · 86 ms · 0% | CTGGIA · 83 ms · 0% |
| 辽宁 | CTGGIA · 102 ms · 0% | CTGGIA · 95 ms · 0% | CTGGIA · 109 ms · 4% | 163 · 93 ms · 0% | 4837 · 96 ms · 0% | CTGGIA · 95 ms · 0% |
| 吉林 | 163 · 97 ms · 0% | CTGGIA · 95 ms · 0% | CTGGIA · 99 ms · 0% | CTGGIA · 98 ms · 0% | CTGGIA · 96 ms · 0% | CTGGIA · 98 ms · 0% |
| 黑龙江 | CTGGIA · 102 ms · 0% | CTGGIA · 104 ms · 0% | CTGGIA · 99 ms · 0% | CTGGIA · 103 ms · 0% | 4837 · 107 ms · 0% | CTGGIA · 99 ms · 0% |
| 江苏 | CTGGIA · 74 ms · 0% | CTGGIA · 80 ms · 0% | CTGGIA · 74 ms · 0% | CTGGIA · 72 ms · 0% | CTGGIA · 76 ms · 0% | CTGGIA · 74 ms · 0% |
| 浙江 | CTGGIA · 72 ms · 0% | CN2GIA · 74 ms · 0% | CMI · 68 ms · 0% | CTGGIA · 72 ms · 0% | CTGGIA · 72 ms · 0% | CTGGIA · 67 ms · 0% |
| 安徽 | CTGGIA · 77 ms · 0% | CTGGIA · 80 ms · 0% | 163 · 73 ms · 0% | CTGGIA · 76 ms · 0% | CTGGIA · 77 ms · 0% | CTGGIA · 72 ms · 0% |
| 福建 | CTGGIA · 75 ms · 0% | CTGGIA · 64 ms · 0% | CN2GIA · 62 ms · 0% | CTGGIA · 74 ms · 0% | CTGGIA · 62 ms · 0% | CTGGIA · 65 ms · 0% |
| 江西 | CTGGIA · 68 ms · 0% | CTGGIA · 78 ms · 0% | CMI · 66 ms · 0% | 163 · 67 ms · 0% | CTGGIA · 72 ms · 0% | CMI · 68 ms · 0% |
| 山东 | CTGGIA · 94 ms · 0% | CTGGIA · 93 ms · 0% | CTGGIA · 90 ms · 0% | 163 · 93 ms · 0% | CTGGIA · 92 ms · 0% | CTGGIA · 91 ms · 0% |
| 河南 | CTGGIA · 82 ms · 0% | 4837 · 71 ms · 0% | CTGGIA · 72 ms · 0% | CTGGIA · 86 ms · 0% | CTGGIA · 74 ms · 0% | CTGGIA · 76 ms · 0% |
| 湖北 | CTGGIA · 65 ms · 0% | CTGGIA · 67 ms · 0% | CTGGIA · 66 ms · 0% | CTGGIA · 66 ms · 0% | CTGGIA · 67 ms · 0% | CTGGIA · 66 ms · 0% |
| 湖南 | CTGGIA · 67 ms · 0% | CTGGIA · 63 ms · 0% | CMIN2 · 66 ms · 0% | CTGGIA · 69 ms · 0% | CTGGIA · 65 ms · 0% | CMIN2 · 64 ms · 0% |
| 广东 | CTGGIA · 50 ms · 0% | CTGGIA · 50 ms · 0% | CTGGIA · 58 ms · 0% | CTGGIA · 49 ms · 0% | CTGGIA · 50 ms · 0% | CTGGIA · 60 ms · 0% |
| 海南 | 163 · 50 ms · 0% | CTGGIA · 59 ms · 0% | CN2GIA · 60 ms · 0% | CTGGIA · 50 ms · 0% | CTGGIA · 60 ms · 0% | CTGGIA · 61 ms · 0% |
| 四川 | CTGGIA · 81 ms · 0% | CTGGIA · 78 ms · 0% | CMIN2 · 78 ms · 0% | CTGGIA · 76 ms · 0% | CTGGIA · 78 ms · 0% | CMIN2 · 76 ms · 0% |
| 贵州 | CTGGIA · 76 ms · 0% | CTGGIA · 63 ms · 0% | CTGGIA · 77 ms · 0% | CTGGIA · 74 ms · 0% | CTGGIA · 77 ms · 0% | CTGGIA · 75 ms · 0% |
| 云南 | CN2GIA · 81 ms · 0% | CTGGIA · 73 ms · 0% | CTGGIA · 77 ms · 0% | 163 · 80 ms · 0% | CTGGIA · 76 ms · 0% | 163 · 75 ms · 0% |
| 陕西 | CTGGIA · 74 ms · 0% | CTGGIA · 92 ms · 0% | CN2GIA · 88 ms · 0% | CTGGIA · 74 ms · 0% | CTGGIA · 88 ms · 0% | CTGGIA · 89 ms · 0% |
| 甘肃 | CN2GIA · 75 ms · 0% | CTGGIA · 93 ms · 0% | CTGGIA · 85 ms · 0% | CTGGIA · 75 ms · 0% | CTGGIA · 93 ms · 0% | CTGGIA · 84 ms · 0% |
| 青海 | CTGGIA · 87 ms · 0% | CTGGIA · 94 ms · 0% | CTGGIA · 94 ms · 0% | CTGGIA · 87 ms · 0% | CTGGIA · 93 ms · 0% | CTGGIA · 96 ms · 0% |
| 内蒙古 | CTGGIA · 89 ms · 0% | CTGGIA · 89 ms · 0% | CTGGIA · 84 ms · 0% | CTGGIA · 90 ms · 0% | CTGGIA · 88 ms · 0% | CTGGIA · 84 ms · 0% |
| 广西 | CTGGIA · 65 ms · 0% | CTGGIA · 58 ms · 0% | CTGGIA · 59 ms · 0% | CTGGIA · 67 ms · 0% | CTGGIA · 69 ms · 0% | CTGGIA · 60 ms · 0% |
| 西藏 | CTGGIA · 77 ms · 0% | CTGGIA · 119 ms · 0% | CMI · 124 ms · 0% | CTGGIA · 78 ms · 0% | CTGGIA · 118 ms · 0% | CTGGIA · 73 ms · 0% |
| 宁夏 | CTGGIA · 93 ms · 0% | CTGGIA · 90 ms · 0% | CTGGIA · 87 ms · 0% | CTGGIA · 94 ms · 0% | CTGGIA · 89 ms · 0% | CTGGIA · 82 ms · 0% |
| 新疆 | CTGGIA · 75 ms · 0% | CN2GIA · 114 ms · 0% | CTGGIA · 122 ms · 0% | CTGGIA · 75 ms · 0% | CTGGIA · 116 ms · 0% | CTGGIA · 124 ms · 0% |
| 北京 | CTGGIA · 84 ms · 0% | CN2GIA · 82 ms · 0% | CTGGIA · 84 ms · 4% | CTGGIA · 82 ms · 0% | CTGGIA · 98 ms · 0% | CTGGIA · 82 ms · 0% |
| 天津 | CTGGIA · 88 ms · 0% | CTGGIA · 83 ms · 0% | CN2GIA · 94 ms · 0% | CTGGIA · 88 ms · 0% | CTGGIA · 82 ms · 0% | CTGGIA · 88 ms · 0% |
| 上海 | CTGGIA · 73 ms · 0% | CTGGIA · 71 ms · 0% | CTGGIA · 71 ms · 0% | 163 · 73 ms · 0% | CTGGIA · 72 ms · 0% | CTGGIA · 69 ms · 0% |
| 重庆 | CTGGIA · 80 ms · 0% | CTGGIA · 70 ms · 0% | CTGGIA · 78 ms · 0% | CTGGIA · 80 ms · 0% | CTGGIA · 73 ms · 0% | CTGGIA · 75 ms · 0% |

**大包（8 KB）统计**：白天 零重传 92 / 1–20% 1 / >20% 0；晚间 零重传 91 / 1–20% 1 / >20% 1。

| 省份 | 白天 电信 | 白天 联通 | 白天 移动 | 晚间 电信 | 晚间 联通 | 晚间 移动 |
| --- | --- | --- | --- | --- | --- | --- |
| 河北 | CTGGIA · 86 ms · 0% | 4837 · 90 ms · 0% | CTGGIA · 98 ms · 0% | CTGGIA · 84 ms · 0% | 4837 · 92 ms · 0% | CTGGIA · 93 ms · 0% |
| 山西 | 163 · 90 ms · 0% | 4837 · 89 ms · 0% | 163 · 85 ms · 0% | 163 · 89 ms · 0% | CTGGIA · 87 ms · 0% | CTGGIA · 86 ms · 0% |
| 辽宁 | 163 · 99 ms · 0% | 4837 · 94 ms · 0% | CTGGIA · 107 ms · 0% | CTGGIA · 95 ms · 0% | 4837 · 94 ms · 0% | CTGGIA · 95 ms · 0% |
| 吉林 | 163 · 98 ms · 0% | CTGGIA · 97 ms · 0% | CTGGIA · 96 ms · 0% | CTGGIA · 100 ms · 0% | CTGGIA · 95 ms · 0% | CTGGIA · 97 ms · 0% |
| 黑龙江 | Hidden · 100 ms · 0% | 4837 · 102 ms · 0% | CMI · 99 ms · 0% | CTGGIA · 104 ms · 0% | 4837 · 108 ms · 0% | 163 · 100 ms · 0% |
| 江苏 | 163 · 73 ms · 0% | CTGGIA · 80 ms · 0% | 163 · 77 ms · 0% | 163 · 73 ms · 0% | CTGGIA · 78 ms · 0% | CTGGIA · 75 ms · 0% |
| 浙江 | 163 · 73 ms · 0% | CTGGIA · 73 ms · 0% | CTGGIA · 67 ms · 0% | CTGGIA · 72 ms · 0% | 4837 · 72 ms · 0% | CTGGIA · 68 ms · 0% |
| 安徽 | 163 · 76 ms · 0% | CTGGIA · 80 ms · 0% | 163 · 74 ms · 0% | CTGGIA · 75 ms · 0% | CTGGIA · 77 ms · 0% | CTGGIA · 73 ms · 0% |
| 福建 | 163 · 73 ms · 0% | 4837 · 64 ms · 0% | 163 · 63 ms · 0% | 163 · 76 ms · 0% | 4837 · 65 ms · 0% | CMI · 65 ms · 0% |
| 江西 | 163 · 70 ms · 0% | CTGGIA · 75 ms · 0% | CMI · 64 ms · 0% | 163 · 69 ms · 0% | CTGGIA · 72 ms · 0% | CMI · 65 ms · 0% |
| 山东 | 163 · 93 ms · 0% | CN2GIA · 92 ms · 0% | CTGGIA · 90 ms · 0% | CTGGIA · 93 ms · 0% | CTGGIA · 94 ms · 0% | CTGGIA · 90 ms · 0% |
| 河南 | CTGGIA · 82 ms · 0% | CN2GIA · 72 ms · 0% | CMI · 75 ms · 0% | 163 · 83 ms · 0% | CTGGIA · 72 ms · 0% | CTGGIA · 74 ms · 0% |
| 湖北 | CTGGIA · 65 ms · 0% | 4837 · 66 ms · 0% | 163 · 65 ms · 0% | CTGGIA · 65 ms · 0% | CTGGIA · 64 ms · 0% | CTGGIA · 67 ms · 0% |
| 湖南 | CTGGIA · 65 ms · 0% | CTGGIA · 64 ms · 0% | CMIN2 · 64 ms · 0% | CTGGIA · 68 ms · 0% | 4837 · 64 ms · 0% | CMIN2 · 66 ms · 0% |
| 广东 | 163 · 49 ms · 0% | CTGGIA · 48 ms · 0% | CMI · 58 ms · 0% | CTGGIA · 49 ms · 0% | CTGGIA · 64 ms · 0% | CTGGIA · 69 ms · 0% |
| 海南 | CN2GIA · 49 ms · 0% | 4837 · 60 ms · 0% | 163 · 59 ms · 0% | CTGGIA · 49 ms · 0% | 4837 · 60 ms · 0% | 163 · 71 ms · 0% |
| 四川 | 163 · 77 ms · 0% | CTGGIA · 80 ms · 0% | CMIN2 · 78 ms · 0% | CTGGIA · 78 ms · 0% | 4837 · 77 ms · 0% | CMIN2 · 78 ms · 0% |
| 贵州 | 163 · 74 ms · 0% | CN2GIA · 64 ms · 0% | CTGGIA · 74 ms · 0% | 163 · 76 ms · 0% | 4837 · 76 ms · 4% | 163 · 76 ms · 0% |
| 云南 | CTGGIA · 81 ms · 0% | 4837 · 71 ms · 0% | 163 · 77 ms · 0% | CTGGIA · 80 ms · 0% | 4837 · 74 ms · 0% | CTGGIA · 76 ms · 0% |
| 陕西 | CTGGIA · 74 ms · 0% | CN2GIA · 90 ms · 0% | CTGGIA · 88 ms · 0% | CTGGIA · 74 ms · 0% | 4837 · 89 ms · 0% | CTGGIA · 91 ms · 0% |
| 甘肃 | CN2GIA · 74 ms · 0% | 4837 · 90 ms · 0% | CTGGIA · 84 ms · 0% | CTGGIA · 75 ms · 0% | 4837 · 91 ms · 0% | CTGGIA · 83 ms · 0% |
| 青海 | CTGGIA · 85 ms · 0% | CN2GIA · 94 ms · 0% | CTGGIA · 93 ms · 0% | 163 · 88 ms · 0% | CTGGIA · 97 ms · 0% | CTGGIA · 96 ms · 0% |
| 内蒙古 | CN2GIA · 90 ms · 0% | CN2GIA · 88 ms · 0% | CTGGIA · 81 ms · 0% | 163 · 91 ms · 0% | CTGGIA · 88 ms · 0% | CTGGIA · 79 ms · 0% |
| 广西 | CTGGIA · 65 ms · 0% | CTGGIA · 58 ms · 0% | 163 · 60 ms · 0% | CTGGIA · 68 ms · 0% | 4837 · 71 ms · 0% | CTGGIA · 61 ms · 0% |
| 西藏 | CTGGIA · 76 ms · 0% | CTGGIA · 115 ms · 0% | CTGGIA · 124 ms · 0% | CTGGIA · 78 ms · 0% | CTGGIA · 122 ms · 0% | 163 · 75 ms · 0% |
| 宁夏 | CTGGIA · 91 ms · 0% | 4837 · 91 ms · 0% | 163 · 87 ms · 0% | CTGGIA · 92 ms · 0% | 4837 · 90 ms · 0% | 163 · 75 ms · 0% |
| 新疆 | 163 · 74 ms · 0% | 4837 · 114 ms · 0% | CTGGIA · 122 ms · 0% | CTGGIA · 77 ms · 0% | CTGGIA · 115 ms · 0% | CMI · 121 ms · 0% |
| 北京 | 163 · 84 ms · 0% | CTGGIA · 82 ms · 0% | 163 · 94 ms · 4% | CTGGIA · 83 ms · 0% | CTGGIA · 103 ms · 0% | CTGGIA · 84 ms · 0% |
| 天津 | 163 · 88 ms · 0% | CN2GIA · 83 ms · 0% | CTGGIA · 85 ms · 0% | 163 · 88 ms · 0% | CTGGIA · 81 ms · 0% | CTGGIA · 80 ms · 52% |
| 上海 | 163 · 74 ms · 0% | CN2GIA · 72 ms · 0% | 163 · 71 ms · 0% | CTGGIA · 71 ms · 0% | CTGGIA · 71 ms · 0% | CTGGIA · 71 ms · 0% |
| 重庆 | CTGGIA · 82 ms · 0% | 4837 · 72 ms · 0% | 163 · 75 ms · 0% | CTGGIA · 79 ms · 0% | CTGGIA · 71 ms · 0% | CTGGIA · 80 ms · 0% |

**UDP 回程（按运营商统计线路条数，白天 / 晚间）**

| 时段 | 线路 | 电信 | 联通 | 移动 |
| --- | --- | --- | --- | --- |
| 白天 | CTGGIA | 23 | 24 | 19 |
| 白天 | 4837 | 0 | 5 | 0 |
| 白天 | 163 | 8 | 0 | 5 |
| 白天 | CMI | 0 | 0 | 5 |
| 白天 | CMIN2 | 0 | 0 | 2 |
| 白天 | CN2GIA | 0 | 0 | 0 |
| 晚间 | CTGGIA | 29 | 30 | 23 |
| 晚间 | 4837 | 0 | 1 | 0 |
| 晚间 | 163 | 2 | 0 | 1 |
| 晚间 | CMI | 0 | 0 | 5 |
| 晚间 | CMIN2 | 0 | 0 | 2 |
| 晚间 | CN2GIA | 0 | 0 | 0 |

## 4 单线程测速与 iperf3（白天 vs 晚间）

**iperf3（家宽 ↔ 新加坡，15 秒 + 3 秒预热，单位 Mbps）**

| 方向 | 白天 单 / 4 连接 | 晚间 单 / 4 连接 | 白天 重传（单 / 4） | 晚间 重传（单 / 4） |
| --- | --- | --- | --- | --- |
| 上行（家宽 → 新加坡） | 47.2 / 48.1 | 47.5 / 95.8 | 3,579 / 11,033 | 14,443 / 30,440 |
| 下行（新加坡 → 家宽） | 235.0 / 577.3 | 252.2 / 765.8（577–986） | 14 / 25,911 | 0 / 0 |

白天 4 连接下行三次实测 577.3 / 555.0 / 601.4 Mbps，重传 41,897 / 25,911 / 23,193 次；晚间同项四次 576.7 / 597.6 / 934.0 / 985.8（重传少的两次才冲到 900+）。可见这条线上行与多流吞吐的波动来自家宽侧与丢包，而不是 BBR 生效与否。

**TcpQuality 三网单线程（回程速度单位 Mbps，延迟 ms）**

| 节点 | 白天 重传 | 白天 回程 | 白天 去程 | 晚间 重传 | 晚间 回程 | 晚间 去程 |
| --- | --- | --- | --- | --- | --- | --- |
| 北京电信 | 0.00% | 237.4 | 336.0 | 0.00% | 248.9 | 126.3 |
| 北京联通 | 0.00% | 233.2 | 70.6 | 0.00% | 250.1 | 328.2 |
| 北京移动 | 0.00% | 214.9 | 322.1 | 0.00% | 225.4 | 337.2 |
| 上海电信 | 0.00% | 273.5 | 269.3 | 0.00% | 282.9 | 111.7 |
| 上海联通 | 0.49% | 189.9 | 241.8 | 0.00% | 269.6 | 292.9 |
| 上海移动 | 0.00% | 261.0 | 339.2 | 0.01% | 245.1 | 204.0 |
| 广东电信 | 0.00% | 417.6 | 475.6 | 0.00% | 419.5 | 447.6 |
| 广东联通 | 0.00% | 195.5 | 14.2 | 0.00% | 211.8 | 192.4 |
| 广东移动 | 0.00% | 403.5 | 431.8 | 0.03% | 181.1 | 138.5 |

晚间数据为开启 BBR 后的复测值；开启前的 cubic 期北京移动 21.97% 丢包 / 1.0 Mbps、上海移动 8.78% / 44.6 Mbps，是全天最差的一档。

## 5 实际业务体验

**HTTP 首字节与整包（100 KB 随机数据，10 次）**

| 时段 | TTFB 区间 | 整包区间 | 最快 TTFB | 最慢 TTFB |
| --- | --- | --- | --- | --- |
| 白天 12:52 | 0.168–0.193 s | 0.434–0.494 s | 0.168 s | 0.193 s |
| 晚间 23:2x | 0.168–0.189 s | 0.433–0.485 s | 0.168 s | 0.189 s |

```text
第 1 次   TTFB 0.175 s  总 0.454 s
第 2 次   TTFB 0.168 s  总 0.434 s
第 3 次   TTFB 0.172 s  总 0.446 s
第 4 次   TTFB 0.185 s  总 0.475 s
第 5 次   TTFB 0.193 s  总 0.494 s
第 6 次   TTFB 0.181 s  总 0.471 s
第 7 次   TTFB 0.184 s  总 0.472 s
第 8 次   TTFB 0.188 s  总 0.489 s
第 9 次   TTFB 0.171 s  总 0.442 s
第 10 次 TTFB 0.177 s  总 0.458 s
```

**流媒体与 AI 服务解锁（NodeQuality IP 质量，两轮一致）**

| 服务 | 状态 | 地区 | 方式 |
| --- | --- | --- | --- |
| TikTok | 解锁 | [SG] | 原生 |
| Disney+ | 解锁 | [SG] | 原生 |
| Netflix | 解锁 | [SG] | DNS |
| YouTube | 解锁 | [SG] | 原生 |
| Amazon Prime Video | 解锁 | [SG] | 原生 |
| Reddit | 屏蔽 | — | — |
| ChatGPT | 解锁 | [SG] | 原生 |

**常用网站可达性与 RTT（从新加坡发起，白天 / 晚间对照）**

| 服务 | 域名 | 白天 RTT | 晚间 RTT | 重传 |
| --- | --- | --- | --- | --- |
| Adobe Assets | assets.adobe.com | 7.494ms | 6.903ms | 0% |
| Amazon | www.amazon.com | 7.947ms | 40.307ms | 0% |
| Apple iCloud | www.icloud.com | 8.920ms | 9.303ms | 0% |
| AWS STS | sts.amazonaws.com | 211.733ms | 212.233ms | 0% |
| ChatGPT | chatgpt.com | 10.293ms | 8.486ms | 0% |
| Claude | claude.ai | 7.367ms | 8.580ms | 0% |
| Cloudflare Dashboard | dash.cloudflare.com | 10.733ms | 9.517ms | 0% |
| Discord Gateway | gateway.discord.gg | 8.604ms | 12.844ms | 0% |
| Dropbox API | api.dropboxapi.com | 49.220ms | 7.073ms | 0% |
| Facebook | www.facebook.com | 167.580ms | 169.160ms | 0% |
| GitHub API | api.github.com | 12.247ms | 8.540ms | 0% |
| GitLab | gitlab.com | 12.147ms | 8.240ms | 0% |
| Gmail | mail.google.com | 225.147ms | 224.947ms | 0% |
| Google Search | www.google.com | 5.373ms | 6.270ms | 0% |
| Google Static | www.gstatic.com | 11.413ms | 11.100ms | 0% |
| Instagram | www.instagram.com | 8.060ms | 10.400ms | 0% |
| Microsoft Login | login.microsoftonline.com | 76.174ms | 8.623ms | 0% |
| Netflix API | api-global.netflix.com | 178.820ms | 178.197ms | 0% |
| NodeSeek | www.nodeseek.com | 8.680ms | 13.554ms | 0% |
| Notion API | api.notion.com | 8.237ms | 9.360ms | 0% |
| OpenAI API | api.openai.com | 7.440ms | 8.104ms | 0% |
| PayPal API | api-m.paypal.com | 6.370ms | 11.066ms | 0% |
| Reddit OAuth | oauth.reddit.com | 8.857ms | 6.230ms | 0% |
| Slack App | app.slack.com | 9.767ms | 11.563ms | 0% |
| Spotify Web | open.spotify.com | 11.767ms | 5.700ms | 0% |
| Steam | store.steampowered.com | 8.967ms | 8.833ms | 0% |
| Telegram | telegram.org | 163.153ms | 158.380ms | 0% |
| Wikipedia | www.wikipedia.org | 7.533ms | 14.467ms | 0% |
| X | x.com | 7.233ms | 7.300ms | 0% |
| YouTube API | youtubei.googleapis.com | 11.063ms | 10.893ms | 0% |
| Zoom API | api.zoom.us | 5.387ms | 11.640ms | 0% |
| Telegram DC1-IPv4 | 149.154.175.50 | 220.180ms | 220.087ms | 0% |
| Telegram DC2-IPv4 | 149.154.167.51 | 160.793ms | 160.247ms | 0% |
| Telegram DC3-IPv4 | 149.154.175.100 | 219.347ms | 220.780ms | 0% |
| Telegram DC4-IPv4 | 149.154.167.91 | 159.733ms | 157.927ms | 0% |
| Telegram DC5-IPv4 | 149.154.171.5 | 7.620ms | 7.120ms | 0% |
| Akamai Edge | www.akamai.com | 4.310ms | 5.026ms | 0% |
| AWS CloudFront | d1.awsstatic.com | 4.574ms | 7.873ms | 0% |
| CacheFly | cachefly.cachefly.net | 4.094ms | 52.827ms | 0% |
| Cloudflare CDNJS | cdnjs.cloudflare.com | 6.153ms | 4.466ms | 0% |
| Fastly Test | http-me.fastly.dev | 8.256ms | 3.640ms | 0% |
| Google Hosted Libraries | ajax.googleapis.com | 224.107ms | 6.320ms | 0% |
| jsDelivr Multi-CDN | cdn.jsdelivr.net | 29.587ms | 5.817ms | 0% |
| UNPKG Cloudflare | unpkg.com | 4.024ms | 7.893ms | 0% |

## 6 AI 站点页面首字节 TTFB（白天 vs 晚间）

| 站点 | 域名 | HTTP 状态 | 白天 TTFB 秒 | 晚间 TTFB 秒 |
| --- | --- | --- | --- | --- |
| ChatGPT | chatgpt.com | 200 | 0.192096 | 0.207712 |
| Claude | claude.ai | 403 | 0.027766 | 0.023864 |
| Gemini | gemini.google.com | 200 | 0.124348 | 0.070156 |
| Google AI Studio | aistudio.google.com | 302 | 1.218573 | 1.199121 |
| Microsoft Copilot | copilot.microsoft.com | 301 | 0.055827 | 0.018423 |
| Perplexity | www.perplexity.ai | 403 | 0.040675 | 0.027877 |
| Grok | grok.com | 403 | 0.017723 | 0.021272 |
| DeepSeek | chat.deepseek.com | 403 | 0.017854 | 0.015057 |
| Poe | poe.com | 307 | 0.047289 | 0.019449 |

403 为 Cloudflare / 站点 WAF 对机房 IP 的拦截响应，302 / 301 / 307 为跳转，均非完整页面加载耗时；仅 200 的 ChatGPT / Gemini 可视为真实首字节。

## 7 原始报告（白天）

**NodeQuality · 11:47–12:05**

硬件信息

```text
                         硬件质量体检报告：185.180.*.*
            报告时间：2026-11-11 11:43:47 CST  脚本版本：v2026-05-21
一、操作系统信息
容器/虚拟化：          KVM 虚拟机
架构：                 x86_64
操作系统/内核：        Debian GNU/Linux 12 (bookworm) 6.12.88+deb13-amd64
运行时间：             0 天 15 小时 16 分钟
负载：                 0.33, 0.09, 0.03
进程：                 1 用户，102 进程，12/162 活跃/总服务
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
Sysbench：单线程 4240.50     多线程 8245.28
GB5基准： J1900 N5105 N100 6700K 9900K 5900X 12900K 14900K 7713 7995WX
GB5单核：      |1250
GB5多核：           |2358
四、显卡测评
显卡：    [集显]AMD Cirrus Logic GD 5446
五、内存测评
内存：    总容量 2.0 GB,  已用 0.3 GB(15%),  可用 1.7 GB(85%)
交换：    总容量 1024 MB,  已用 15 MB(1%),  可用 1009 MB(99%)
超开指标： ✔ 气球回收     ✘ KSM 复用
Sysbench：读取 54632.6 MB/s    写入 31604.9 MB/s    延迟 135 ns
六、硬盘测评
硬盘：    数量 1,  总容量 38G,  已用容量 4.4G(11%),  可用容量 32G(89%)
测试设备：sda2(/r**t) -> DISK
Fio测试： RND4K/Q1    IOPS||RND4K/Q32   IOPS||SEQ1M/Q1    IOPS||SEQ1M/Q8    IOPS
读取：    27.3MB/s    7.0k||446MB/s     114k||1719MB/s    1.7k||10737MB/s    11k
写入：    38.6MB/s    9.9k||367MB/s      94k||3089MB/s    3.1k||8641MB/s    8.6k
七、HQ硬件加权评分
项目：      总 分          CPU           GPU          内 存         硬 盘
分数：      48266    =    27481    +     N/A     +    16175    +    4610
排名：      17.5%         28.1%          N/A          12.8%         19.2%
报告链接：Report.Check.Place/hardware/1WENN1EPZ.svg
```

IP 质量

```text
                      IP质量体检报告：185.180.*.*
        报告时间：2026-11-11 11:47:20 CST  脚本版本：v2026-11-07
一、基础信息（Maxmind 数据库）
自治系统号：            AS25820
组织：                  IT7 Networks Inc
坐标：                  118°16′15″W, 34°3′5″N
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
IPQS：                        75|可疑IP
DB-IP：         |低风险
四、风险因子
库： IP2Location ipapi ipregistry IPQS Scamalytics ipdata IPinfo DB-IP
地区：    [SG]    [SG]    [SG]    [SG]    [SG]    [SG]    [SG]    [SG]
代理：     是      否      否      是      否      否      否      否
Tor：      否      否      否      否      否      否      否      无
VPN：      是      否      是      是      否      无      否      无
服务器：   否      是      是      无      否      是      是      无
滥用：     否      否      否      否      否      否      无      无
机器人：   否      否      无      否      否      无      无      否
五、流媒体及AI服务解锁检测
服务商：  TikTok   Disney+  Netflix Youtube  AmazonPV  Reddit   ChatGPT
状态：     解锁     解锁     解锁     解锁     解锁     屏蔽     解锁
地区：     [SG]     [SG]     [SG]     [SG]     [SG]              [SG]
方式：     原生     原生      DNS     原生     原生              原生
六、邮局连通性及黑名单检测
本地25端口出站：可用
通信：+Gmail+Outlook+Yahoo+Apple-QQ+MailRU+AOL+GMX+MailCOM+163-Sohu+Sina
IP地址黑名单数据库：  有效 423   正常 376   已标记 47   黑名单 0
报告链接：Report.Check.Place/ip/ABNAYVGW9.svg
```

网络质量

```text
                         网络质量体检报告：185.180.*.*
            报告时间：2026-11-11 11:48:29 CST  脚本版本：v2026-09-01
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
四、三网TCP大包延迟（依次为电信|联通|移动 Step=80ms） 京⣠⣀⣀⣀⣀78 ⣀⣀⣄⣀⣀77 ⣀⣠⣀⣀⣀78
津⣤⣤⣤⣤⣤80 ⣄⣀⣀⣀⣀79 ⣤⣤⣤⣤⣤81  冀⣀⣤⣀⣠⣤80 ⣤⣤⣤⣤⣤82 ⣤⣤⣤⣤⣤83  晋⣤⣦⢠⣦⣦112⣶⢰⢰⢠⢰156⠀⠀⠀⠀⠀0
蒙⣤⣤⣤⣤⣤85 ⣤⣤⣤⣤⣤81 ⣄⣤⣠⣠⣤81  辽⣤⣤⣤⣤⣤86 ⣤⣤⣤⣤⣤92 ⣤⣤⣤⣤⣤92  吉⣤⣤⣤⣤⣤104⣤⣤⣤⣤⣤94 ⣤⣤⣤⣤⣤93
黑⣤⣤⣤⣤⣤97 ⣤⣤⣤⣤⣤95 ⣤⣤⣤⣤⣤96  沪⣀⣀⣀⡀⣀69 ⡀⣀⣀⣀⡀68 ⢀⣀⢀⢀⣀68  苏⣀⣀⣀⣀⣀66 ⣀⣀⣀⣀⣀72 ⣀⣀⣀⣀⣀72
浙⣀⣀⣀⣀⣀68 ⣀⣀⣀⣀⣀67 ⣀⣀⣀⣀⣀63  皖⣀⣀⣀⣀⣀75 ⣠⣀⣀⣀⣀74 ⣀⣀⣀⣀⣀70  闽⣀⣀⣀⣀⣀67 ⣀⣀⣀⣀⣀58 ⣀⣀⣀⣀⣀64
赣⣀⣀⣀⣀⣀62 ⣀⣀⣀⣀⣀75 ⣀⣀⣀⣀⣀64  鲁⣤⣤⣤⣤⣤91 ⣤⣤⣤⣤⣤84 ⡀⣤⣄⣠⣤80  豫⣄⣀⣀⣀⣀79 ⣀⣀⣀⣀⣀66 ⣀⣀⣀⣀⣀70
鄂⣀⣀⣀⣀⣀64 ⣀⣀⣀⣀⣀63 ⣀⣀⣀⣀⣀62  湘⣀⣀⣀⣀⣀57 ⣀⣀⣀⣀⣀58 ⣀⣀⣀⣀⣀59  粤⣀⣀⣀⣀⣀47 ⣀⣀⣀⣀⣀42 ⣀⣀⣀⣀⣀48
桂⣀⣀⣀⣀⣀60 ⣀⣀⣀⣀⣀54 ⣀⣀⣀⣀⣀56  琼⣀⣀⣀⣀⣀43 ⣀⣀⣀⣀⣀57 ⣀⣀⣀⣀⣀56  渝⣀⣄⣀⣀⣀78 ⣀⣀⣀⣀⣀67 ⣀⣀⣀⣀⣀72
川⣀⣀⣀⣀⣀74 ⣀⠀⢀⣀⣀74 ⣀⣠⣀⣀⣀73  贵⣀⣀⣀⣀⣀71 ⣀⣀⣀⣀⣀62 ⣀⣀⣀⣀⣀73  云⣀⣀⣀⣄⣀76 ⣀⣀⡀⠀⠀69 ⣀⣀⣀⣀⣀69
藏⣀⣀⣀⣀⣀71 ⣤⣤⣤⣤⣤115⣤⣤⣤⣤⣤124 陕⣀⣀⣀⣀⣀71 ⣤⣤⣤⣄⣤83 ⣤⣤⣤⣤⣤88  甘⣀⣀⣀⣀⣀70 ⣤⣤⣤⣤⣤86 ⣤⣤⣤⣤⣤81
青⣤⣤⣤⣤⣤81 ⣤⣤⣤⣤⣤88 ⣤⣤⣤⣤⣤85  宁⣤⣤⣤⣤⣤90 ⣤⣤⣤⣤⣤86 ⣤⣤⣤⣤⣤83  新⣀⣀⣀⣀⣀70 ⣤⣤⣤⣤⣤111⣤⣤⣤⣤⣤126
五、三网回程路由（NextTrace API）
北京TCP：电信      IT7->CTGGIA || 联通      IT7->CTGGIA || 移动      IT7->CTGGIA
北京UDP：电信      IT7->CTGGIA || 联通      IT7->CTGGIA || 移动      IT7->CTGGIA
上海TCP：电信      IT7->CTGGIA || 联通      IT7->CTGGIA || 移动      IT7->CTGGIA
上海UDP：电信      IT7->CTGGIA || 联通      IT7->CTGGIA || 移动      IT7->CTGGIA
广州TCP：电信      IT7->CTGGIA || 联通      IT7->4837   || 移动      IT7->CTGGIA
广州UDP：电信      IT7->CTGGIA || 联通      IT7->CTGGIA || 移动      IT7->CTGGIA
六、国内测速   发送  延迟    接收  延迟||单位：Mbps ms  发送  延迟    接收  延迟
杭州电信          ERROR         ERROR  ||宁波电信          ERROR         ERROR
上海联通          ERROR         ERROR  ||香港联通          ERROR         ERROR
苏州移动          ERROR         ERROR  ||深圳移动          ERROR         ERROR
七、国际互连   延迟 发送 重传 接收 重传||单位：ms Mbps  延迟 发送 重传 接收 重传
香港      ⣀⣀⣀⣀⣀⣀ 30   723   0   860   0||东京      ⣀⣀⣀⣀⣀⣀ 65   386   1   466   0
新加坡    ⣀⣀⣀⣀⣀⣀  1  6781  28  7772   0||悉尼      ⣤⣤⣤⣤⣤⣤ 92   266   0   320   0
洛杉矶    ⣶⣶⣶⣶⣶⣶168   129   0   159   0||纽约      ⣶⣶⣶⣶⣶⣶204     ERROR     ERROR
法兰克福  ⣤⣤⣤⣤⣤⣤147   125   0   171   0||伦敦      ⣤⣤⣤⣤⣤⣤156   145   0   174   0
阿姆斯特丹⣤⣤⣤⣤⣤⣤156   138   0   169   0||圣保罗    ⣿⣿⣿⣿⣿⣿332     ERROR     ERROR
报告链接：Report.Check.Place/net/O1BQZG44O.svg
```

回程路由追踪

```text
正在运行回程路由追踪...
                         网络质量体检报告：185.180.*.*
            报告时间：2026-11-11 11:53:17 CST  脚本版本：v2026-09-01
五、三网回程路由（NextTrace API）
  北京 电信    IT7 -> 163
地理路径：荷兰 -> 新加坡 -> 香港 -> 广东 -> 北京    自治系统路径：AS25820 -> AS4134 -> AS4847
 2       0.38ms  10.39.*.*                                * RFC1918
 3       0.80ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net
 4-5     1.16ms  203.144.*.*                              * 新加坡 电信
 6       2.21ms  183.91.*.*             [APNIC-AP]        * 中国 香港 电信
10      43.12ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 广东 广州 en.chinatelecom.com.cn 电信
13-15   79.38ms  219.141.*.*  AS4847    [CHINATELECOM-BJ] 中国 北京 中国国家网络交换中心
16      79.25ms  106.37.*.*   AS4847    [CHINANET-HN]     中国 北京 中国国家网络交换中心
  北京 联通    IT7 -> CN2GT
地理路径：荷兰 -> 新加坡 -> 广东 -> 北京    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS4134 -> AS4808
 2       0.28ms  10.39.*.*                                * RFC1918
 3       0.95ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net
 4       1.21ms  203.144.*.*                              * 新加坡 电信
 5       1.16ms  69.194.*.*   AS23764                     * 日本 东京都 东京 电信/CTGNet
 9      46.28ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信
10      43.39ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 广东 广州 en.chinatelecom.com.cn 电信
15-16   74.15ms  202.96.*.*   AS4808    [UNICOM-BJ]       中国 北京 4808-Core 中国联通 联通
19      79.10ms  221.222.*.*  AS4808    [UNICOM-BJ]       中国 北京 海淀 联通
  北京 移动    IT7 -> CN2GT
地理路径：荷兰 -> 新加坡 -> 广东 -> 香港 -> 北京    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS4134 -> AS9808 -> AS56048
 2       0.95ms  10.39.*.*                                * RFC1918
 3       1.04ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net
 4       1.18ms  203.144.*.*                              * 新加坡 电信
 5       1.11ms  69.194.*.*   AS23764                     * 日本 东京都 东京 电信/CTGNet
 7      41.58ms  203.22.*.*   AS23764   [CTG-CN]          * 新加坡 CTGNet
 9      44.92ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信
10      44.27ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 广东 广州 en.chinatelecom.com.cn 电信
11      76.33ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 香港 en.chinatelecom.com.cn
12      73.17ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 北京 北京 en.chinatelecom.com.cn
14-15   73.55ms  221.183.*.*  AS9808    [CMNET]           中国 北京 chinamobileltd.com 移动
17-19   77.04ms  211.136.*.*  AS56048   [CMNET]           中国 北京 bj.10086.cn 移动
21      73.35ms  211.136.*.*  AS56048   [CMNET]           中国 北京 bj.10086.cn 移动
  上海 电信    IT7 -> CTGGIA
地理路径：荷兰 -> 新加坡 -> 广东 -> 上海    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS4134 -> AS4812
 2       2.96ms  10.39.*.*                                * RFC1918
 3       1.05ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net
 4-6     1.19ms  203.144.*.*                              * 新加坡 电信
 7      41.59ms  203.22.*.*   AS23764   [CTG-CN]          * 新加坡 CTGNet
 9      44.23ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信
11      65.69ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 上海 en.chinatelecom.com.cn 电信
12      68.08ms  101.95.*.*   AS4812    [CHINANET-SH]     中国 上海 chinatelecom.cn 电信
18      68.62ms  101.226.*.*  AS4812    [CHINANET-SH]     中国 上海市 chinatelecom.cn 电信
  上海 联通    IT7 -> CTGGIA
地理路径：荷兰 -> 新加坡 -> 广东 -> 上海    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS4134 -> AS140979
 2       0.42ms  10.39.*.*                                * RFC1918
 3       0.98ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net
 4-5     1.30ms  203.144.*.*                              * 新加坡 电信
 6       2.31ms  183.91.*.*             [APNIC-AP]        * 中国 香港 电信
 7      39.23ms  203.22.*.*   AS23764   [CTG-CN]          * 新加坡 CTGNet
 9      61.69ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信
10      43.06ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 广东 广州 en.chinatelecom.com.cn 电信
11      75.11ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 上海 en.chinatelecom.com.cn 电信
21      69.71ms  58.246.*.*   AS140979                    中国 上海市 chinaunicom.cn 联通
  上海 移动    IT7 -> CTGGIA
地理路径：荷兰 -> 新加坡 -> 广东 -> 湖北 -> 上海    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS9808
 2       0.42ms  10.39.*.*                                * RFC1918
 3       0.83ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net
 4       1.13ms  203.144.*.*                              * 新加坡 电信
 5       2.09ms  69.194.*.*   AS23764                     * 新加坡 电信/CTGNet
 7      41.60ms  203.22.*.*   AS23764   [CTG-CN]          * 新加坡 CTGNet
 9      46.16ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信
13      70.24ms  221.183.*.*  AS9808    [CMNET]           中国 湖北省 武汉市 chinamobileltd.com
14      69.94ms  221.183.*.*  AS9808    [CMNET]           中国 上海 chinamobileltd.com 移动
17      71.45ms  117.185.*.*  AS9808    [CMNET]           中国 上海市 chinamobileltd.com 移动
  广东 电信    IT7 -> CTGGIA
地理路径：荷兰 -> 新加坡 -> 广东    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS4134 -> AS58466
 2       0.62ms  10.39.*.*                                * RFC1918
 3       1.40ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net
 4       1.36ms  203.144.*.*                              * 新加坡 电信
 5       6.73ms  69.194.*.*   AS23764                     * 新加坡 电信/CTGNet
 9      45.16ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信
11-13   44.99ms  113.96.*.*   AS4134    [CHINANET-GD]     中国 广东 广州 en.chinatelecom.com.cn 电信
17      44.22ms  14.22.*.*    AS58466                     中国 广东省 广州市 chinatelecom.com.cn 电信
  广东 联通    IT7 -> CTGGIA
地理路径：荷兰 -> 新加坡 -> 广东    自治系统路径：AS25820 -> AS23764 -> AS4809 -> AS4134 -> AS17816 -> AS136958
 2       0.53ms  10.39.*.*                                * RFC1918
 3       1.02ms  142.0.*.*    AS25820                     荷兰 北荷兰省 阿姆斯特丹 it7.net
 4       1.47ms  203.144.*.*                              * 新加坡 电信
 5-6     1.83ms  69.194.*.*   AS23764                     * 新加坡 电信/CTGNet
 7      38.88ms  203.22.*.*   AS23764   [CTG-CN]          * 新加坡 CTGNet
 9      46.94ms  59.43.*.*    AS4809    [CN2-BackBone]    * 中国 广东 广州 chinatelecom.cn 电信
10      44.02ms  202.97.*.*   AS4134    [CHINANET-BB]     中国 广东 广州 en.chinatelecom.com.cn 电信
15      44.20ms  120.80.*.*   AS17816   [APNIC-AP]        中国 广东省 广州市 中国联通/基站WiFi
……（此处省略 17 行，完整内容见本节报告链接）
```

**TcpQuality · 11:58–12:45**

```text
单线程测速
          IPv4    回程重传      回程速度      去程速度    回程延迟    去程延迟
      北京电信       0.00%     237.4Mbps     336.0Mbps        95ms        93ms
      北京联通       0.00%     233.2Mbps      70.6Mbps        90ms        86ms
      北京移动       0.00%     214.9Mbps     322.1Mbps        98ms        97ms
          IPv4    回程重传      回程速度      去程速度    回程延迟    去程延迟
      上海电信       0.00%     273.5Mbps     269.3Mbps        78ms        79ms
      上海联通       0.49%     189.9Mbps     241.8Mbps        89ms        88ms
      上海移动       0.00%     261.0Mbps     339.2Mbps        81ms        87ms
          IPv4    回程重传      回程速度      去程速度    回程延迟    去程延迟
      广东电信       0.00%     417.6Mbps     475.6Mbps        55ms        55ms
      广东联通       0.00%     195.5Mbps      14.2Mbps       106ms        97ms
      广东移动       0.00%     403.5Mbps     431.8Mbps        57ms        60ms
      国际方向    下载重传      下载速度      上传速度    下载延迟    上传延迟
    Apple IPv4       0.00%    2365.4Mbps     949.4Mbps        15ms        14ms
  注：代理速度由回程速度+下载速度的短板决定。
```

## 8 原始报告（晚间，含 BBR 前）

**NodeQuality · 23:00–23:40**

硬件信息

```text
                         硬件质量体检报告：185.180.*.*
            报告时间：2026-11-10 22:43:07 CST  脚本版本：v2026-05-21
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
GB5单核：      |1230
GB5多核：           |2316
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
报告链接：Report.Check.Place/hardware/2B0R830OL.svg
```

IP 质量

```text
                      IP质量体检报告：185.180.*.*
        报告时间：2026-11-10 22:46:41 CST  脚本版本：v2026-11-07
一、基础信息（Maxmind 数据库）
自治系统号：            AS25820
组织：                  IT7 Networks Inc
坐标：                  118°16′15″W, 34°3′5″N
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
报告链接：Report.Check.Place/ip/MOLV7J9N6.svg
```

网络质量

```text
                         网络质量体检报告：185.180.*.*
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
报告链接：Report.Check.Place/net/20BXEVOSS.svg
```

回程路由追踪

```text
正在运行回程路由追踪...
                         网络质量体检报告：185.180.*.*
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
……（此处省略 15 行，完整内容见本节报告链接）
```

**TcpQuality · 22:55–23:45**

```text
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
```

## 9 总体评价

两个时段结论一致：**延迟 87 ms、零丢包、无绕路**，白天与晚高峰的差距主要体现在单线程可用带宽和重传率上，而不是 RTT。

晚高峰最能说明问题的是拥塞控制：内核默认 cubic 下 TcpQuality 测到北京移动 21.97% 丢包、单线程 1.0 Mbps，上海移动 8.78% / 44.6 Mbps；切到 BBR + fq 后同节点回到 225.4 / 245.1 Mbps，重传降到 0.00% 量级。白天开 BBR 复测，单线程三网分布在 180–420 Mbps，属于这条路径的正常上限。

iperf3 多流下行两轮合计 577–986 Mbps，上行固定受家宽限制（单线程 47 Mbps、4 线程 48–96 Mbps），与 VPS 无关；这也解释了为什么三网优化 VPS 的「单线程 160–170 Mbps」在家庭千兆下属于正常水平——瓶颈在家宽侧。

本次 NodeQuality 的国内测速 6 个节点两轮均返回 ERROR（对端 iperf3 未响应），因此国内带宽统一以 TcpQuality 单线程结果为准。

## 10 公开平台发布链接

| 平台 | 链接 |
| --- | --- |
| md5.pw | md5.pw/index.php?title=山东联通｜中国联通｜家庭宽带｜Bandwagon Ultra Singapore｜白天与晚高峰测试 |
| 个人博客 | blog.wikihub.ch/posts/bandwagon-ultra-singapore-peak/ |
| NodeQuality 白天 | Report.Check.Place/hardware/1WENN1EPZ.svg |
| NodeQuality 晚间 | Report.Check.Place/hardware/2B0R830OL.svg |
| TcpQuality 白天 | tcpquality.ibgss.uk/r/Vzjom6Dr1V |
| TcpQuality 晚间 | tcpquality.ibgss.uk/r/L2lBcsszDz |
