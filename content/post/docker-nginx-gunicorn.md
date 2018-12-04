---
title: "Docker Nginx Gunicorn"
date: 2018-11-15T21:15:33+08:00
toc: true
author: "zvector"
author_homepage: "https://bivectorfoil.github.io/"
tags: ["docker", "nginx", "gunicorn"]
categories: ["技术备忘录"]
CreativeCommons: true
draft: false
---

## 记一次 Dockerize Flask Web App 的踩坑之旅

当我们的 Flask Web App 开发完毕，要部署上线时，有多种方案可供我们选择，传统的有如远程服务器部署，便利点的有云部署（PaaS，Platform as a Service）。云部署十分方便，不需要过多操心具体细节，故不在本文的讨论范围之内。如果需要自己做更多的定制的话，远程服务器部署可能更适合，本文诞生背景为利用虚拟主机（VPS）部署 Flask Web App，利用 Docker 容器技术实现一次构建，到处部署的效果。

### 背景知识

继续阅读前假定读者已经对以下概念有所了解：

1. 什么是 Docker 及其主要作用？- [文档](https://docs.docker.com/)
2. 如何安装 Docker 和常用的命令。- [文档](https://docs.docker.com/install/linux/docker-ce/debian/#install-docker-ce)
3. Dockerfile 文件编写的基本规则。 - [文档](https://docs.docker.com/engine/reference/builder/#usage)
4. Docker-compose 工具是什么及常用命令。- [文档](https://docs.docker.com/compose/)
5. Flask Web App 的主要概念。- [文档](http://flask.pocoo.org/)

### 开始踩坑

选择 Docker 之前其实已经配合 Shell 脚本部署成功了，虽然成功达到了`一键安装依赖并部署`的效果，但是体验确实一般 ~~水平不行写一行调试半天~~。而且 Shell 脚本与系统耦合太强，要保证高通用性必须针对不同系统做适配，这样的累活一定有解决的方案。于是，Docker 就有了出场机会了。

#### 部署前的准备

对于一个典型的 Flask Web 应用，要部署在线上，光凭借 Flask 自带的调试服务器肯定是不行的，在这里，我们选用 [Gunicorn](http://docs.gunicorn.org/en/latest/index.html) 做服务器，并配合 [Nginx](http://nginx.org/en/docs/) 做反向代理服务器。系统架构为：Flask 在后台处理请求，Gunicorn 做 WSGI 服务器，Nginx 反向代理静态文件，处理缓存，提供一层防护。

#### 创建镜像

一个典型的 Python 镜像 Dockerfile 如下：

```bash
FROM python:2.7.15-alpine3.8

RUN mkdir -p /usr/src/app  && mkdir -p /var/log/gunicorn

COPY ./requirements.txt /usr/src/app/requirements.txt

WORKDIR /usr/src/app

RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt
```

这里为了生成的镜像大小考虑，使用精简版的 Python 版本，并定义容器内部目录 `/usr/src/app/` 为项目目录，复制必要的项目依赖文件到项目目录中，并安装依赖包。在这里，我并没有将项目代码也一起拷贝到容器中，而是在接下来的 `docker-compose.yml` 文件中，使用 `Volumes` 挂载宿主机的项目代码到容器中。因为使用到了 Gunicorn 和 Nginx 配合工作，故编排 `docker-compose.yml` 文件，利用 `docker-compose` 工具，来构建镜像，运行容器。定义如下两个 Docker 服务：

```bash
version: '3'
services:
  flaskweb:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - .:/usr/src/app
    networks:
      - nginx_network
    command: '/usr/local/bin/gunicorn -w 2 -b :8000 wsgi:app'

  nginx:
    # restart: always
    image: nginx:stable-alpine
    volumes:
      - ./nginx/conf.d/:/etc/nginx/conf.d
      - ./gogo/static:/opt/services/flaskapp/static
    depends_on:
      - flaskweb
    ports:
      - "80:80"
    networks:
      - nginx_network

networks:
  nginx_network:
    driver: bridge
```

简要说明下文件含义，开头定义的 `version: '3'` 表示使用 `docker-compose` 的语法版本，当初刚接触 `docker-compose` 语法规则时还以为是所定义的服务的数量，然而并不是。对于 Flask 服务，需要注意的点不多，主要在于使用 `volumes` 挂载宿主机文件到容器中，定义了容器构建后要运行的命令。为了与 `Nginx` 互相通信，自定义了 `network` 。踩坑较多的地方在定义 `Nginx` 的服务时，除了配置文件从宿主机挂载到 `Nginx` 容器时的**位置**（否则自定义的配置文件 Nginx 加载不到）要注意的以外，`Nginx`所代理的`Flask` 静态文件设置也要特别小心。在接下来，Nginx 配置文件的设置中，代理静态文件的设置要与`docker-compose.yml`中定义的一致。一份自定义的 Nginx 配置文件如下：

```bash
upstream web {
    server flaskweb:8000;
}

server {
    listen 80;

    server_name localhost;
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    
    location / {
        proxy_pass http://web; # gunicorn run port
        proxy_redirect off;
        
        proxy_set_header Host               $host;
        proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
    }

    location /static {
        alias /opt/services/flaskapp/static/;
        expires 30d;
    }
}
```

注意所定义的上游服务器名字，需要与 `dockerfile.yml` 文件中定义的一致，使用自定义的 `network` 通信。代理静态文件的位置也应与 `dockerfile.yml` 文件中定义的一致。`proxy_pass` 代理入口也和非 `Docker` 部署的配置有所差别。

#### 开始部署

准备好 `Docker` 和 `docker-compose` 后，项目部署就十分简单了。进入项目根目录下：

```bash
$ docker-compose up -d  # run the Docker service in the background
$ docker-compose ps  # check for status
```

### 小结

使用 `Docker` 部署服务还是十分便捷的，当然也要付出一定的学习成本才好掌握。
