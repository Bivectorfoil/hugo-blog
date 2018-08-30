---
title: "使用Git部署Hugo静态博客到VPS上"
date: 2017-12-02T11:07:44+08:00
toc: true
author: "zvector"
author_homepage: "https://zvector.tk/"
tags: ["VPS", "Hugo", "Let's Encrypt", "blog", "memo"]
categories: ["技术备忘录"]
CreativeCommons: true
draft: false
---

## 预先准备

1. 一台VPS，可见[如何购买VPS及购买后的安全措施](https://zvector.tk/post/how-to-buy-vps-and-security-setting/)
2. 一个DNS解析到VPS的域名，可见[如何拥有一个域名](https://zvector.tk/post/how-to-get-a-domain-name/)
3. 本地搭建好的Hugo博客，可见[极简Hugo搭建个人博客笔记](https://zvector.tk/post/minimal-hugo-build-personal-blog-notes/)
4. 使用Nginx作为网站服务器，可见[Install Nginx Ubuntu16.04](https://zvector.tk/post/install-nginx-ubuntu16.04/)

如果一切就绪，那让我们现在开始吧~

## Git部署

```bash
$ hugo # 在博客目录下生成了public文件夹，这就是博客网站的目录
```

ssh登录到VPS，我们需要修改Nginx配置文件，在目录```/etc/nginx/nginx.conf```下，以sudo权限修改，找到http段，修改以下项：

```
server {
        listen 80;
        server_name your_blog_domain_name;
        root /usr/share/nginx/html;
        location / {
                index index.html;
        }
        error_page 404 /404.html;
```

将include /etc/nginx/sites-enabled/*;注释掉，否则网站的默认目录会是/var/www/html，而不是/usr/share/nginx/html了。

既然是使用Git完成博客的部署，VPS上首先应先安装Git，此处略过。依次执行：

```bash
$ cd ~ # 回到用户目录
$ mkdir blog && cd blog # 此处以目录名blog为例，可自行修改
$ git init --bare && cd hook && touch post-receive # 创建裸仓库，并创建Git hook文件
$ vim post-receive # 用你喜欢的编辑器打开post-receive文件并写入以下内容：
```

```git-hook
#!/bin/sh 
GIT_REPO=/home/your_user/blog
TMP_GIT_CLONE=/tmp/blog
NGINX_HTML=/usr/share/nginx/html
rm -rf ${TMP_GIT_CLONE}
git clone $GIT_REPO $TMP_GIT_CLONE
rm -rf ${NGINX_HTML}/*
cp -rf ${TMP_GIT_CLONE}/* ${NGINX_HTML}
```

关于git hook的工作原理，可以查看[自定义 Git - Git 钩子](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)。

保存文件后添加可执行权限```sudo chmod +x post-receive```，每当blog仓库有push更新，Git钩子可以自动将更新拷贝到先前设置的Nginx网站目录中去，使网站内容更新。我们还需要修改网站文件夹/usr/nginx/html的所有权为你所拥有：```sudo chown your_user:your_user /usr/share/nginx/html```。

## 本地仓库建立

在博客的public目录下：

```bash
$ cd public
$ git init
$ git add --all
$ git commit -m "Initial blog repo"
$ git remote add origin your_user@IPAddress:/home/your_user/blog
$ git remote set-url origin ssh://your_user@VPS_IP:Port/home/your_user/blog # 若VPS的ssh默认端口不是22，需另设置ssh端口。
$ git push origin master
```

以上便完成了Hugo在VPS上的部署，下面我们更进一步，为自己的网站添加HTTPS支持。

## 添加网站HTTPS化,Let's Encrypt!

### 前言

> **Let's Encrypt**是一个于2015年三季度推出的[数字证书认证机构](https://zh.wikipedia.org/wiki/%E6%95%B0%E5%AD%97%E8%AF%81%E4%B9%A6%E8%AE%A4%E8%AF%81%E6%9C%BA%E6%9E%84)，旨在以自动化流程消除手动创建和安装证书的复杂流程，并推广使[万维网](https://zh.wikipedia.org/wiki/%E8%90%AC%E7%B6%AD%E7%B6%B2)服务器的加密连接无所不在，为安全网站提供免费的[SSL](https://zh.wikipedia.org/wiki/SSL)/[TLS](https://zh.wikipedia.org/wiki/TLS)证书。--wikipedia

### 第一步，安装Certbot

Certbot是Let's Encrypt推出的自动化配置工具，我们通过添加官方源来下载它。

```bash
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx
```

先前我们已经设置了Nginx所要配置的网站域名，可以通过以下命令查看有没有配置错误。

```bash
$ sudo nginx -t
```

### 第二步，调整防火墙设置

```bash
$ sudo ufw status # 查看ufw状态
$ sudo ufw allow 'Nginx Full' # 开启Nginx HTTP和HTTPS支持
$ sudo ufw delete allow 'Nginx HTTP' # 关闭先前设置的HTTP，避免和Nginx HTTP重复
$ sudo ufw status
```

### 第三步，获取SSL证书

```bash
$ sudo certbot --nginx -d your_domain_name
```

Certbot使用nginx插件，选项-d指定需要使用SSL证书的域名。一路回车，输入联系邮件地址，Certbot工具会自动完成证书的注册和部署。之后再重新打开网站，应该就会看到网站首部的绿色安全标记了。

### 确认Certbot自动更新

Let's Encrypt只有三个月的有效期，这时为了督促用户及时更新网站的安全设置而考虑的。Certbot工具在/etc/cron.d/目录下设置了自动任务，可以以一天两次的频率确保证书不过期，我们来测试一下自动更新。

```bash
$ sudo certbot renew --dry-run
```

一切顺利的话将会看到模拟自动更新成功。如果当后续的自动更新失败的话，Let's Encrypt会向我们先前设置的邮箱地址发一封警告邮件，至此，网站的HTTPS化完成。

## 参考资料

1. [VPS+hugo+Nginx博客搭建全过程](https://paysonli.com/post/VPS+hugo+Nginx%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E5%85%A8%E8%BF%87%E7%A8%8B/)
2. [How To Set Up Let's Encrypt with Nginx Server Blocks on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-let-s-encrypt-with-nginx-server-blocks-on-ubuntu-16-04)



