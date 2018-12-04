---
title: "极简Hugo搭建个人博客笔记"
date: 2017-11-28T13:50:09+08:00
toc: true
author: "zvector"
author_homepage: "https://bivectorfoil.github.io/"
tags: ["memo", "Hugo", "Blog"]
categories: ["技术备忘录"]
CreativeCommons: true
draft: false
---

## 前言：

既然本文是极简笔记，那将不会把来龙去脉说个清楚，只记录下最关键和常用的步骤，详细文档请参阅[官方文档](https://gohugo.io/getting-started/)和[Hugo中文文档](http://www.gohugo.org/)。

> 注：另可参阅博主：[CoderZh](http://blog.coderzh.com/)的博文[使用hugo搭建个人博客站点](https://blog.coderzh.com/2015/08/29/hugo/)。原文地址：https://blog.coderzh.com/2015/08/29/hugo/

首先还是要先简单介绍下Hugo是什么，我们看看官方文档的介绍。

> Hugo is a fast and modern static site generator written in Go, and designed to make website creation fun again.

简而言之，Hugo是一个用Go语言写的静态网站生成器，对于像生成静态博客网站这样的任务（也是最主要的任务），Hugo十分高效和快速，而且优雅（在我看来）。不像其他静态网站生成器在使用之前需要安装一大堆的包，配置各种文件，Hugo只需要一个二进制文件（hugo.exe，在Windows下的话），和一个网站的主题（后文会提到它的作用）。这是我十分欣赏的一点，就像官方文档介绍的一样，`make website creation fun again`，对于博客写作，更应该关注于写作本身，而不是将心力耗费在各种配置，调整文件上。（想起了水平不行时强行用Django自己学习写博客系统，汗`）话不多说，开始搭建。

## 第一步：安装

1. 二进制安装

   可以在[Hugo Release](https://github.com/gohugoio/hugo/releases)下载对应操作系统版本的Hugo二进制文件。对于Ubuntu（14.04+）用户

   ```bash
   $ sudo apt-get install hugo
   $ hugo version # 查看版本 output v0.25.1
   ```


2. 源码安装（参阅[官方文档](https://gohugo.io/getting-started/installing/))或[中文文档](http://www.gohugo.org/))，在此不做介绍。

## 第二步：生成站点

以生成博客站点example-hugo-blog为例，生成到路径`/path/to/example-hugo-blog/`：

 ```bash
 $ hugo new site /path/to/example-hugo-blog
 $ cd example-hugo-blog # 这个目录就是博客站点的目录
 ```

目录结构：

```bash
.
├── archetypes
├── content
├── data
├── layouts
├── static
└── themes

6 directories
```

各个目录主要作用参照[官方文档](https://gohugo.io/getting-started/directory-structure/)，目前我们只需要关注两个目录，content（网站内容）和themes（网站主题）。

## 第三步：创建一些文章

在content目录下新建一个about.md页面（第一个页面总是关于自己的介绍嘛：），BTW，Hugo的默认写作格式是markdown，关于markdown的介绍，有空我会再写一篇博文。

```bash
# ~/example-hugo-blog/
$ hugo new about.md # 默认新建在content目录下 或者
$ hugo new path_you_want/about.md # 会新建content/path_you_want/about.md
```

新生成的页面默认内容与archetypes/default.md文件相同：

```markdown
---
title: "About"
date: 2017-11-28T15:14:14+08:00
draft: true
---
```

“---”扩起来的内容是md文件的[元数据](https://zh.wikipedia.org/wiki/%E5%85%83%E6%95%B0%E6%8D%AE)，由Hugo自动生成。定义了关于这个页面要显示那些信息，如何渲染成html文件等等。以“---”分隔开是[YAML](http://www.yaml.org/)格式，其他格式如以“+++”分隔的TOML格式详细内容见[官方文档](https://gohugo.io/getting-started/configuration/)。接下来我们新建第一篇博文，为了统一管理，均放置在content/post/目录下。

```bash
$ hugo new post/first.md
```

使用任何你喜欢的编辑器编辑first.md文件，输入一些内容，保存。

## 安装皮肤主题

Hugo默认是不带皮肤的，需要自行下载使用，否则使用hugo server查看网站情况时会是一片白茫茫大雪真干净。在官方的[皮肤列表](http://www.gohugo.org/theme/)选择一个下载使用。或者你也可以使用自己编写的主题，文档参阅[这里](https://gohugo.io/themes/creating/)。

```bash
$ cd themes
$ git clone https://github.com/digitalcraftsman/hugo-cactus-theme.git
```

## 运行站点

```bash
# ~/example-hugo-blog
$ hugo server --theme=hugo-cactus-theme --buildDrafts
```

在浏览器地址栏打开```localhost:1313```，That's all ！Hugo搭建个人博客先告一段落，关于Hugo的部署有空我会再写一篇博文：2018.5.28 更新，请参看[这里](https://bivectorfoil.github.io/post/deploy-personal-blogs-on-vps/)）Now,enjoy writing~
