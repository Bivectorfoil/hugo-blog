---
title: "浅谈Linux命令rm及回收站"
date: 2017-12-22T21:34:12+08:00
toc: false
author: "zvector"
author_homepage: "https://bivectorfoil.github.io/"
tags: ["shell", "linux"]
categories: ["技术闲谈"]
CreativeCommons: true
draft: false
---

## 灌水预警

很久没有写博客了，今天在整理硬盘空间时发现一个小常识，惊异于自己居然没想过这个问题，特此记录。

## 安全的rm命令

在谈到那个小常识前，我想先顺带记录一下很久前写的这个alias命令。alias是什么？字面意思，它是一个别名，用于给常用的命令起一个简单易记的别名，或用于替换某些“杀伤力强”的命令，没错，正是我要说到的rm命令。

使用Linux系统的用户应该都知道rm命令的威力，尤其是那个著名的rm -rf笑话，毋庸置疑，rm是一个效力很强的命令，如果使用不得当，很容易造成无法挽回的后果，误删文件。在经历一次小小的误删文件后，我决定要“改造”下这个命令了。在《Linux Shell脚本攻略》中，提供了一个alias命令，用于替换rm命令，使得我们使用rm命令时只是将文件移动到了自定义的Trash目录中，更安全的删除文件，这个alias命令如下：

```Bash
alias rm='cp $@ ~/backup && rm $@'
```

命令的大概意思是，首先复制要删除文件（文件名由$@传递）到~/backup 目录，然后删除它。但在我的实际使用中，这个alias命令无法运行。查了资料后，发现，在Bash Shell中，alias命令不接受参数传递。有关讨论可以看[这个问题](https://www.zhihu.com/question/23137414)，function才能。于是我修改了下，改为：

```Bash
alias rm='move(){ /bin/mv -v $@ ~/Trash ;};move $@'
```

定义了move函数，调用命令mv，替代rm命令，移动要删除文件到~/Trash目录下。那么，怎么真正地删除~/Trash目录下的文件呢？使用“\"转义即可，\rm即转义 alias命令，执行系统命令rm。但是要注意的是，在shell脚本中调用的rm命令仍然是系统级的rm。关于更加安全，健壮的删除命令，Github上有很多工具可以使用。

## 图形界面中的delete

rm是终端操作中的删除命令，那么在使用Linux系统下的图形界面时，我们使用键盘上del键删除的文件，到了哪里？先前我以为和rm命令一样，是直接删除的，今天突然发现不是，以我目前使用的Deepin15.5为例，del删除的文件均移动到了用户目录的~/.local/share/Trash/files/目录下，所有使用系统一段时间后，要注意及时清理。
