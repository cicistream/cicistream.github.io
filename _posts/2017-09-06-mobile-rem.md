---
layout: post
title: "移动端布局单位"
subtitle: "关于像素与font-size"
author: "cici"
tags:
  - mobile
  - 单位
  - css
---


移动端设备种类繁多，需要适配多种情况的响应式布局来保证美观的页面实现，先来解释容易弄混的多个名词。
---

 - **PPI**
 > 单位英寸像素数
 
 - **DPR**
 > 设备像素比： 设备像素 / CSS像素(某一方向上)

 - **DPI**
 >Dpi（每平方英寸像素数目）：图像细节程度的度量
 
 建议看一下[知识小科普！像素英寸与DPI的那些事儿](http://mp.weixin.qq.com/s?__biz=MjM5NTA0NjY4MA==&mid=203073243&idx=1&sn=c71ff9f0c0fb96fea2d3ea2b213018e1#rd)
 - **单位：**
 1. em
>    em是相对长度单位。相对于当前对象内文本的font-size(即**父元素尺寸**)。如当前对行内文本的字体尺寸未被人为设置，则相对于浏览器的默认字体尺寸。

2. rem
> 　“font size of the root element (根元素的字体大小)”--- W3C官网

　　也就是说1rem就等于`<html>`的字体大小，因为网页`<html>`的默认字体大小是 16px，所以 1rem=16px 。我在实习的公司前辈教我，用1rem=100px替换设计稿上的内容，不需要复杂的计算转换就可以将px转换为rem，从而达到适配的效果。

```javascript
!function(){
	function a(){
document.documentElement.style.fontSize=document.documentElement.clientWidth/7.2+"px"
}
var b=null;
window.addEventListener("resize",function(){
	clearTimeout(b),b=setTimeout(a,10)},!1),a()
}(window);
```
　　我们的设计稿宽为720，我们将浏览器可见区域/7.2得到100px的大小，将他赋值给根元素fontSize，他的值也就是1rem的大小。
　　详细的[各种宽高](http://caibaojian.com/js-name.html)可以看一下
　　
3. rpx

> 微信小程序CSS单位，可以根据屏幕宽度进行自适应

rpx实际上就是系统级的rem（把页面按比例分割750份,我们就不用手动设置JS改变根元素font-size啦）
>1rpx=window.innerWidth/750

4. vm 、vh、vmin、vmax

>  vw：视窗宽度的百分比 <br>
vh：视窗高度的百分比  <br>
vmin：当前较小的vw和vh  <br>
vmax：当前较大的vw和vh
