---
title: '[C#] TOC Tiny - 基于TCP套接字简易网络聊天室'
slug: '20201231010808'
published: 2020-12-31T01:08:08
tags:
  - dotnet
  - csharp
category: '.NET'
description: '简介:之前开了一个大坑, 额, 其实就是带有注册登录, 然后完美解决粘包, 心跳包, 还有并发量等问题的坑, 但是太难填了(我太菜了), 于是我就开了一个新项目, 砍掉登陆注册功能, 直接作为聊天室开放…程序美照:难题是如何解决的:首先是传输协议, 我这里的数据传输, 无论是什么数据, 都是一个TransPackage实例, 里面有基本的结构, 就是Name, Content, ClientGuid, PackageType这四个字段(除此之外就没了). 然后把它们弄成字符串, 然后弄'
---

## 简介:

- 之前开了一个大坑, 额, 其实就是带有注册登录, 然后完美解决粘包, 心跳包, 还有并发量等问题的坑, 但是太难填了(我太菜了), 于是我就开了一个新项目, 砍掉登陆注册功能, 直接作为聊天室开放...

## 预览:

![客户端连接界面](/images/20201231004509995.png)

![客户端主页面](/images/20201231004532309.png)

![添加有颜文字](/images/20201231004603771.png)

![服务端](/images/20201231004652925.png)
**难题是如何解决的**:

1. 首先是传输协议, 我这里的数据传输, 无论是什么数据, 都是一个TransPackage实例, 里面有基本的结构, 就是Name, Content, ClientGuid, PackageType这四个字段(除此之外就没了). 然后把它们弄成字符串, 然后弄成字符数组, 然后发送.
2. 关于粘包, 也就是两个包黏在一起, 我这里使用的是自己封装好的事件驱动的套接字, 粘包问题发生时, 就是在一次收到消息时, 收到了两个或以上的包, 而每个包都是一个Json数据, Json数据的特点呢, 就是很容易能分开, 例如你Object并列, 没有分隔符, 也能明确的知道这是两个Object, 于是我又为我自己造的轮子 [CHO.Json](/posts/20201028235244/) 优化了一波, 使它支持这个功能
3. 关于并发, 其实这个问题在你发现TCP时全双工的时候就不需要考虑了, 因为是聊天室, 所以也不必担心什么. 之所以我考虑这个, 是因为我在很久之前写的聊天小程序, 由于我把TCP当成半双工用, 导致不能太过频繁的处理消息, 否则就可能会嗝屁, 但这次有我封装好的事件套接字, 就方便多了



## 更新记录:

最新版已经支持发送图片, 以及对图片的查看(旋转缩放移动) 


## 下载链接：

- [CSDN下载](https://download.csdn.net/download/m0_46555380/13987649)
