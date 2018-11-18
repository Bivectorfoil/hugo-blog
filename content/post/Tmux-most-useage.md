---
title: "Tmux的快速操作入门"
date: 2018-04-29T15:25:02+08:00
toc: true
author: "zvector"
author_homepage: "https://bivectorfoil.github.io/"
tags: ["tmux"]
categories: ["技术备忘录"]
CreativeCommons: true
draft: false
---

# 什么是Tmux

[Tmux](https://github.com/tmux/tmux) 是一个多终端复用器，简单来说，我们可以使用Tmux来管理多个终端（Shell）。Tmux可以实现在一个终端窗口中管理多个终端，而不必打开多个终端窗口。并且可以方便的分割窗口，随时最大化当前窗口或恢复原大小。Tmux带来的便利性远不止于此。它甚至还可以管理多个Sessions，更进一步的将我们从打开的繁多个终端迷雾中解放出来。

工欲善其事，必先利其器。盲目地追求酷炫的工具，沉迷于炫技一般的工具操作中是本末倒置的做法，十分不可取。工具应该为我们所用，而不是令我们迷失在工具繁杂的操作技巧中不能自拔。那么怎么知道Tmux适不适合我使用呢？如果你差不多是一个“活在命令行”中的人，每天都要打开多个终端窗口，对成天辗转于多个数不清的终端窗口中感到十分厌倦。追求效率的最大化，安全性。那么，你可以接着往下看了，在了解了Tmux的基本操作后，Tmux不会让你失望。Tmux的安装方式和其他软件大同小异，在此不再赘述。

# Tmux的基础操作方式

在介绍Tmux的基础操作前，先让我们大体上了解一下Tmux的工作原理。

## C/S模型

根据[维基百科](https://zh.wikipedia.org/wiki/Tmux)，tmux 采用 [client/server](https://zh.wikipedia.org/wiki/Client/server) 模型，主要由以下模块组成：

| 模块      | 简介                          |
| ------- | --------------------------- |
| server  | 服务。tmux 运行的基础服务，以下模块均依赖此服务。 |
| session | 会话。一个服务可以包含多个会话。            |
| window  | 窗口。一个会话可以包含多个窗口。            |
| panel   | 面板。一个窗口可以包含多个面板。            |

当在命令行输入 ```tmux```时，Tmux就开启了一个server，session，window，panel等模块均运行于此。session，window，panel 的包含关系为：session < window < panel，后者为前者的子集。可以看出Tmux可以包含数量众多的**窗格**（panel），而每一个Panel都可以最大化为当前**窗口**（window），方便使用，这是一个层级包含的关系。Tmux操作带来的便利性就在于此。我们只需在一个终端窗口中开启tmux，便可管理多个**session**，**window**，**panel**。并且这些是层次分明，有条不紊的，从而不会使得我们迷失工作要点。

工具中百分之二十的基础操作差不多就可以应对工作中百分之八十的需求。熟练掌握Tmux的一些基本操作，便可享受其所带来的极大的便利性。下面让我们来看看Tmux的几个基本操作。

## 基础操作

### 窗格操作

终端中输入

```bash
tmux
```

我们就进入了Tmux的工作环境，Tmux Server已经在后台运行了，目前我们处在一个serssion中，它包含了一个window，在没有进行下一步操作后，当前window只有一个窗格（Panel）让我们来分割窗口。

```bash
ctrl-b %
```

注意，ctrl-b（下文均使用C-b表示） 是Tmux基础操作中一个重要的前缀键，在基础操作命令中均需要先按下C-b键（松开后）输入其他命令。C-b %表示将当前窗口竖分为左右两个窗格。

而命令：

```bash
C-b " # C-b加上 "（双引号）
```

表示将当前窗口水平分为上下两个窗格，使用C-b  + %，C-b + "，可将窗口任意分为多个窗格。

要查看窗格的编号，我们可以使用命令：

```bash
C-b q
```

会显示当前窗口下所打开的窗格的顺序。想要暂时最大化当前窗格应该怎么做呢？命令：

```bash
C-b z
```

可以最大化光标处的窗格，方便编辑文件，代码等。再次使用此命令可以退出最大化。需要将窗格升级为窗口？使用命令：

```bash
C-b !
```

可将窗格升级为新窗口，也使得我们进入到接下来的操作了。

### 窗口操作

```bash
C-b c
```

新建窗口，此时你会发现在下方的状态栏里出现了多个窗口的名称。使用命令：

```bash
C-b w
```

可以查看当前session（还记得三者间的层级关系吗？）中有几个窗口，上下键选择后回车可进入选中的窗口。

在新建的窗口中同样的可以执行C-b + %，或C-b + "，新建窗格。为了不导致窗口之间的混乱，我们可以使用命令：

```bash
C-b , # ,为英文逗号
```

重命名当前窗口，给窗口一个主要任务的命名。开启了多个窗口，该怎么在窗口之间切换？命令：

```bash
C-b n # 切换到下一个窗口
C-b p # 切换到上一个窗口
C-b [0-9] # 在窗口[0-9]之间进行切换
```

在下方的状态栏中，当前所在窗口名称后会有“*”号标识，方便识别。

### Session操作

Tmux最酷的一点是，我们可以临时挂起当前会话，转而去做其他的事情，而挂起的会话不会受到影响。使用命令：

```bash
C-b d
```

挂起当前会话，我们会退回到了一开始使用命令``tmux``建立Tmux Server的位置。Tmux的这一特性可以保证在使用ssh连接远程服务器时，若应未知的网络中断导致session中止，当我们再次进入ssh时可以恢复中断的session。使用命令：

```bash
tmux attach
```

可以恢复先前挂起的Session。要注意，Tmux不能保证关机后Session的恢复，要实现这一点，可以配合适当的Tmux插件和配置来完成，此处不谈。如同给window命名一样，我们也可以给Session命令，使用命令：

```bash
C-b $
```

可以在下方的状态栏中按照提示重命名当前Session。打开了多个Session，使用完毕后怎么关闭呢？我们先使用命令：

```bash
C-b s # or 
tmux ls
```

查看所有Session名称，同时也可以看到每一个Session中所含的窗口数量，或者切换到不同的Session中去。为避免不必要的麻烦，我们不在要关闭的Session中执行以下命令：

```bash
tmux kill-session -t [session-name]
```

[session-name]即为要关闭的Session名称。若在要关闭的Session中执行，Tmux会退出Session，来到终端命令行。此时我们可以使用```tmux a```回到未退出的Session中。

以上就是Tmux的一些基本操作，对于初学者来说，上述操作可以解决大部分使用Tmux的需求了。若有其他需求，可自行搜索。

# Tmux的配置

很多时候，Tmux会和Vim配合使用，如果Vim已经配置了自己的配色方案，在Tmux中使用Vim会发现Vim的配色方案失效了。这时候我们需要配置Tmux。新建（如果没有的话）Tmux的配置文件```~/.tmux.conf```，写入如下配置：

```bash
set -g default-terminal "screen-256color"
# navigate panes with hjkl, optional
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
# mouse support, optional
set -g mouse on

```

绑定hjkl键为Tmux中在窗格（Panel）中移动的方式，Vim-mode方法。注意，使用hjkl前同样需要先按前缀键C-b。最后一行是设置Tmux支持鼠标操作，在窗格中上下浏览更方便。否则需要使用命令

```bash
C-b Pgup/Pgdn # up or down scroll
```

来上下浏览。在vimrc中，写入：

```bash
set term=screen-256color
```

完成设置。这样，vim在Shell和Tmux中的行为应该是一致的了。
