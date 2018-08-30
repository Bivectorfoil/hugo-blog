---
title: "如何购买VPS及购买后的安全措施"
date: 2017-11-24T22:46:36+08:00
toc: true
author: "zvector"
author_homepage: "https://zvector.tk/"
tags: ["Tech","memo"]
categories: ["技术备忘录"]
CreativeCommons: true
draft: false
---

# 如何购买VPS及购买后的安全措施




## 先备知识--什么是VPS

> **虚拟专用服务器**（英语：Virtual private server，缩写为 VPS），是将一台服务器分区成多个虚拟专享服务器的服务。实现VPS的技术分为容器技术和虚拟化技术 。                                          --wikipedia

通俗地，对个人来说，VPS就是一台24小时不断电的计算机，每台VPS可分配得独立公网IP（这很重要），在VPS上安装什么样的操作系统也取决于用户，知乎上有一个关于[VPS有哪些有趣的用途的问题](https://www.zhihu.com/question/24284566)也很有趣。

## 购买平台--以Vultr为例

那么，如何购买VPS呢？国人经常谈论的几家VPS商无外乎[Vultr](http://www.vultr.com/)，[DigitalOcean](https://www.digitalocean.com/)，[搬瓦工](https://bwh1.net/)，等等，在此不再赘述。我以Vultr为例，因为我购买的第一台VPS即是Vultr的，本博客也是搭建在Vultr上的。（关于在VPS上搭建个人博客，也许以后有空会写一篇教程。2018.5.28 更新，搭建个人博客可以查看[教程一](https://zvector.tk/post/minimal-hugo-build-personal-blog-notes/)，[教程二](https://zvector.tk/post/deploy-personal-blogs-on-vps/)）

首先，需要创建一个账户，使用电子邮箱注册，注册成功后Vultr会给邮箱发送一封确认邮件，包含确认链接。Vultr现在支持使用支付宝支付，创建账户以后，需要先充值一定的金额才能正常购买VPS，充值以后，有些教程会有优惠码赠送，这是Vultr提供给推荐者的优惠，推荐者和新注册用户都可得到优惠。但是，很遗憾，本教程没有：）账户在手，金额已充值，万事大全，那么重点来了，该选择那种操作系统呢？我当时选择的是Ubuntu16.04，无他，熟悉矣。读者可以自由选择，或者自行上传ISO文件。Vultr的几种套餐由读者量力选择，避免浪费。值得注意的是，购买好VPS以后，启动以后，等待几分钟，应该就能正常工作了，这时要注意复制记好VPS的IP地址，和ssh密码，后面通过ssh登录VPS需要用到。

## VPS购买后需要做的几件安全设置，ssh安全设置

### ssh安全设置

新购买的VPS极易遭到暴力破解的攻击，例如Root密码暴力破解。并且ssh默认使用22端口，这也是一个极易被攻击的入口。首先通过ssh以Root用户身份登录刚购买的VPS。Linux或者MacOS用户可以通过终端使用ssh工具登录，Windows系统下也有类似的工具，可自行搜索。
```bash
 $ ssh root@your_ip_addr # 输入VPS商提供的密码
```

登录后我们首先新建一个普通用户(第一次的登录是Root用户身份登录的)，以用户名ming为例

 ```bash
 $ adduser ming
 $ passwd ming # 这一步为普通用户ming配置密码
 $ passwd # 更改Root用户密码
 ```

给用户添加 sudo 命令的使用权限：

```bash
$ usermod -aG sudo ming
```

试试看sudo权限成功与否：

```bash
$ su ming
$ sudo whoami # root
```

此时我们可以退出ssh，以普通用户身份登录VPS。

 ```bash
 $ ssh ming@your_ip_addr
 ```

修改ssh默认的登录端口

 ```bash
 $ sudo vi /etc/ssh/sshd_config
 ```

找到Port 22 一行，修改为： Port number(你想要的数字，大于1024)。

禁止以root用户身份登录,仍是在/etc/ssh/sshd_config文件里，找到下面这一行：

 ```bash
 PermitRootLogin yes # yes更改为no
 ```

重启 ssh 服务：
```bash
sudo systemctl restart ssh
```

退出ssh，重新以新端口登录试试。

 ```bash
 $ ssh -p port ming@your_ip_addr # port为之前设置的登录端口
 # 若尝试以Root用户登录，会获得一个Permission denied, please try again的警告。
 $ ssh -p port root@your_ip_addr
 ```

### ssh公钥登录

更安全的ssh登录方法是使用密钥登录，可以参考阮一峰的博文[SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)，并了解什么是RSA加密，[数字签名是什么？](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)。

为了使用ssh公钥登录，首先修改配置文件/etc/ssh/sshd_config，确保以下几项没被注释掉：

 ```bash
 RSAAuthentication yes
 PubkeyAuthentication yes
 AuthorizedKeysFile .ssh/authorized_keys # 指定公钥数据库文件
 ```

本地生成密钥对

 ```bash
 $ ssh-keygen -t rsa # 选择REA加密方式。
 ```

之后一路回车，若不放心私钥安全，可在询问passphrase时设置一个私钥的口令。在本地$HOME/.ssh/目录下，会发现生成了 id_rsa(私钥)，id_rsa.pub(公钥)，两个文件。接下来就是如何安全地把公钥发送到VPS上，这样本地就可以利用私钥登录远程主机。这一步当初颇费了一番周折。在将公钥发送到远程主机时出现了各Permitssion denied错误，查了各种资料，大概的解决过程如下。

首先，需要确保VPS在$HOME下存在.ssh/目录，这是公钥的存储位置。在本例中，目录为：

 ```bash
 /home/ming/.ssh # 在创建用户时会分配了用户目录
 $ sudo mkdir .ssh # 若无，则创建一个
 ```

并且要确保这个目录的所有权为当前用户及用户组，这一步很重要，如果文件夹的所有权为root，那么使用ssh-copy-id发送密钥时会被拒绝。

```bash
$ sudo chown user.group .ssh -R
$ ls -l .ssh # 查看.ssh目录所有者
```

使用如下命令，将公钥发送到远程主机

```bash
$ ssh-copy-id -i ~/.ssh/id_dsa.pub "ming@your_ip_addr -p port"
```

也可以选择命令

```bash
$ ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub
```

详细解释参见阮一峰的[博文](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)。

为了使得ssh登录更加方便，我们可以在**本地**设置配置文件，避免每次都需要输入端口号，ip地址，密码等。在目录$HOME/.ssh/下，创建文件config。常用的ssh配置项有：

+ Host 别名
+ HostName 主机名
+ Port 端口
+ User 用户名
+ IdentityFile 密钥文件的路径
+ IdentitiesOnly 只接受SSH key 登录
+ PreferredAuthentications 强制使用Public Key验证

写入以下项：

```config
Host name # 为ssh登录设置的主机别名
HostName your_IP_addr # VPSIp地址
Port port # ssh端口
User 用户名
IdentityFile ~/.ssh/id_rsa # 私钥位置
PreferredAuthentications publickey
```

保存后退出。通过以上设置，在远程主机上重启ssh服务。

```bash
$ sudo systemctl restart sshd.service
```

退出远程主机，尝试使用配置文件登录

```bash
$ ssh user@server-alias # 通过先前配置的主机别名和用户名使用密钥登录
```

通过密钥登录成功以后，可以修改ssh配置，禁用密码登录，进一步提高安全性。

```bash
$ sudo vi /etc/ssh/sshd_config # 找到 #PasswordAuthentication yes 修改为
PasswordAuthentication no
```

同样的，重启ssh服务

```bash
$ sudo systemctl restart sshd.service
```

