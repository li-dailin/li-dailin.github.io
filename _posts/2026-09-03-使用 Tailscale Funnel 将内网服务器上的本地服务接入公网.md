---
title: '使用 Tailscale Funnel 将内网服务器上的本地服务接入公网'
date: 2026-09-03
permalink: /blogs/2026/使用-Tailscale-Funnel-将内网服务器上的本地服务接入公网
excerpt: 最近部署一个比赛 Agent 项目时，使用了由学校提供、无公网入站能力的服务器，但学校并未提供域名。为了方便进行小范围内测，选用了 Tailscale Funnel 将服务器上的本地服务映射到公网，从而可以通过 Tailscale 提供的 HTTPS 域名直接访问。
tags:
  - 计算机网络
---

最近部署一个比赛 Agent 项目时，使用了由学校提供、无公网入站能力的服务器，但学校并未提供域名。为了方便进行小范围内测，选用了 Tailscale Funnel 将服务器上的本地服务映射到公网，从而可以通过 Tailscale 提供的 HTTPS 域名直接访问。

## Tailscale Funnel 介绍

传统的公网部署通常需要 `本地服务 -> Nginx -> 80/443 -> 公网 IP`    的过程。而校园服务器往往只有出网能力，没有公网入站能力，因此无法直接让互联网访问服务器。

Tailscale Funnel 采用另一种方式：服务器主动连接 Tailscale 网络，再由 Tailscale 提供公网入口，因此不需要服务器直接暴露公网端口。同时，Tailscale 会提供类似 `https://xxx.ts.net` 这样的 HTTPS 地址，所以内测阶段也不需要单独购买和配置域名。整个过程可以概括为 `本地服务 -> Tailscale 基础设施 -> Tailscale Funnel -> https://xxx.ts.net -> 公网访问`。

## 1. 检查服务器是否可以访问 Tailscale

首先确认服务器能够访问 Tailscale 的基础设施。可以测试：

```sh
curl -4 -v --connect-timeout 10 https://login.tailscale.com/
curl -4 -v --connect-timeout 10 https://controlplane.tailscale.com/
```

如果能够完成 TLS 握手并返回 HTTP 响应，说明服务器可以正常访问 Tailscale 的控制面，可以继续安装 Tailscale。

## 2. 安装 Tailscale

以 Ubuntu 为例：

```sh
curl -fsSL https://tailscale.com/install.sh | sh
```

安装完成后启动并登录：

```sh
sudo tailscale up
```

命令会输出一个登录链接，例如：

```
To authenticate, visit:

https://login.tailscale.com/a/xxxxxxx
```

使用自己的 Tailscale 账号完成认证即可。

登录完成后，可以检查节点状态：

```sh
tailscale status
tailscale ip
```

如果能够看到类似 `100.x.x.x` 的 Tailscale IP，说明服务器已经成功加入 Tailnet。

## 3. 确认本地服务

假设本地服务监听 `127.0.0.1:7860`，可以通过：

```sh
ss -lntp | grep 7860
```

确认服务正在运行。

## 4. 使用 Tailscale Funnel 暴露到公网

直接执行：

```sh
tailscale funnel --bg 7860
```

（使用 `--bg` 可以让 Funnel 在后台持续运行，而不需要一直保持当前终端会话）

如果成功，会看到类似以下提示：

```
Available on the internet:

https://xxx.ts.net/
|-- proxy http://127.0.0.1:7860

Funnel started and running in the background.
```

此时访问 `https://xxx.ts.net/` 即可进入服务器上的 `127.0.0.1:7860`。

也就是说，原本只能由服务器自己访问的本地服务，现在可以通过公网 HTTPS 地址访问。

查看状态：

```
tailscale funnel status
```

关闭 Funnel：

```
tailscale funnel --https=443 off
```
## 总结

对于“服务器可以出网，但没有公网入口，只希望临时给几个人访问一个 Web 服务”的场景，Tailscale Funnel 是一个比较方便的方案。对于开发阶段的 Demo、内测服务和临时 Web 服务，这种方式可以省掉公网 IP、端口映射和域名配置等额外工作。

