---
title: "如何重置MySQL的root密码"
date: 2018-06-02T12:47:10+08:00
toc: true
author: "zvector"
author_homepage: "https://bivectorfoil.github.io/"
tags: ["mysql"]
categories: ["技术备忘录"]
CreativeCommons: true
draft: false
---

适用于mysql 5.7.8及以上版本。

### Step 1 - 停止数据库服务

```bash
$ sudo systemctl stop mysql
```

### Step 2 - 绕过授权检查重置密码

停止加载用户权限信息的授权表，以及停用网络连接，防止其他用户登录。

```bash
$ sudo mysqld_safe --skip-grant-tables --skip-networking &
```

之后切换到其他终端，若上一步有报错提示：

```bash
mysqld_safe Directory ‘/var/run/mysqld’ for UNIX socket file don’t exists
```

解决方案如下：

```bash
$ sudo mkdir /var/run/mysqld
$ sudo chown mysql:mysql /var/run/mysqld
```

### Step 3 - 用客户端连接到mysql

```bash
$ mysql -u root
```

若之前的步骤不出问题的话，接下来应该就能看到mysql 的提示符了

```bash
...
mysql >
```

### Step 4 - 重置Root密码

```bash
mysql > use mysql;
mysql > update user set authentication_string=password('new_password') where user='root' ;
mysql > flush privileges;
mysql > quit
```

注意每条命令后以";"结尾（quit除外），另外，在旧版的MySQL（mysql 5.7.6及以下）中，```authentication_string```字段可能为```password```，要注意区别。

### Step 5 - 停止MySQL服务，并重启

停止先前创建的```mysqld_safe```服务，重新打开```mysql```服务并检验密码更改是否成功。

```bash
$ sudo systemctl start mysql
```

```bash
$ mysql -u root -p
```

输入新设置的密码后登录。



#### 参考来源

1. [来源1](https://www.cyberciti.biz/tips/recover-mysql-root-password.html)
2. [来源2](https://blog.csdn.net/csdnones/article/details/53706762)
