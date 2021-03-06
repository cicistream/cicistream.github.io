---
layout: post
title: "如何保持datePicker与服务器时间一致"
subtitle: "避免用户修改系统时间导致的时间组件bug"
date:  2020-08-07
author: "cici"
header-img: "img/browser-time.png"
tags:
  - React
  - antd
  - browser
---

-------

使用到的的技术栈：react + umi + antd + moment

-----

### 前因

项目中一个表单包含时间组件datePicker，选取的时间范围为包括今天在内的15天。

**BUG**：如果用户修改了当前系统时间，那么datePicker的今天就会同步变化，相应的选取范围也会变化

**期望**：datePicker采用服务器时间（网络标准时间），不会因为客户端时间的修改而变化。

### 解决过程

首先如果你的网站使用了HTTP协议，那么在如果你修改时间超出数字证书的有效期，浏览器会自动禁止你的访问

![image](/img/in-post/post-browser-time/certificate.png)

![image](/img/in-post/post-browser-time/wrong_time.png)

但是如果你更改的客户端时间在证书有效期内，这个拦截就不会出现。现在要解决的就是这个前提下出现的问题。

#### 考虑调整前端解决

我首先思考能不能直接校准datePicker的时间。

因为我们的项目使用的是antd的[datePicker](https://ant-design.gitee.io/components/date-picker-cn/)组件，所以我查了antd的文档和gitissues，更多的讨论是时区问题的时间校准（通过设置local或偏移量解决的）.

然后我发现时间组件依赖的是[moment](http://momentjs.cn/docs/)库，通过moment()获取当前时间，又顺藤摸瓜看了moment的文档和gitissues，发现了某个问题下的这个回答

![image](/img/in-post/post-browser-time/server-side.png)

所以没有办法仅通过前端解决，得考虑从接口返回/从接口header读取服务器时间。最终通过自己搜索和与小伙伴探讨，总结出了两个可行的办法。

### 最终方案

#### 1. 配合接口获取服务器时间，手动校准

从接口获取的服务器时间会有一定的误差，解决的方法有很多，由于我采用的时间不需要那么精确，就没有做相应的偏移。

这个方法暴露出的一个问题是，在我的使用场景下，必须手动设置时间组件的`defaultValue`和`disabledDate`，同时去掉“今天”“此刻”等时间组件当前时间相关的项。不同的使用场景需要定制化处理。

而且我们整个项目的时间格式化使用的都是`moment`库，我想用这个方法必须排查所有的时间组件来统一到服务器时间。

这个办法的缺点是工程量有些大，需要重新测试相关页面。不能一劳永逸。

#### 2. 不建议用户修改客户端时间，修改则跳转到提示页面

这个方法的优点是可以**一劳永逸**！

- 第一次打开网页时，从接口返回的header里取服务器时间，记录一下服务器和客户端的时间差值，如果时间差值过大，则判断为客户端时间被修改了。
- 另外，设置一个函数每2s（自定义）获取一下客户端时间并记录，每次比较上一个时间和当前时间的差值，如果这个差值大于某个值（自定义）也可以判断为客户端时间被修改了。

当客户端时间被修改时，统一跳转到一个提示页面，建议用户设置正确的客户端时间。

当使用这个方法时，记得定义一下local，否则如果别的时区的用户登录系统，可能会被认为是错误的客户端时间。

**缺点**：对用户不太友好，修改了客户端时间就会无法继续当前操作。


