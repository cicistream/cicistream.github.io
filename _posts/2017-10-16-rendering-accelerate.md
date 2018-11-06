---
layout: post
title: "如何更快地构建与渲染"
subtitle: "加速构建可以从CSS/HTML/JS多方面入手"
author: "cici"
header-style: text
tags:
  - JavaScript
  - 性能
  - rendering
---

在[上一篇](https://cicistream.github.io/2017/10/06/rendering-process/)中，我详细描述了浏览器渲染的过程，那么，如何来提升渲染效率，使页面更快的加载完成呢。
我将这些方法分为几部分分别介绍。

### CSS部分

#### 1. **媒体类型和媒体查询**
 
将CSS分割为片段，对于不同的浏览器，不同的终端，不同的阅读模式，应用不同的CSS样式表。如果将这些内容写到统一个文件中，浏览器需要下载并解析他们。所以我们应该将这些内容通过对link元素的`media`属性来指定：

```javascript
<head>
<link rel="stylesheet" type="text/css" href="theme.css" />
<link rel="stylesheet" type="text/css" href="print.css" media="print"/>
<link rel="stylesheet" type="text/css" href="style.css" media="(min-width:40em)"/>
</head>
```

注意：无论是否需要解析使用，浏览器都会下载这些样式表。

#### 2. **编写高效的CSS**

关键选择器，是一个复杂的CSS选择器中**最右边部分**。它是浏览器最先寻找的。写CSS代码的时候，关键选择器是能否高效的决定因素。

```javascript
#content .intro {..}
```
浏览器会寻找.intro的实例（可能会很多），然后沿着DOM树向上查找，确定刚才找到的实例是否在一个带有ID为`content`的容器里面。

但是，下面的选择器就有点糟糕啦：
　　
```
#content * {..}
```

这个选择器所做的是选择所有在页面上的单个元素（是每个单个的元素），然后去看看它们是否有一个 `#content` 的父元素。这是一个非常不高效选择器因为它的关键选择器执行开销太大了。
更多高效选择器的知识戳[这里](http://blog.jobbole.com/35339/)

#### 3.**GPU加速**

利用CSS3 `transform：translate3d`开启硬件加速，提升网站动画渲染性能。更多相关知识戳[这里](http://www.cnblogs.com/PeunZhang/p/3510083.html)

### JS部分

#### 1. **异步加载**

DOM 扮演着两种角色：它既是HTML文档的对象表示，也充当着外界（比如JavaScript）和页面交互的接口。

js脚本的加载会阻塞DOM树的构建，因为有些js脚本需要操作DOM，这将改变DOM树。js脚本还可以查询关于 DOM 的一些东西，如果是在 DOM 还在在构建的时候，它可能会返回意外的结果。这些脚本如果在DOM树构建完成再操作，减少了对浏览器渲染速度的影响。

`defer` 和`async` 属性提供给开发者一个方式来告诉浏览器哪些**外部**脚本(包含src的脚本)是需要异步加载的。他们都告诉浏览器在“后台”加载脚本的同时继续解析 HTML，并在脚本加载完执行。

defer 和 async 之间的不同是他们开始执行脚本的时机的不同。

defer 比 async 要先引入浏览器。它的执行在**解析完全完成**之后才开始，它处在[DOMContentLoaded](https://developer.mozilla.org/zh-CN/docs/Web/Events/DOMContentLoaded)事件之前。 它保证脚本会按照它在 HTML 中出现的**顺序**执行，并且不会阻塞解析。

async 脚本在它们完成下载完成后的**第一时间执行**，它处在 window 的load 事件之前。 这意味着有可能（并且很有可能）设置了 async 的脚本不会按照它们在 HTML 中出现的顺序执行。这也意味着他们可能会中断 DOM 的构建。

设置async的脚本的加载有着较低的优先级。他们通常在所有其他脚本加载之后才加载，而不阻塞 DOM 构建。然而，如果一个指定async 的脚本很快就完成了下载，那么它的执行会阻塞 DOM 构建以及所有在之后才完成下载的同步脚本。

#### 2.**预加载**

Preloader 简介 ：

> 　　HTML 解析器在创建 DOM 时如果碰上同步脚本（synchronous script)，解析器会停止创建DOM，转而去执行脚本。所以，如果资源的获取只发生在解析器创建DOM时，同步脚本的介入将使网络处于空置状态，尤其是对外部脚本资源来说，当然，页面内的脚本有时也会导致延迟。

预加载器（Preloader）的出现就是为了优化这个过程，预加载器通过分析浏览器对 HTML文档的早期解析结果（这一阶段叫做“令牌化（tokenization）”），找到可能包含资源的标签（tag），并将这些资源的 URL收集起来。令牌化阶段的输出将会送到真正的 HTML 解析器手中，而收集起来的资源 URLs会和资源类型一起被送到读取器（fetcher）手中，读取器会根据这些资源对页面加载速度的影响进行有次序地加载。

`preload`使开发者能够自定义资源的加载逻辑，且无需忍受基于脚本的资源加载器带来的性能损失。通过这一方法我们告诉浏览器开始获取某一特定资源，毕竟我们是作者，知道浏览器很快就会用到这一资源。

　　你只需写上：
```javascript
<link rel="preload" href="very_important.js" as="script">
```
　　你可用as告诉浏览器你让他预加载文件的类型：

> "script" "style" "image" "media" "document"

　　忽略 as 属性，或者错误的 as 属性会使 preload 等同于 XHR 请求，浏览器不知道加载的是什么，因此会赋予此类资源非常低的加载优先级。

关于预加载，我们已经有`<link rel=“prefetch”>`,而且浏览器支持情况还不错。但是他的作用是告诉浏览器加载**下一页面**可能会用到的资源，不是当前页。

web 字体是较晚才能被发现的关键资源（late-discovered critical resources）中常见的一类 。有了 preload 这个标准，简单的一段代码就能搞定字体的预加载。

```javascript
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
```
　　需要注意的一点是：**crossorigin** 属性是必须的，即便是字体资源在自家服务器上，因为用户代理必须采用匿名模式来获取字体资源。

　
#### 参考文章：
1. [关于Preload，你应该知道些什么](http://www.jianshu.com/p/24ffa6d45087)
2. [更快地构建 DOM: 使用预解析, async, defer 以及 preload](http://www.jianshu.com/p/24ffa6d45087)