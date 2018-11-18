---
title: "Install Nginx on Ubuntu16.04"
date: 2017-11-27T20:26:31+08:00
toc: true
author: "zvector"
author_homepage: "https://bivectorfoil.github.io/"
tags: ["Nginx", "Ubuntu16.04", "memo"]
categories: ["技术备忘录"]
CreativeCommons: true
draft: false
---

# 如何在Ubuntu16.04上安装Nginx

## 简介

> **Nginx**（发音同engine x）是一个 [Web服务器](https://zh.wikipedia.org/wiki/%E7%B6%B2%E9%A0%81%E4%BC%BA%E6%9C%8D%E5%99%A8)，也可以用作[反向代理](https://zh.wikipedia.org/wiki/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86)，[负载平衡器](https://zh.wikipedia.org/wiki/%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1)和 [HTTP缓存](https://zh.wikipedia.org/w/index.php?title=HTTP%E7%BC%93%E5%AD%98&action=edit&redlink=1)。        ----wikipedia 
>
> 本文将大致介绍如何在Ubuntu 16.04服务器上安装Nginx。参考于博文：[How To Install Nginx on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04)，感谢博文[作者](https://www.digitalocean.com/community/users/jellingwood)的辛勤劳动。

## 预备条件

开始安装之前，确保以具有```sudo```权限的用户身份执行接下来的命令，如果没有的话，这篇博文：[initial server setup guide for Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)是一个不错的教程。

一切就绪以后，以非Root用户身份登录Ubuntu服务器。

### 第一步： 安装Nginx

Ubuntu 的官方源已经包含了Nginx，我们可以通过官方源安装它。

```Bash
$ sudo apt-get update
$ sudo apt-get install nginx
```

不出意外地话，Nginx已经安装完毕。

### 第二步：调整设置防火墙

> ufw是一个主机端的iptables类防火墙配置工具。

如果服务器上还没有安装ufw的话，可以通过apt安装它，ufw的一般使用方法可以在[这里](https://wiki.ubuntu.com.cn/UFW%E9%98%B2%E7%81%AB%E5%A2%99%E7%AE%80%E5%8D%95%E8%AE%BE%E7%BD%AE)查看。

> ```bash
> $ sudo apt-get install ufw
> ```

通过调整防火墙使得Nginx服务可以正常运行。通过命令

> ```bash
> $ sudo ufw app list
> ```

可以列出ufw知道如何配置的应用程序，如下：

> ```shell
> # Output
> Available applications:
>   Nginx Full
>   Nginx HTTP
>   Nginx HTTPS
>   OpenSSH
> ```

和Nginx有关的项目解释如下：

- Nginx Full：负责打开80端口（常用的，未加密的网络流量）和443端口（TLS/SSL 加密流量）
- Nginx HTTP： 只打开80端口（常用的，未加密的网络流量）
- Nginx HTTPS ：只打开443端口（TLS/SSL 加密流量）

安全起见，应该选择配置限制条件最严格的设置，例如：

> ```bash
> $ sudo ufw enable # 开启ufw
> $ sudo ufw default deny # 阻止一切外部访问
> $ sudo ufw allow server_your_want # 依需要开启服务
> ```

目前网站还未配置SSL连接，所以我们只打开80端口。

> ```bash
> $ sudo ufw allow 'Nginx HTTP'
> $ sudo ufw status # 查看开启情况
> # Output
> Status: active
>
> To                         Action      From
> --                         ------      ----
> OpenSSH                    ALLOW       Anywhere                  
> Nginx HTTP                 ALLOW       Anywhere                  
> OpenSSH (v6)               ALLOW       Anywhere (v6)             
> Nginx HTTP (v6)            ALLOW       Anywhere (v6)
> ```

### 第三步：检查Web服务

经过上面的安装和设置以后，Ubuntu 16.04 应该已经启动了Nginx服务，使用```systemd```命令查看启动情况。

> ```bash
> $ systemctl status nginx
> # Output
> ● nginx.service - A high performance web server and a reverse proxy server
>    Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enab
>    Active: active (running) since Sun 2016-10-12 11:07:55 UTC; 1 day 2h ago
>   Process: 1120 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=e
>   Process: 1083 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on
>  Main PID: 1159 (nginx)
>     Tasks: 2
>    Memory: 10.7M
>       CPU: 154ms
>    CGroup: /system.slice/nginx.service
>            ├─1159 nginx: master process /usr/sbin/nginx -g daemon on; master_proce
>            └─1161 nginx: worker process 
> ```

通过标识Active：active(running)可知Nginx服务已经正常运行了。虽然Nginx服务已经启动 ，但是**真正**的通过域名或者IP地址访问服务器能更直观地感受到Nginx的运行状况。关于域名申请，有空的话我会再写一篇博客。

如果已经有了**能正确解析到服务器**上的域名或者知道服务器的IP地址，在**本地**浏览器地址栏输入：

> http://server_domain_or_IP

访问服务器，不出意外地话可以看到Nginx经典的默认欢迎界面了。

### 第四步：管理Nginx

常用的几个管理Nginx命令。

> ```bash
> $ sudo systemctl stop nginx # 停止Web服务
> $ sudo systemctl start nginx # 开启Web服务
> $ sudo systemctl restart nginx # 重启Web服务
> $ sudo systemctl reload nginx # 修改Nginx配置文件后可以保持连接地重新载入Nginx
> $ sudo systemctl disable nginx # 关闭Nginx的默认自启动
> $ sudo systemctl enable nginx # 开启Nginx的默认自启动
> ```

### 第五步：熟悉几个重要的Nginx文件和目录

#### 网站内容

- ```/var/www/html```：默认的网站内容存放目录，先前的Nginx欢迎界面就存放在这。可以通过修改Nginx配置文件更改。

#### 服务器配置

- ```/etc/nginx```：Nginx配置文件，修改它配置Nginx全局设置。
- ```/etc/nginx/sites-available/```：虚拟主机的目录，可在此创建多个虚拟主机。
- ```/etc/nginx/sites-enabled/```：Nginx默认站点目录，nginc.conf默认包含的文件夹。在此文件夹建立的站点，需要建立软连接到目录/etc/nginx/sites-enabled/里。

> 注：此段内容部分来自[博文](http://www.jianshu.com/p/fd25a9c008a0)

#### 服务日志

- ```/var/log/nginx/access.log```：如无其他设置，此文件记录了每一次的网页服务请求。
- ```/var/log/nginx/error.log```：Nginx错误请求记录文件。

That's all！更多的Nginx文档请参阅[官方文档](https://nginx.org/en/docs/)。

