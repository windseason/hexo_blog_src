---
title: URL&URL encoding
date: 2015-10-15 23:42
tags: http
---

## 介绍
当我们浏览网页的时候，其中涉及到不少技术。其中，有一项最基本的机制使得我们可以顺利的在浩瀚的互联网海洋中快速的定位到我们想要的资源并接收数据。这项技术是网页的根本，是形成互联网的基石。它就是URL。

URL 英文全名 Uniform resource locator，翻译过来就是统一资源标识符。例如，[”http://www.baidu.com”](http://www.baidu.com) 是一个URL。关于URL怎么定义，该遵守什么规则，是有着一套世界公认的规则的，[第一版规则是在1994年诞生的](http://tools.ietf.org/html/rfc1738)。

## 一般URL语法
我们先来看一个例子： 
`https://jay:123456@www.example.com:8080/file;p=1?q=2#section2`

分解该URL，我们能够获得以下信息：

| 组件名        | 数据          |
| :------------- |-------------:|
| scheme     | https |
| user     | jay      |
| password | 123456      |
| host address | www.example.com      |
| port | 8080     |
| path | file     |
| path parameter | p=1     |
| query parameter | q=2     |
| fragment | section2     |

## HTTP URL 语法

HTTP URL的scheme通常组成为

1. Scheme: http或者https
2. Path: 指定获取数据的路径
3. Query参数（可选）
4. fragment参数（可选）


## Path 与 Path parameters

其中，Path跟我们熟知的文件路径非常类似。Path通常是以一个“/”开头，每一个文件夹又都被一个”/”互相分隔，例如”/Study/URLSyntax/starter/first.docx”分为四个部分：”study”，”URLSyntax”，”starter”，”first.docx”。

每一个Path部分后都能够跟一个可选的参数(path parameters)部分，判断的根据是以”;”字符开始。参数可以有多个，并以”;”分隔。每个参数都有值，以”=”分隔，例如”/file;a=1”，定义了”file”这个path的参数为”a”，值为”1”。这些参数是很少用到的，但是他们是确实存在的。

## Query

在Path后面以”?”开始的部分就是Query了，Query包含一组以”&”分隔的参数与数值对。参数及其数值以”=”分隔。例如，”?url=http://domain.tld/&title=The title of a post”，定义了两个参数”url”和”title”，他们的值分别为”http://domain.tld“和”The title of a post”。Query 用的最多的情况是提交一个form的时候或者是搜索的时候。

## Fragment

Fragment是用来定位到当前页面某一个具体位置的url部分。在url部分以”#”开始的部分为fragment。

## URL encoding

好了，聊完URL基本组成部分，接下来就要介绍URL encoding了。经常上网的同学一定对下面这种URL不陌生吧？ 
[https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=1&rsv_idx=2&tn=baiduhome_pg&wd=%E7%BA%A2%E7%B1%B3note2&rsv_spt=1&oq=green&rsv_pq=947783b70000](https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=1&rsv_idx=2&tn=baiduhome_pg&wd=%E7%BA%A2%E7%B1%B3note2&rsv_spt=1&oq=green&rsv_pq=947783b70000)

可以看到，该URL包括了一些神秘的字符，例如”wd=%E7%BA%A2%E7%B1%B3note2”，一般的人是无法读的。这就是被编码后的样子了。

为什么给URL 编码呢？原因很简单，因为如果允许任意字符的话，解析URL的任务将会非常艰巨而且效率低下，比如URL允许中文或者阿拉伯文。所以W3C组织就规定了一套协议，现在最新的应该是[RFC 3986](https://tools.ietf.org/html/rfc3986)，有兴趣深入研究的同学推荐阅读。编码就是对URL每个部分，不允许出现的字符进行编码。那么允许的字符有哪些呢？如下所示： 
![](http://img.blog.csdn.net/20151016130703641)
摘取自:[https://en.wikipedia.org/wiki/Percent-encoding#Types_of_URI_characters.](https://en.wikipedia.org/wiki/Percent-encoding#Types_of_URI_characters.)

URL encoding介绍的差不多了，现在来讲讲URL encoding需要注意的问题。

## URL encoding 陷阱

RFC标准中没有指明使用哪种特定的编码格式来对url 进行编码。一般来说ASCII字符都是可以被使用的，但是对于保留字符(reserved characters)和非ASCII字符，我们必须考虑需要使用哪种编码方式来进行编码。最新的RFC标准是使用UTF-8来进行编码。但是，开发者也许也会遇到不是用UTF-8编码的时候。。。

对于HTTP URTs来说，空格符号在Path部分会被编码为”%20”，而”+”是不用被编码的。 
对于Query部分来说，空格符号会被编码为”+”或者”%20”，而”+”会被编码为”%2B”。

这样就意味着一个字符串在不同的部分就会被编码为不同的形式。例如”blue+light blue”字符串。我们来看一个具体的例子：http://example.com/blue+light%20blue?blue%2Blight+blue“。从这个例子可以看出如果对URL语法不熟悉的话，是无法对一个URL进行编码的。要对URL进行编码，就需要知道每一部分对应的保留字符是哪些而不是简单的对URL进行遍历并编码。

每个部分的保留字符

- “?”在Query部分是不用被编码的
- “/”在Query部分也是不用被编码的
- “:@-._~!$&’()*+,;=”在Path部分是不用被编码的
- “/?:@-._~!$&’()*+,;=”在fragment部分是不用被编码的


## 解码后的URL无法被正确分析
通过一个例子来分析更清楚：
`”http://example.com/blue%2Fred%3Fand+green” `

该URL有如下几个部分：

| 部分        | 值          |
| :------------- |-------------:|
| scheme     | http |
| host     | example.com |
| path     | blue%2Fred%3Fand+green |
| 解码后的path     |blue/red?and+green |

分析后，很明显，我们是在找一个”blue/red?and+green”的资源，而不是找一个在blue文件夹下的”red?and+green”的资源。

如果我们首先对URL解码，然后再分析呢？解码后的URL:
`”http://example.com/blue/red?and+green”`

| 部分        | 值          |
| :------------- |-------------:|
| scheme     | http |
| host     | example.com |
| path     | blue/red |
| query parameter name    |and+green |

很明显，这个是错误的。这个例子告诉我们，分析并拆解一个URL必须要在解码之前进行。

## 解码后的URL有可能无法被编码回原来的格式

如果对”http://example.com/blue%2Fred%3Fand+green“解码，你将得到”http://example.com/blue/red?and+green“，然后又进行编码，你将得到”http://example.com/blue/red?and+green“，因为这个是合法的URL不需要进行character escape。

参考原文： 
[http://blog.lunatech.com/2009/02/03/what-every-web-developer-must-know-about-url-encoding](http://blog.lunatech.com/2009/02/03/what-every-web-developer-must-know-about-url-encoding)
