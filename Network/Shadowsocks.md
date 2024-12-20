---
title: 基于Shadowsocks的节点搭建
date: 2024-01-30 13:00:48
tags:
- original
categories:
- Computer Network
---

## 购买并连接vps服务器

常见服务器卖家有腾讯、阿里、亚马逊，但是这些VPS比较贵。这里我们在[*Vultr*](https://www.vultr.com/)上购买*Ubuntu*服务器。我们用ssh工具[*FinalShell*](https://www.hostbuf.com/)连接我们购买的VPS。

![finalshell](https://ref.xht03.online/202412101927534.png)

## Shadowsocks原理

Shadowsocks（简称SS）是一种用于科学上网的代理工具，旨在绕过网络审查和限制。它采用 SOCKS5 协议，并使用加密算法来保护数据传输的隐私性。Shadowsocks 的工作原理是通过在本地计算机和远程服务器之间建立一个加密的隧道，将用户的网络流量通过该隧道传输。这样，用户可以访问被封锁的内容或服务，同时保护了通信的隐私性。

![Shadowsocks原理](https://ref.xht03.online/202412101927421.png)

## 在Ubuntu服务器上安装Shadowsocks

1. 命令行安装Shadowsocks：`apt install shadowsocks-libev`

2. 查看Shadowsocks状态：`systemctl status shadowsocks-libev.service`

   ![Shadowsocks状态](https://ref.xht03.online/202412101927371.png)

   - *这里：enabled表明开机自启，active表示已经启动，目前正在监听8388端口*

3. 编辑配置文件：`vim /etc/shadowsocks-libev/config.json`

   ```json
   {
       "server":[ "0.0.0.0"],
       "mode":"tcp_and_udp",
       "server_port":8388,
       "local_port":1080,
       "password":"your password",
       "timeout":86400,
       "method":"chacha20-ietf-poly1305"
   }
   ```

   - *这里我是用 vim 编辑配置文件。如果你对 vim 不熟悉，也可以直接在 FinalShell 底部的图形界	面找到 config.json 文件，双击打开，进行编辑。操作上与在 Windows 操作系统上一致。*

   - *这里把ip地址改成 0.0.0.0 ，意思是允许所有 ip 地址想 8388 端口发送数据，默认是只允许本机的程序向它发送数据。127.0.0.1是本地的环回地址。*

   - *这里的加密方式也是 AEAD ，目前最安全的加密方式。*

4. 重启Shadowsocks：`systemctl restart shadowsocks-libev.service`

5. 再查看Shadowsocks状态：`systemctl status shadowsocks-libev.service`

   ![Shadowsocks新状态](https://ref.xht03.online/202412101928793.png)

   - *这里已经是在监听 0.0.0.0 的 8388 端口，就可以进行连接了*

6. 编辑 Clash 的 .yaml 配置文件，然后在 Clash 里尝试连接：

   ```yaml
   port: 7890
   socks-port: 7891
   redir-port: 7892
   allow-lan: false
   mode: Rule
   log-level: info
   external-controller: '127.0.0.1:9090'
   secret: ''
   proxies:
     - name: "My Server"
       type: ss
       server: your vps ip
       port: 8388
       cipher: chacha20-ietf-poly1305
       password: "your password"
   proxy-groups:
     - name: Proxy
       type: select
       proxies:
         - "My Server"
   rules:
     - DOMAIN-SUFFIX,google.com,Proxy
     - MATCH,DIRECT
   ```

   - *此时会连接超时 Timeout 。这是因为 Ubuntu 服务器上的防火墙 ufw 还没有关*

7. 查看ufw状态：`ufw status`

   ![ufw](https://ref.xht03.online/202412101929201.png)

   - *ufw 默认只开放 22 号端口*

8. 开放8388端口：`ufw allow 8388`

   - *我们可以直接关掉 ufw 防火墙。但这样直接暴露在公网下很危险，所以我们选择新开一个端口 8388 。*

9. 使用Clash连接：

   ![Clash](https://ref.xht03.online/202412101929582.png)

10. 查看Ubuntu服务器日志：`journalctl -u shadowsocks-libev.service -f`

    - *到第九步为止，我们已经搭建完最简陋的 VPN 了。这条指令可以帮助你发现搭建过程中的错误。*



