---
layout: post
title: "关于Vue中nextTick()的思考"
subtitle: "nextTick是怎么利用JS中的异步方法的？"
author: "cici"
tags:
  - Vue源码
  - nextTick
  - js异步
  - JavaScript
---

　　我的项目中有一个swiper插件，在vue实例created(生命周期相关)函数中，先用ajax异步加载数据，再初始化swiper轮播插件时，遇到了一个问题，由于动态数据加载导致了swiper初始化后会滑动到最后一个item。我当时的解决方法是用`setTimeout()`来延迟初始化，之后在学习es6的时候，我发现更好的解决办法是使用`promise.then`.但是，没有最后只有更好，`promise.then`可能会有的兼容性问题，可以用Vue自带的`nextTick()`方法来解决。

`nextTick()`方法的官方说明是这样的:

> 在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

　　看起来非常适合解决我的问题，那`nextTick()`方法到底是怎样实现的呢，是否包括我之前思考过的两种解决方案呢，带着这样的思考我翻看了一下nextTick的源码:[Vue—nextTick](https://github.com/vuejs/vue/blob/471de4a31d229e681cc9dce18632b5bcab944c77/src/core/util/next-tick.js)
　　
我在看源码之前有看过一些解析nextTick的文章，他们一般描述nextTick使用了优雅降级的方式，内部其实是以下三种方式回调：

1. promise.then
2. Mutation.Observe
3. setTimeout(fn,0)

但是当我翻看源码时，发现更新后的源码是这样定义nextTick的：

1. **microtasks function**
2. **macrotasks function**

如果你对这两个名词感到困惑，可以读读[这个](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

　　官方源码中的解释是这样的：
　　
> Here we have async deferring wrappers using both micro and macro tasks.<br>
In < 2.4 we used micro tasks everywhere, but there are some scenarios where micro tasks have too high a priority and fires in between supposedly sequential events (e.g. #4521, #6690) or even between bubbling of the same event (#6566). However, using macro tasks everywhere also has subtle problems when state is changed right before repaint (e.g. #6813, out-in transitions).<br>
> 　　Here we use micro task by default, but expose a way to force macro task when needed (e.g. in event handlers attached by v-on).

　　简单来讲，就是以前(<2.4版本)我们总是使用`microtask`，不过他的优先级太高会导致一些问题（比如插入一些顺序执行的事件处理之中），但是完全使用`macrotask`也有一些麻烦，所以我们现在默认使用microtask，在需要的时候再强制使用`macrotask`。
　　<br>
　　为什么默认使用`microtask`呢，如下：

> 　　根据 HTML Standard，在每个 task 运行完以后，UI 都会重渲染，那么在 microtask 中就完成数据更新，当前task 结束就可以得到最新的 UI 了。反之如果新建一个 task 来做数据更新，那么渲染就会进行两次。

　　理解了这些知识后我们来详细的看源码。源码中`microtask`主要是使用**promise.then**
```javascript
// Determine MicroTask defer implementation.

if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  microTimerFunc = () => {
    p.then(flushCallbacks)
    if (isIOS) setTimeout(noop)
  }
} else {
  microTimerFunc = macroTimerFunc
}
```
　　macrotask则比较复杂，按照顺序依次是**setImmediate**->**MessageChannel**->**setTimeout(fn,0)**
```javascript
// Determine (macro) Task defer implementation.
// Technically setImmediate should be the ideal choice, but it's only available
// in IE. The only polyfill that consistently queues the callback after all DOM
// events triggered in the same loop is by using MessageChannel.

if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  macroTimerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else if (typeof MessageChannel !== 'undefined' && (
  isNative(MessageChannel) ||
  // PhantomJS
  MessageChannel.toString() === '[object MessageChannelConstructor]'
)) {
  const channel = new MessageChannel()
  const port = channel.port2
  channel.port1.onmessage = flushCallbacks
  macroTimerFunc = () => {
    port.postMessage(1)
  }
} else {
  
  macroTimerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
```
　　注释说的很明确，技术上`setImmediate()`应该是理想的选择，但它只能在IE中使用。在同一循环中触发的所有DOM事件之后，唯一一个对回调进行排队的polyfill是使用**MessageChannel**。这里给MessageChannel对象port1设置事件处理，再利用port2给port1传递消息来调用。注意，`setTimeout(fn,0)`即使第二个参数设为0，仍然会有最少4ms的延迟。(我目前还没有使用过MessageChannel对象,网上的资料好像也不多，如果读者有推荐的资料，请私信告诉我~
　　<br>
　　以上是我的一些总结，希望可以帮到遇到同样问题的萌新。