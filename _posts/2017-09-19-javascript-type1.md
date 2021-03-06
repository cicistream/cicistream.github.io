---
layout: post
title: "基本包装类型"
subtitle: "一些基础知识"
author: "cici"
tags:
  - JavaScript
---

好久之前看的高程，最近有些忘记了，疑惑基本包装类型存在的意义，下面总结一下

----------

> 为了便于操作基本类型，ECMAScript提供了三个特殊的引用类型：Boolean、Number、String。

　　实际上，每当读取一个基本类型值的时候，后台就会创建一个对应的基本包装类型的对象，从而让我们能够调用一些方法来操作这些数据。
　　
```javascript
var s1="nice";
var s2=s1.substring(2);
```
　　在你写出上面的代码时，后台都会完成如下操作：
　　
1. 创建String类型的一个实例
2. 在实例上调用指定方法
3. 销毁这个实例
 
 可以想象为：
```javascript
var s1=new String("nice");
var s2=s1.substring(2);
s1=null
```
经过此番处理，基本类型的字符串值就变得像对象一样了。

引用类型与基本包装类型的主要区别就是**对象的生存期**。使用new操作符创建的引用类型的实例，在执行流离开当前作用域之前一直都保存在内存中。而自动创建的基本包装类型的对象，只存在一行代码的执行瞬间，然后立即被销毁。这意味着我们不能在运行时为基本类型添加属性和方法。

当然可以显示创建基本包装类型，对这样的实例调用typeof会返回“object”，而且基本包装类型的对象在装换为布尔类型值的时候会返回true。