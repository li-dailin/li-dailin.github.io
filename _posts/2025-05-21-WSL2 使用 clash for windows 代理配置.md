---
title: 'WSL 代理配置'
date: 2025-05-21
permalink: /blogs/2025/WSL代理配置
excerpt: 使用 Clash for Windows 配置 WSL 代理。
tags:
  - WSL
  - Linux
  - Windows
---

1. 允许 7890 端口通过防火墙。CMD 执行：
  
   ```
   netsh advfirewall firewall add rule name="Open Port 7890" dir=in action=allow protocol=TCP localport=7890
   ```

2. `Clash for Windows - General - Allow LAN` 选择开启。

3. WSL 中设置环境变量：
  
   ```bash
   export hostip=$(ip route show | grep -i default | awk '{ print $3}')
   export https_proxy="http://${hostip}:7890"
   export http_proxy="http://${hostip}:7890"
   ```

4. 测试连接，显示 `HTTP/1.1 200 OK...` 则连接成功。
  
   ```bash
   curl --head --silent http://www.google.com
   ```
