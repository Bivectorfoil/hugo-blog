---
title: "使用 Ruby 自制 Web 服务器"
date: 2020-12-27T17:26:29+08:00
toc: true
author: "zvector"
author_homepage: "https://bivectorfoil.github.io/"
tags: ["Ruby", "Web"]
categories: ["技术翻译"]
CreativeCommons: true
draft: false
---

> 原文链接：[Build Your Own Web Server With Ruby](https://www.rubyguides.com/2016/08/build-your-own-web-server/) ，作者：[Jesus Castello](https://www.rubyguides.com/author/admin/) 

少年，你是否曾用 Ruby 写过专属于你自己的 web 服务器？虽然我们已经有了如下[服务器应用](https://www.rubyguides.com/2019/08/puma-app-server/)：

- Puma

- Thin

- Unicorn

  

但是我还是觉得自己写一个简单的 web 服务器会是学习服务器运行原理的**绝佳机会**。今天你算是赶上了。
让我们一步一步来！

## 第一步：监听连接

首要的第一件事是监听 80 端口的 TCP 新连接。
我之前写过一篇 [Ruby中的网络编程](https://www.rubyguides.com/2015/04/ruby-network-programming/)，所以这里我就不多解释原理了，**直接上代码**：

```ruby
  require 'socket';
  server = TCPServer.new('localhost', 80)

  loop {
    client = server.aceept
    request = client.readpartial(2048)
    
    puts request
  }
```

运行上述代码，我们得到了一个在 80 端口上接受请求的
server。目前来说它做的工作还不多，不过也足够你看到进来的请求是什么样子的了。

> “在 Linux/Mac 系统中使用 80 端口可能需要 root
> 权限，作为替代，你也可以换成其他大于 1024 的端口，我比较钟意 8080”

使用浏览器或者是 `curl` 可以简单地构造一个请求。

如下是你这么做了之后所能看到的打印信息：

```html
  GET / HTTP/1.1
  Host: localhost
  User-Agent: curl/7.58.1
  Accept:*/*
```

这是一个 [HTTP 请求](https://www.rubyguides.com/2018/08/ruby-http-request/)。HTTP 是一个为了在 web 浏览器和 web
服务器之间通信的文本协议。正式的协议细节详见这里：[https//tools.ietf.org/html/rfc7230](https//tools.ietf.org/html/rfc7230)。

## 第二步：解析请求

现在我们需要将请求分割成服务器所能理解的小份形式。要实现这样的效果我们可以自己实现解析器或者复用已有的工具。为了加深对请求每一部分的理解，我们还是撸起袖子自己干吧。

**这副配图应该有帮助**

![](https://i2.wp.com/i.imgur.com/WEhYtyK.png?ssl=1)



请求头里包括浏览器缓存、虚拟地址和数据压缩等等，不过对于一个基础的服务器实现来说，我们可以忽略这些要素。

认识到如下事实对我们实现自己的解析器大有俾益：请求数据是被换行符号 `\r\n`
所分割的。为保持简单，我们不会做错误处理或数据校验。

那么我们就得到了如下代码：

```ruby
  def parse(request)
    method, path, versoin = request.lines[0].split

    {
      path: path,
      method: method,
      headers: parse_headers(request)
    }
  end

  def parse_headers(request)
    headers = {}

    request.lines[1..-1].each do |line|
      return headers if line == "\r\n"

      header, value = line.split
      header = normalize(header)

      headers[header] = value
    end

    def normalize(header)
      header.gsub(":", "").downcase.to_sym
    end
  end
```

以上将会返回经过解析的请求数据。既然我们已经得到了规范化后的请求数据，现在可以为客户端构造响应了。

## 第三步 准备好发送响应

在返回响应之前我们需要检查请求资源是否可用。换句话说，我们需要检查文件是否存在。

以下是为此准备的代码：

```ruby
  SERVER_ROOT = "/tmp/web-server/"

  def prepare_response(request)
    if request.fetch(:path) == "/"
      respond_with(SERVER_ROOT + "index.html")
    else
      respond_with(SERVER_ROOT + request.fetch(:path))
    end
  end

  def respond_with(path)
    if File.exists?(path)
      send_ok_response(File.binread(path))
    else
      send_file_not_found
    end
  end
```

**代码中包含两种情况：**

- 首先，如何请求路径是 `/` 的话，我们假设所希望请求的文件是 `index.html`。
- 其次，如果请求文件存在，我们便将文件内容连同一个 OK 响应发回。

不过如果文件不存在的话，我们就要返回经典的 `404 Not Found` 响应了。

## 常用的 HTTP 响应码

供参考。


| Code | 描述                  |
| ---- | --------------------- |
| 200  | OK                    |
| 301  | Moved permanently     |
| 302  | Found                 |
| 304  | Not Modified          |
| 400  | Bad Request           |
| 401  | Unauthorized          |
| 403  | Forbidden             |
| 404  | Not found             |
| 500  | Internal Server Error |
| 502  | Bad Gateway           |

## 响应类和方法

以下是我们上面用到的 "send" 方法：

```ruby
  def send_ok_response(data)
    Response.new(code: 200, data: data)
  end

  def send_file_not_found
    Response.new(code: 404)
  end
```

以下是 `Response` 类：

```ruby
  class Reponse
    attr_reader :code

    def initialize(code:, data: "")
      @response = 
      "HTTP / 1.1 #{code}\r\n" +
      "Content-Length: #{data.size}\r\n" +
      "\r\n" +
      "#{data}\r\n"

      @code = code
    end

    def send(client)
      client.write(@response)
    end
  end
```

响应由模板和字符串插值构成。到此阶段之后只要将所有这些代码组合进之前的接受连接的
`loop` 里，我们就得到了一个功能完备的服务器了。

```ruby
  loop {
    client = server.accept
    request = client.readpartial(2048)

    request = RequestParse.new.parse(request)
    response = ResponseParse.new.parse(request)

    puts "#{client.peeraddr[3]} #{request.fetch(:path)}-#{response.code}"

    response.send(client)
    client.close
  }
```

试着在`SERVER_ROOT` 目录下添加些 HTML
文件，然后你应该就能从浏览器加载它们了。其他一些包括图像等静态资源文件也同样如此。

当然了，真实世界中的 web 服务器包含许多我们未在这里实现的功能。

我在这里列出了**一些**缺失的功能，作为提高练习留给你们（熟能生巧！！！）：

- 虚拟主机
- 多媒体类型
- 数据压缩
- 访问控制
- 多线程
- 请求验证
- 解析查询字符串
- POST 请求体解析
- 浏览器缓存（响应 304 状态码）
- 重定向

## 安全相关

接受并操作用户的输入永远是一件有风险的事情。在我们的小服务器项目中，用户输入是[HTTP请求](https://www.rubyguides.com/2018/08/ruby-http-request/)。

我们引入了一个被称为“路径遍历”的漏洞。用户可以访问任何服务器应用用户有权限访问的路径，包括那些不在我们的 `SERVER_ROOT` 路径下的目录。

下面这行代码是导致此问题的关键：

```ruby
  FIle.binread(path)
```

你可以亲自实验一下，看看这问题是怎么产生的。为此你需要“手动”构造 HTTP
请求，因为大多数 [HTTP客户端](https://www.rubyguides.com/2018/08/ruby-http-request/)（包括 `curl` ）会预先处理请求 URL 并移除会导致此漏洞的部分。

你可以用 [netcat](https://en.wikipedia.org/wiki/Netcat/) 尝试一下。

以下是其中一种生成办法：

```bash
  $ nc localhost 8080
  GET ../../etc/passwd HTTP/1.1
```

如果你是类 Unix 系统的话，上述请求会返回 `/etc/passwd` 文件的内容。其中的原理是因为双点号（`..`）表示上层目录，于是你便“跳出“了 `SERVER_ROOT` 目录。

某一个解决办法是将多个点号”压缩成一个：

```ruby
  path.gsub!(/\.+/,".")
```

解决安全问题要时刻有“以己之矛，攻己之盾”的准备。例如，如果你只是简单的
`path.gsub!("..", ".")`，那么使用三重点号（...）便可以轻易绕过此限制。

## 完整可运行的代码

我知道本文中的代码四散在文章各处，所以如果你想得到完整，可运行的代码的话。。。

### 链接在此：


https://gist.github.com/matugm/efe0a1c4fc53310f7ac93dcd1f041f6c#file-web-server-rb

好好享用吧！

## 总结

在本文中，你学到了如何监听新连接、HTTP 请求是什么样子了和如何解析它们。同样地，你学会了如何用响应码和请求文件的内容（如何提供的话）构造响应。

最后，你学到了什么是“路径遍历”漏洞和如何避免它。

希望本文能对你有所帮助！不要忘了订阅我的博客（地址在下面），这样你就不会漏掉我的每一篇文章啦：）

https://www.rubyguides.com/