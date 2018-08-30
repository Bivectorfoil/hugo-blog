---
title: "一个批处理缩进的VIM技巧"
date: 2018-03-05T11:07:06+08:00
author: "zvector"
author_homepage: "https://zvector.tk/"
tags: ["vim", "script"]
categories: ["技术闲谈"]
CreativeCommons: true
draft: false
---
### 灌水预警！
今天处理文本时遇到一个很常见的问题，我在~/src/projects/目录下有一大堆C文件，需要对这些文件执行一个相同的命令，如何在vim中批量处理呢？一番搜索，在stackoverflow上找到了一个漂亮的解决方案，特此记录下，地址[在这里](https://stackoverflow.com/questions/3218528/indenting-in-vim-with-all-the-files-in-folder)。具体如下：

`:args ~/src/myproject/**/*.ttl | argdo execute "normal gg=G" | update`

- `args` 设置要操作的文件列表，使用**匹配当前目录及子目录。
- 使用|设置管道，传递命令。
- `argdo`依次对满足`args`的文件执行接下来的命令。
- `execute`执行命令。
- `normal`执行normal模式下的命令。
- `update` 确保在文件被修改后保存。

原答案提供了一个通用的批量解决方法，用`:args .....| argdo .....|update`模式实际上可以解决很多常见的批处理问题，非常棒。
