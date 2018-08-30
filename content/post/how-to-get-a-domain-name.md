---
title: "如何拥有一个域名"
date: 2017-12-01T21:46:44+08:00
toc: true
author: "zvector"
author_homepage: "https://zvector.tk/"
tags: ["domain name", "memo"]
categories: ["技术备忘录"]
CreativeCommons: true
draft: false
---

## 前言

本篇是我当初搭建个人博客时所经历的步骤，在此记录。话不多说，让我们先看看什么是域名：

> **域名**（英语：**Domain Name**），简称**域名**、**网域**，是由一串用点分隔的名字组成的[Internet](https://zh.wikipedia.org/wiki/%E5%9B%A0%E7%89%B9%E7%BD%91)上某一台[计算机](https://zh.wikipedia.org/wiki/%E8%AE%A1%E7%AE%97%E6%9C%BA)或[计算机组](https://zh.wikipedia.org/w/index.php?title=%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BB%84&action=edit&redlink=1)的名称，用于在数据传输时标识计算机的电子方位（有时也指地理位置）。--wikipedia

简单来说，域名是为了避免直接记忆IP地址而诞生，域名有一级域名，二级域名乃至三级域名等区分，我们最熟悉的莫过与www开头的二级域名。相关谈论可以在[这个问题](https://www.zhihu.com/question/20414602)中看到。相应的我们需要DNS服务来解析域名。

> [网域名称系统](https://zh.wikipedia.org/wiki/%E7%BD%91%E5%9F%9F%E5%90%8D%E7%A7%B0%E7%B3%BB%E7%BB%9F)（[DNS](https://zh.wikipedia.org/wiki/DNS)，Domain Name System，有时也简称为域名）是因特网的一项核心服务，它作为可以将域名和[IP地址](https://zh.wikipedia.org/wiki/IP%E5%9C%B0%E5%9D%80)相互[映射](https://zh.wikipedia.org/wiki/%E6%98%A0%E5%B0%84)的一个分布式数据库，能够使人更方便的访问互联网，而不用去记住能够被机器直接读取的[IP地址](https://zh.wikipedia.org/wiki/IP%E5%9C%B0%E5%9D%80)数串。  --wikipedia

域名注册可选国内，也可选国外。国内注册需要备案，域名解析服务选择了国内的话访问速度会比较快。再次不赘述，本文选择[freenom](http://www.freenom.com/en/index.html?lang=en)注册一个.tk的免费域名，以做学习练习使用。

## 注册登录

在[freenom](http://www.freenom.com/en/index.html?lang=en)使用电子帐号注册登录，含有确认邮件，此处略过。

## 选择域名

在[freenom](http://www.freenom.com/en/index.html?lang=en)的搜索栏选择一个想要的域名名称，检查有没有被抢注。可以选择持有日期，我们选择12个月。选好了就可以Check Out了，会跳转到注册登录界面，登录后到邮箱中点击确认链接后完成注册域名。在[My Domains](https://my.freenom.com/clientarea.php?action=domains)界面我们可以查看域名信息，设置域名服务器等。接下来我们设置域名服务器

## 域名服务器设置

进入Nameservers页面，设置DNS解析。我们选择freenom自带的域名解析就行，也就是默认设置。或者可以设置为其他的DNS解析服务商地址，这方面教程已经有很多了，可自行搜索。关于DNS解析的记录类型，有如下几种：

> - 主机记录（A记录）：RFC 1035定义，A记录是用于名称解析的重要记录，它将特定的主机名映射到对应主机的IP地址上。
> - 别名记录（CNAME记录）: RFC 1035定义，CNAME记录用于将某个别名指向到某个A记录上，这样就不需要再为某个新名字另外创建一条新的A记录。
> - IPv6主机记录（AAAA记录）: RFC 3596定义，与A记录对应，用于将特定的主机名映射到一个主机的[IPv6](https://zh.wikipedia.org/wiki/IPv6)地址。
> - 服务位置记录（SRV记录）: RFC 2782定义，用于定义提供特定服务的服务器的位置，如主机（hostname），端口（port number）等。
> - NAPTR记录：RFC 3403定义，它提供了[正则表达式](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)方式去映射一个域名。NAPTR记录非常著名的一个应用是用于[ENUM](https://zh.wikipedia.org/w/index.php?title=ENUM&action=edit&redlink=1)查询。  --wikipedia

以上。
