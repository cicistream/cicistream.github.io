---
layout: post
title: "有关浏览器缓存 200vs304"
subtitle: "浏览器刷新到底需不需要重新请求呢？"
author: "cici"
tags:
  - cache
  - 304
  - 缓存
  - 刷新
---

## 基本概念

 - **Etag**：web服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器决定）。
 - **If-None-Match**：当资源过期时（使用Cache-Control标识的max-age），发现资源具有Etage声明，则再次向web服务器请求时带上头If-None-Match （Etag的值）。web服务器收到请求后发现有头If-None-Match 则与被请求资源的相应校验串进行比对，决定是否命中协商缓存；
 - **Last-Modify/If-Modify-Since**：浏览器第一次请求一个资源的时候，服务器返回的header中会加上Last-Modify，Last-modify是一个时间标识该资源的最后修改时间；当浏览器再次请求该资源时，request的请求头中会包含If-Modify-Since，该值为缓存之前返回的Last-Modify。服务器收到If-Modify-Since后，根据资源的最后修改时间判断是否命中缓存。
 - **Expires**：response header里的过期时间，浏览器再次加载资源时，如果在这个过期时间内，则命中强缓存。
 - Cache-Control：当值设为max-age=300时，则代表在这个请求正确返回时间（浏览器也会记录下来）的5分钟内再次加载资源，就会命中强缓存。
 - -no-cache：不使用本地缓存。需要使用缓存协商，先与服务器确认返回的响应是否被更改，如果之前的响应中存在ETag，那么请求的时候会与服务端验证，如果资源未被更改，则可以避免重新下载。
 - -no-store：直接禁止浏览器缓存数据，每次用户请求该资源，都会向服务器发送一个请求，每次都会下载完整的资源。

#### 浏览器缓存过程
![浏览器缓存过程](https://upload-images.jianshu.io/upload_images/6038331-613485beb848e6a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/554)

 1. 初始状态

 第一次访问时，向服务器发送请求，成功收到响应，返回**200**，浏览器下载资源文件，并记录下response header和返回时间。

 2. 再次请求相同资源

 再次访问相同资源时，本地先判断是否需要发送请求给服务端：
  - 比较当前时间和上一次返回200时的时间差，如果未超过过期时间，则**命中强缓存**，读取本地缓存资源
  - 如果过期了，则向服务器发送header带有**If-None-Match**和**If-Modified-Since**的请求
  
3.服务器收到请求后，**优先根据Etag**的值判断被请求的文件有没有做修改，Etag值一致则没有修改，**命中协商缓存**，返回**304**；如果不一致则有改动，直接返回新的资源文件带上新的Etag值并返回**200**；

4.如果服务器收到的请求没有Etag值，则将If-Modified-Since和被请求文件的**最后修改时间**做比对，一致则**命中协商缓存**，返回**304**；不一致则返回新的last-modified和文件并返回**200**；

## 两种刷新对页面的影响

- 点击刷新按钮或者按F5
浏览器直接对本地的缓存文件过期，但是会带上If-Modifed-Since，If-None-Match,这就意味着服务器会对文件检查新鲜度，返回结果可能是304，也有可能是200.

- 用户按Ctrl+F5（强制刷新）
浏览器不仅会对本地文件过期，而且不会带上 If-Modifed-Since，If-None-Match，相当于之前从来没有请求过，返回结果是200.

### 参考文章：

- [强制缓存和协商缓存有什么区别](https://www.jianshu.com/p/1a1536ab01f1)