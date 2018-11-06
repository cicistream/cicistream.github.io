---
layout: post
title: "iview Form表单验证"
subtitle: "async-validator相关及踩坑"
author: "cici"
header-style: text
tags:
  - Form
  - iview
  - async-validator
---

手头的项目有一个需求是创建自定义表单模板，深入使用了iview的Form控件，踩了不少iview的坑，同时也锻炼了validate相关的能力。

----------

## 基本知识
![Form](/img/in-post/form-validate.jpg)
Form由一些FomeItem组成，可以为Form设置rule来指定相关表单项的限制条件。
通过$ref 可以访问到 Form 组件，调用 validate 函数，即可获取相应的校验函数。iview中，**Form验证是根据FormItem的prop属性来验证**。
在编写校验函数前，首先需要了解表单验证相关知识，参看iview组件中使用的异步表单验证插件 [async-validator](https://github.com/yiminghe/async-validator)
   - validate 	`function(source, [options], callback)`
   即`function( 验证对象（必须）,  验证选项（可选）, 回调函数（必须）)`
   - rule `function(rule, value, callback, source, options)`
   - type 验证类型的限制，默认为string，全类型type如图
   ![全类型type](/img/type-all.png)

一个栗子：

```javascript
const validateNameList = (rule, value, callback) => {
    if (value.length === 0) {
        callback(new Error('请选择name'));
    } else {
        callback();
    }
};
validateAdvancedFormItem: {
  // 对于单个字段来说，通常需要多个验证规则，这些规则以数组展示
  name: [
    { required: true, message: '任务名称不能为空', trigger: 'blur' },
    { type: 'string', max: 20, message: '不能超过20个字符', trigger: 'blur' },
  ],
  nameList: [
    { required: true, type: 'array', message: '请选择nameList', trigger: 'blur' },
    { validator: validateBusiness, trigger: 'blur' },
  ]
  endTimeVal: [
    { required: true, type: 'date', message: '请选择结束时间', trigger: 'change' }
  ],
  
  noVerify:[{ required: true, validator: noVerify }], 
  // 自带的验证器不能满足要求时，需要我们自定义验证器
},
```

## 记录踩过的坑
  **1. input 默认输入为String类型**<br>
  如果在表单验证中声明`type：number`，建议input中加上number属性，将用户的输入自动转换为Number类型。<br>

  **2. select 单选多选**<br>
  **提示**： 单选返回的是一个项，而多选返回的是数组。<br>

  **3. dataPicker v-model失效**<br>
  必须on-change返回并赋值才能实现数据绑定，否则:value无法捕捉日期的而选择变动。