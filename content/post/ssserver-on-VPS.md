---
title: "在VPS上搭建绿色上网工具"
date: 2017-12-02T15:25:11+08:00
toc: true
author: "zvector"
author_homepage: "https://bivectorfoil.github.io/"
tags: ["ssserver"]
categories: ["技术备忘录"]
CreativeCommons: true
draft: false
---



## 服务端搭建

以下以SS指代Shadowsocks。SS分为服务器端和客户端，安装时两个都会安装上，配置文件也相同，区分服务端和客户端的是运行程序的不同。

服务端搭建主要有以下几个步骤：

1. SSH登录VPS；
2. 下载安装pip，shadowsocks；
3. 编辑配置文件并配置开机自启动；
4. 优化SS。

### 第一步，SSH登录VPS

方法参考之前的博文[如何购买VPS及购买后的安全措施](https://bivectorfoil.github.io/post/how-to-buy-vps-and-security-setting/)，此处不再赘述。

### 第二步，下载安装

Debian/Ubuntu	用户运行：

```bash
$ sudo apt-get install python-pip
$ sudo pip install shadowsocks
$ ssserver --version # 查看版本号
```

### 第三步，编辑配置文件和开机自启动

#### 编辑服务端配置文件

   ```bash
   $ cd /etc/
   $ sudo mkdir shadowsocks && sudo touch shadowsocks.json # 创建SS配置文件
   $ sudo vim shadowsocks.json # 使用你喜欢的编辑器打开并写入以下项，具体解释可参考官方文档
   ```

   ```bash
   {
       "server": "my_server_ip", // 这里输入本机的 IP 地址
       "server_port": 8388, // 为了安全，可修改为大于 1024 的数字
       "local_address": "127.0.0.1",
       "local_port": 1080
       "password": "mypassword", // 设置一个密码
       "timeout": 300,
       "method": "aes-256-cfb",
       "fast_open": false
   }
   ```

   保存后建议修改文件权限为ROOT，提高安全性。配置完成后启动SS服务端

   ```bash
   $ sudo ssserver -c /etc/shadowsocks/shadowsocks.json # 前台运行
   $ sudo ssserver -c /etc/shadowsocks/shadowsocks.json -d start # 后台运行	
   $ sudo ssserver -c /etc/shadowsocks/shadowsocks.json -d stop # 后台停止
   ```

#### 配置开机自启文件

   ```bash
   $ cd /etc/
   $ sudo vim rc.local # 在此文件的exit 0这一行之前添加下面这句内容
   ssserver -c /etc/shadowsocks/shadowsocks.json -d start # 保存并退出
   ```

### 第四步，优化SS

#### 优化内核参数

```bash
$ sudo touch /etc/sysctl.d/local.conf # 创建优化文件，并写入以下内容
# max open files
fs.file-max = 51200
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

# for high-latency network
net.ipv4.tcp_congestion_control = hybla

# for low-latency network, use cubic instead
# net.ipv4.tcp_congestion_control = cubic
```

保存并退出，然后执行

```bash
$ sudo sysctl --system
$ sudo sysctl -p /etc/sysctl.d/local.conf # 对较老的系统
```
#### TCP优化

在服务端的客户端的配置文件将```fast_open```设置为```true```，然后，执行：
```Bash
$ su # 切换为ROOT用户
$ echo 3 > /proc/sys/net/ipv4/tcp_fastopen
$ ssserver -c /etc/shadowsocks/shadowsocks.json # 重启SS服务
```
至此，服务端搭建并优化完成。

## 客户端

SS在各大平台均有客户端软件，可在[这里查看](https://shadowsocks.org/en/download/clients.html)。对GUI类型，有：

### 1. 以Window平台，有[1](https://github.com/shadowsocks/shadowsocks-windows/releases)，[2](https://github.com/shadowsocks/shadowsocks-qt5/wiki/Installation)
### 2. Linux平台，有[1](https://github.com/shadowsocks/shadowsocks-qt5/releases)

客户端的配置文件与服务器端一样，不再赘述，可参考官方文档。



## 参考资料

1. [使用 Shadowsocks 自建翻墙服务器，实现全平台 100% 翻墙无障碍](https://www.loyalsoldier.me/fuck-the-gfw-with-my-own-shadowsocks-server/)
2. [官方文档](https://shadowsocks.org/en/config/quick-guide.html)
3. [官方文档2](https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks)

