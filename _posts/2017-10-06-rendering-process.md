---
layout: post
title: "完整渲染过程——从URL到页面"
subtitle: "页面渲染过程发生了什么?"
author: "cici"
tags:
  - 页面渲染
  - rendering
---

当用户输入一串URL（统一资源定位符）到完整的页面渲染出来，到底经历了什么呢。

首先列出大概流程：
1.  浏览器根据请求的URL交给**DNS域名解析**，找到真实的IP，向服务器发起请求
2.  服务器交给后台处理完后返回数据，浏览器接受文件（HTML/CSS/JS/图像等）
3.  浏览器对加载到的数据进行解析，建立起相应的内部结构
4.  载入解析到的资源文件，渲染页面，完成

我们主要分为两部分具体解释，一部分重点讲解**URL**，另一部分讲解**渲染过程**。

### 0.输入URL后

首先，浏览器会开启一个线程来处理这个请求，对URL分析判断，如果是http协议就按照Web方式来处理，如果是本地文件就不用发送请求啦。

然后，调用浏览器内核中的相应方法，比如Web中的loadUrl方法；

### 1.缓存

我们会查看**浏览器内部缓存**，系统缓存，路由器（有时候也叫DNS缓存）缓存，看是否需要向服务器请求全部数据，如果查到域名对应的ip，就根据缓存的具体情况，判断从服务器端请求文件还是从本地读取文件。不同的浏览器默认缓存保留时间也有不同。具体可以参看我的另一篇[博客](https://cicistream.github.io/2018/08/28/browser-cache/).

![http cache](/img/cache.jpg)

### 2.域名解析

如果没有从本地找到对应ip，我们需要进一步解析域名。

首先你要了解，URL是统一资源定位符。由于IP地址很长且难以记忆，直接通过IP不方便，我们使用URL访问网站，并发明了DNS（域名系统）服务器，来解析域名为IP地址。抽象的来说，DNS就像是一个记录IP地址的超级分布式数据库。

当我们输入www.cici.cn访问网站，实际上是访问域名表示的IP地址。在这段URL中,”cn”是顶级域，她和”cici”（二级域）合起来可以称为主域名（一级域名），”www”是三级域名。”www.cici.cn”就是当前域名（二级域名）。  <br>
我们将当前域名交给DNS服务器解析。首先会发送查询请求给本地DNS服务器，如果在本地DNS服务器中查询不到指定的域名，他就会向其他根域名服务器继续发送查询请求（替主机继续查询）。其他根服务区或者返回查到的IP，或者告诉本地服务器”我没查到，你下一步可以问问那个谁（别的域名服务器）”。 

简单提一下，域名解析时本地DNS发送请求使用无连接的UDP协议，广播式发送，速度快但是不是完全可靠。其他情况以及TCP/UDP的比较可以戳[这里](http://www.cnblogs.com/549294286/p/5172435.html).


### 3.建立连接

如果上一步成功返回IP地址，那么我们可以接下来的操作。在服务器端与浏览器建立链接，这需要
[TCP三次握手](https://img-blog.csdn.net/20170104214009596?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2h1c2xlaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)<br>
![http cache](/img/tcp-connect.jpg)<br>
这部分可以插入一些[cookie & session](https://zhuanlan.zhihu.com/p/27669892?utm_source=com.daimajia.gold)的知识。<br>
建立好可靠的TCP连接，客户端就可以发送http请求啦。服务端处理好请求后，将需要的资源发送回来。

### 4. 服务器处理

服务器接（如Apache、Node.JS等）收到了浏览器发送的请求后，根据某个协议，通过web-server把浏览器发送的数据进行打包（包含请求头，ip地址，请求路径和查询参数等）。 

web-server把数据打包后，发送给后端应用。 

进入到部署好的后端应用，找到对应的请求处理。后端服务软件会根据路径和查询参数进行相应处理，返回给浏览器对应的数据包（包括http协议组成的代码。里面包含页面的布局、文字。数据也可能是图片、脚本程序，反应头，反应数据，请求头等）

### 5. 浏览器渲染

终于可以接受数据啦，在浏览器下载html文档，同时使用缓存。 

浏览器将HTML字节数据经过一个流程解析为DOM树：
> 字节->字符->令牌->节点对象->对象模型

当HTML代码遇到`<link>`标签时，浏览器会发送请求获得该标签中标记的CSS文件。令人欣喜的是，这是一个异步请求，不影响其他操作，浏览器获得数据后就会像构建DOM树一样构建CSSOM树。

DOM树描述了文档的结构与内容，CSSOM树描述了对文档应用的样式规则，想要渲染出页面，就需要将DOM树和CSSOM树结合在一起，成为渲染树。

　　浏览器会先从DOM树的根节点开始遍历每个可见节点（不包括`display:none`,包括`visibility:hidden`），对每个可见节点找到配适的样式规则，最后渲染树构建完成。
　　
下一步就是布局阶段,计算每个节点在窗口内的确切位置与大小。

布局阶段的输出是一个盒子模型，精确计算出每个元素的位置与大小，将测量值转换为像素值。当布局事件完成后，就可以渲染啦，渲染出整个页面给用户。 

这是比较理想状态下，没有考虑浏览器渲染阻塞的情况。实际应用中，HTML/CSS/JS资源都可能对渲染过程造成阻塞。

HTML是构建DOM必须的。

如果CSS过大，虽然下载异步，但下载过慢，会造成渲染的暂停等待，因为没有CSS样式是无法对DOM进行渲染的，避免渲染一部分后，CSSDOM树构建好了，DOM又得从头渲染，用户可能会看到闪烁的**回流重绘**过程。

如果将JS文件放在`<head>`中，当浏览器的HTML解析器遇到`<script>`标签会暂停构建DOM，然后将控制权移交给JS引擎，这是引擎会开始执行JS脚本，直到执行结束后，浏览器才会从刚刚断开的地方恢复，继续构建DOM。
>为什么解析必须要暂停呢？
    那是因为脚本可以改变 HTML以及它的产物 —— DOM。 

如果这个JS脚本还操作了CSSOM树，而正好CSSOM树还没有下载和构建，浏览器甚至会先下载并构建好CSSOM树。显而易见，JS脚本的执行位置不当会严重影响渲染速度。 

　　这就是为什么我们常常将外部css文件放在head中，而将js文件放在body最后。更多帮助js延迟加载提高浏览器性能的知识，戳这个->[更快地构建 DOM](https://mp.weixin.qq.com/s/NRLdvloo8CQfNJeP0016zg?)。

---

到这里，整个从URL到页面渲染的过程就结束啦，我将自己看到的一些文章和知识捏合汇总，整个过程里面包含着许多网络和渲染的知识并没有一个一个写明，需要大家自己搜索汲取。 
