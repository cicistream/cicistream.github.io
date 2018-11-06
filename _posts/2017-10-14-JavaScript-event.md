---
layout: post
title: "JS——事件绑定与处理总结（上）"
subtitle: "不同内核浏览器的事件处理方式有差异"
author: "cici"
header-style: text
tags:
  - JavaScript
  - 事件
---

　　JavaScript与HTML之间的交互是**通过事件实现**的。
> 事件是指文档或浏览器窗口发生的一些特定的交互瞬间，可以用监听器（或处理程序）来预定事件，以便在事件发生时执行相应的代码。——摘自《高程》

理解事件是实现良好交互的必要条件，本文旨在系统地**总结**事件相关知识。

(上)主要内容：

 - 理解事件流
 - 使用事件处理程序
 
### **1.理解事件流**

**事件流**描述的是从页面中接收事件的顺序。

首先你应该明白存在两种完全相反的事件流概念。一个是自下而上的事件**冒泡**流（IE提出），另一个是自上而下的事件**捕获**流（Netscape提出）。

![事件触发](/img/event.png)

图为“DOM2级事件”规定的事件流，它包括三个过程：事件捕获阶段、处于目标阶段和事件冒泡阶段。

在DOM事件流中，实际的目标在捕获阶段不会接收到事件，捕获阶段到`<body>`就停止了，下一阶段是目标阶段，事件在`<div>`上发生，并在事件处理中被看做是冒泡阶段的一部分（所以，是否取消冒泡与事件的发生有密切联系）

但是，实际上，多数支持DOM事件的浏览器（IE9+,Safari，chrome、Firefox和Opera9.5+）都会在捕获阶段触发事件对象上的事件，因此，就有两个机会在目标对象上操作事件啦。

### 2.事件处理程序
事件处理程序绑定方法主要有两种：在HTML标签中和在JavaScript脚本中。

在**HTML**标签中绑定事件有两种方法：

1. **纯HTML**  

```javascript
<input type="button" value="Click me" onclick="alert('Clicked')">
```
这种方法注意不能在其中使用未经转义的html语法字符——（& “”< >）
　　
2. **HTML+函数**

```javascript
<input type="button" value="Click me" onclick="showMessage()">
```

即使showMessage()函数是外部文件的也可以，事件处理程序代码在执行时有权访问全局作用域中任何代码。

一般不建议在HTML中添加事件处理，因为绑定的this指向全局作用域。

在脚本中添加的事件处理程序分为三种：DOM0级、DOM2级和IE
　　
1. **DOM0级事件处理的创建与删除**

```javascript
var btn=document.getElementById("myBtn");
btn.onclick=function(){
    alert(this.id);    //"myBtn"
}
btn.onclick=null;
```
　　使用DOM0级事件处理程序被认为是元素的方法，注意例中的this，以这种方法添加的事件是在元素的作用域中运行的，且在事件**冒泡阶段**被处理。

这种方法同一个事件只能添加**一个**事件处理程序。
　　
2. **DOM2级事件处理程序的创建与删除**

```javascript
var btn=document.getElementById("myBtn");
var handler=function(){
    alert(this.id);
    };
btn.addEventListener("click",handler,false);
btn.addEventListener("click",function(){
    alert("Hello Cici!");
},false);
btn.removeEventListener("click",handler,false);
```
`addEventListener()`和`removeEventListener()`接受三个参数：要处理的事件名、作为事件处理的函数和一个布尔值（默认为false，true表示在捕获阶段调用事件处理函数，false表示在冒泡阶段调用事件处理函数）。

注意，如果想用`removeEventListener()`移除事件，一定要保证三个参数与添加事件时完全一致，所以`addEventListener()`添加的**匿名函数**是无法被移除的，如上例中给“click”添加的第二个事件处理程序。

使用DOM2级最大的好处是可以添加**多个**事件处理程序，他们会按照**添加的顺序**触发。
　　

3. **IE事件处理程序的创建与删除**

```javascript
var btn=document.getElementById("myBtn");
var handler=function(){
    alert("Hi Cici~");
    };
btn.attachEvent("onclick",handler);
btn.attachEvent("onclick",function(){
    alert("Best Cici!");
});
btn.detachEvent("onclick",handler);
```

IE8及更早版本只支持事件冒泡，`attachEvent()`和`detachEvent()`接受两个参数，事件处理程序名称和函数。注意第一个参数与DOM不同，需要加"on"。<br>
IE中使用`attachEvent()`与DOM2级事件处理程序的相同之处是，不能移除添加的是**匿名函数**的事件处理函数。

他们的主要区别在于事件处理程序的**作用域**。这一点在写跨浏览器的代码时肥肠重要！！！在使用`attachEvent()`方法的情况下，事件处理程序会在**全局作用域**运行，也就是说，此时**`this===window`**。

还有一点不同就是，`attachEvent()`也可以添加**多次**事件处理程序，但是触发顺序与添加顺序**相反**。

知道了以上知识，我们可以着手写一个跨浏览器的事件处理程序了。

```javascript
var EventUtil = {
	addHandler:function(element,type,handler){
	    if(element.addEventListener){
			element.addEventListener(type,handler,false);
		}else if(element.attachEvent){
			element.attachEvent("on"+type,handler);
		}else{
			element["on"+type]=handler;
		}
    },
    removeHandler:function(element,type,handler){
	    if(element.removeEventListener){
			element.removeEventListener(type,handler,false);
		}else if(element.dtachEvent){
			element.dtachEvent("on"+type,handler);
		}else{
			element["on"+type]=null;
		}
    },
}
```

　　

　　