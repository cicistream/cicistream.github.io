---
layout: post
title: "Promise & Generator & Async Await 比较"
subtitle: "ES6中重要的异步函数"
author: "cici"
tags:
  - ES6
  - js异步
  - JavaScript
---

最近重读了ES6，想总结一下ES6这些异步函数的区别，如果你对这些函数一点都不了解，建议先读一下文章最后参考的几篇基础知识。
```javascript
async function async1() {
    console.log("async1 start");
    await  async2();
    console.log("async1 end");

}

async  function async2() {
   console.log( 'async2');
}

console.log("script start");

setTimeout(function () {
    console.log("settimeout");
},0);

async1();

new Promise(function (resolve) {
    console.log("promise1");
    resolve();
}).then(function () {
    console.log("promise2");
});

console.log('script end');
```
运行结果是？

在纸上写下你的答案，在输出结果之前，让我们看看js异步调用的知识。

### promise之前

JS的异步调用方式有3种：
- 回调函数
- 事件监听
- 发布/订阅

### Promise对象
Promise对象有以下两个特点。

- **对象的状态不受外界影响**。<br>
Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是Promise这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

- 一旦状态改变，就不会再变，**任何时候都可以得到这个结果**。<br>Promise对象的状态改变，只有两种可能：从pending变为fulfilled和从pending变为rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。如果改变已经发生了，你再对Promise对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

Promise也有一些缺点。<br>
首先，**无法取消Promise**，一旦新建它就会立即执行，无法中途取消。

其次，Promise 对象的错误具有“冒泡”性质，会**一直向后传递**，直到被捕获为止。也就是说，错误总是会被下一个catch语句捕获。如果 Promise 状态已经变成resolved，再抛出错误是无效的。**如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。**

第三，当处于pending状态时，无法得知目前**进展**到哪一个阶段（刚刚开始还是即将完成）。



### Generator函数

Generator 函数有多种理解角度。语法上，首先可以把它理解成，Generator 函数是一个状态机，封装了多个内部状态。

执行 Generator 函数会返回一个遍历器对象，也就是说，Generator 函数除了状态机，还是一个遍历器对象生成函数。返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。

由于 Generator 函数返回的**遍历器对象**，只有调用next方法才会遍历下一个内部状态，所以其实提供了一种可以**暂停执行**的函数。yield表达式就是暂停标志。

yield表达式本身没有返回值，或者说总是返回undefined。next方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值。

通过next方法的参数，就有办法在 Generator 函数开始运行之后，**继续向函数体内部注入值**。也就是说，可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。

Generator 函数返回的遍历器对象，都有一个throw方法，可以在函数体外抛出错误，然后在 Generator 函数体内捕获。如果 Generator 函数内部没有部署`try...catch`代码块，那么throw方法抛出的错误，将被外部`try...catch`代码块捕获。

一旦 Generator 执行过程中抛出错误，且没有被内部捕获，就**不会再执行下去**了。如果此后还调用next方法，将返回一个value属性等于undefined、done属性等于true的对象，即 JavaScript 引擎认为这个 Generator 已经运行结束了。

### Async Await
Async是 Generator 函数的语法糖。
async函数对 Generator 函数的改进，体现在以下四点。

1）**内置执行器**。

Generator 函数的执行必须靠执行器，所以才有了co模块，而async函数自带执行器。也就是说，async函数的执行，与普通函数一模一样，只要一行。

2）**更好的语义**。

async和await，比起星号和yield，语义更清楚了。async表示函数里有异步操作，await表示紧跟在后面的表达式需要等待结果。

3）**更广的适用性**。

co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是 **Promise 对象和原始类型的值**（数值、字符串和布尔值，但这时等同于同步操作）。

4）**返回值是 Promise**。

async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作。

async函数的错误处理机制是：只要一个await语句后面的 Promise 变为reject，那么整个async函数都会**中断执行**。最好把await命令放在`try...catch`代码块中。

### 最后

你可以先读一下这个 [Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)。

首先，js是**单线程**的，主要的任务是处理用户的交互，而用户的交互无非就是响应DOM的增删改，使用事件队列的形式，一次事件循环只处理一个事件响应，使得脚本执行相对连续，所以有了**事件队列**，用来储存待执行的事件，那么事件队列的事件从哪里被push进来的呢。

那就是另外一个线程叫**事件触发线程**做的事情了，他的作用主要是在**定时触发器线程**、**异步HTTP请求线程**满足特定条件下的回调函数push到事件队列中，等待js引擎空闲的时候去执行，当然js引擎执行过程中有优先级之分，首先js引擎在一次事件循环中，会先执行js线程的主任务，然后会去查找是否有**微任务microtask**（promise），如果有那就优先执行微任务，如果没有，在去查找宏任务**macrotask**（setTimeout、setInterval）进行执行。

运行结果揭晓，你写对了嘛？

```javascript
script start
async1 start
async2
promise1
script end
async1 end
promise2
settimeout
```

#### 参考文章
- [ES6 - Promise 对象（阮一峰）](http://es6.ruanyifeng.com/#docs/promise)
- [ES6 - Generator 函数（阮一峰）](http://es6.ruanyifeng.com/#docs/generator)
- [ES6 - async 函数（阮一峰）](http://es6.ruanyifeng.com/#docs/async)