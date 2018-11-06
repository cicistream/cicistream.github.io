---
layout: post
title:  "菜鸟系列——forEach、map、for...in、for...of"
subtitle: "常用的数组方法"
author: "cici"
tags:
    - JavaScript
    - 数组方法
---

作为一个前端小菜鸟，有好多容易弄混的名词
我想把搞清楚的一一记录下来，帮助记忆

`forEach`、`map`、`for...in`、`for...of`
这四个方法都是可以遍历数组或类数组的，很容易就懵了，必须总结一下他们的使用对象和区别，加以区分。
首先，`map()`和`forEach()`是Array自带的方法，Map对象也有`forEach()`方法，而`for...in`和`for...of是`对数组/类数组元素进行for循环操作的方法。

也就是说，在使用map()和forEach()的时候，需要用数组调用。(先只说数组的forEach())

```javascript
var arr=[1,2,3,4,5];
arr.map();
arr.forEach();
for( k in arr ){};
for( k of arr ){};
```

这么说来，我们可以把他们分为两组分别介绍

### 第一组：map()和forEach()

- 相同点：

1. 只能遍历数组，且循环遍历数组中的每一项
2. 包含的匿名函数都是三个参数(item,index,input)（当前项，当前项的索引，原始数组）
3. 在IE6-8下都不兼容

- 不同点：

1. `map()`有返回值，可以用'链式'连接；`forEach()`无返回值，写了也没用

举个栗子吧~ 建议不要只看看，自己多尝试一下

```javascript
var arr = [1,2,3,4];
var res1 = arr.map(function(item,index,input){
      return input[index] = item*2;   //返回每一项哦
})
console.log(res1);   //[2,4,6,8]
console.log(arr);    //[2,4,6,8]
 // 如果不想改变原数组，可以return item*2；那么最后arr=[1,2,3,4]
var arr = [1,2,3,4];
var res2 = arr.forEach(function(item,index,input){
      return input[index] = item*2;
})
console.log(res2);   //undefined
console.log(arr);    //[2,4,6,8]
 // 如果写return item*2，原数组也不会改变
 ```
 
forEach()不能使用continue, break;
map()速度最快.

作为 Map 的forEach方法时，与数组的forEach方法类似，也可以实现遍历。

```javascript
map.forEach(function(value, key, map) {
  console.log("Key: %s, Value: %s", key, value);
});
```

forEach方法还可以接受第二个参数，用来绑定this。

### 第二组：for...in和for...of

em....

`for...in`的毛病真是太多了，直接别用了，他作为一个for循环，不能保证**遍历的顺序**是预期的，还只能访问索引还会把数组属性遍历，实在是很糟糕，唯一能替代`for...of`的场景就是循环普通对象（`for...of`不能循环没有**遍历器**（Iterator）的对象）

`for...of`可以循环具备 Iterator 接口的数据结构，如下

    ES6中定义的原生具备 Iterator 接口的数据结构:
    Array
    Map
    Set
    String
    TypedArray
    函数的 arguments 对象
    NodeList 对象

另外，通过数组&Map对象包含的三个遍历器生成函数，`for...in` 还可以循环索引和键值对

    keys()：返回键名的遍历器。
    values()：返回键值的遍历器。
    entries()：返回所有成员的遍历器。

详细的内容大家可以通过阮一峰大大的[ES6入门](http://es6.ruanyifeng.com/#docs/iterator)了解