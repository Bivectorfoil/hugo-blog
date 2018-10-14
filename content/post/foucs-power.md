---
title: "专注的力量"
date: 2018-10-14T10:09:51+08:00
author: "zvector"
author_homepage: "https://bivectorfoil.github.io/"
tags: ["Web"]
categories: ["技术闲谈"]
CreativeCommons: true
draft: false
---

Pyhton 开发者中很少有人没听过 Kenneth Reitz 的名号，Requests ( HTTP for Human), Pipenv (Pip and Virtulenv) 等库均是由他主持开发的。其余闪闪发光的事迹不必赘述，今天我想扯些闲篇，谈谈大牛们的执行力，学习一下大牛们优秀的开发习惯。

## Responser

> 先插入一段看似无关紧要的库介绍，摘自[官网](http://python-responder.org/en/latest/)

简单来说 Responser 是一个用 Python 开发的 Web 框架， 你可能会说，Web 框架已经有 Django 和 Flask 等一些优秀的框架了，何必再来一个，不是重复造轮子了么，我们来看看[官方文档](http://python-responder.org/en/latest/#a-familiar-http-service-framework)的解释：

> The Python world certainly doesn’t need more web frameworks. But, it does need more creativity, so I thought I’d spread some [Hacktoberfest](https://hacktoberfest.digitalocean.com/) spirit around, bring some of my ideas to the table, and see what I could come up with.

一言以蔽之，Responser，为创意而生。一个简单的例子：

```python
import responder

api = responder.API()

@api.route("/{greeting}")
async def greet_world(req, resp, *, greeting):
    resp.text = f"{greeting}, world!"

if __name__ == '__main__':
    api.run()
```

`async` 装饰符可选，这样，就构建了一个简单的 ASGI 应用，带有预装的 Jinja2 模板引擎。更多例子及用法可在[官方文档](http://python-responder.org/en/latest/#a-familiar-http-service-framework)找到。

Responser 糅合了 Flask 和 Falcon 的基本思想。到目前为止，似乎没有什么值得惊讶的事情，又一个 Flask-like Web 框架而已。然而事实是，这一框架，从雏形到官方网站上线，到 v 0.0.3 发布，仅仅只用了不过短短几天的时间！（第一次提交为 2018 年 10 月 8 日）密集的提交还在继续，短短几天已经有了近三百次提交记录，绝大部分是 Kenneth Reitz 的提交，不得不让人感慨大牛疯狂的执行力和开源社区的力量。What a crazy ! 推特上有人问 Kenneth 怎么 ~~跑得这么快~~ 开发得这么快， 一个似乎昨天才刚刚提及的概念，今天就做出来了。大牛戏谑地[答道](https://twitter.com/kennethreitz/status/1050734477826318336)：

> focus ;)

这也是我开这篇 ~~扯淡~~ 闲聊博文的主要动机。给如此咸鱼的自己一个敲打，最真实的励志故事就是活生生地发生在身边的事。灌水结束，May the fouce be with you, 我也要去追随大牛的执行力了：）
