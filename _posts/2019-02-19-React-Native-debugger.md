---
layout: post
title: "React-Native填坑笔记"
subtitle: "React-Native中令人摸不着头脑的坑"
date:  2019-02-19
author: "cici"
header-img: "img/home-bg-art.jpg"
tags:
  - React-Native
  - debug
  - Mobile
---

react-native巨坑，相关插件也巨坑，版本更新慢，维护不及时。
我手头使用的是RN目前最新版本0.57.8，以下均为此版本环境下遇到的问题，在解决问题时配合开发调试方法，事半功倍。

----------


我使用的技术栈是：react-native（0.57.8）+ [react-navigation](https://reactnavigation.org/zh-Hans/) + react-redux + ant-design + axios

项目中使用的插件有
- [react-native-splash-screen](https://github.com/crazycodeboy/react-native-splash-screen)
- [react-native-camera](https://github.com/react-native-community/react-native-camera)
- [react-native-image-picker](https://github.com/react-native-community/react-native-image-picker)
- [react-native-local-barcode-recognizer](https://github.com/januslo/react-native-local-barcode-recognizer)

安装插件切记：react-native link *

下面是几个开发中阶段遇到的问题及解决方法。
原生问题可以从[rn论坛](https://bbs.reactnative.cn/category/4/%E6%B1%82%E5%8A%A9%E4%B8%93%E5%8C%BA)找,插件问题从GitHub上的issue找

### 1.初始化
1. 初始化运行失败，一般是环境问题，还有可能是RN版本问题
```javascript
A problem occurred configuring project ':app'.
SDK location not found. Define location with sdk.dir in the local.properties file or with an ANDROID_HOME environment
```
![图片1](/img/in-post/post-rn-debug/prob1.png)


2. A problem occurred configuring project ':app'.
SDK location not found. Define location with sdk.dir in the local.properties file or with an ANDROID_HOME environment
![图片2](/img/in-post/post-rn-debug/prob2.png)

### 2.开发过程
1. **绝对定位元素被键盘顶起**<br>
打开android工程，在AndroidManifest.xml中配置如下：
```javascript
 android:windowSoftInputMode=“stateAlwaysHidden|adjustPan”
```
2. **StatusBar 主题污染**
```javascript
componentDidMount() {
    this._navListener = this.props.navigation.addListener('didFocus', () => {
      StatusBar.setBarStyle('dark-content');
    });
  }
  componentWillUnmount() {
    this._navListener.remove();
  }
```
3. **React Native Text截断**<br> 显示数字加粗后最后一位无法显示问题，添加
```javascript
fontFamily: 'System'
```
4. **使用Button进行路由跳转时报错**<br>
这个错误一般只在开启`Debug JS Remotely`时出现，可以通过使用TouchableOpacity包裹或替换解决

5. **使用react-native-camera插件的页面二次进入黑屏**<br>
通过focusedScreen的true/false来显示RNcamera组件
```javascript
componentDidMount() {
    const { navigation } = this.props;
    navigation.addListener('willFocus', () =>
      this.setState({ focusedScreen: true })
    );
    navigation.addListener('willBlur', () =>
      this.setState({ focusedScreen: false })
    );
  }
```

6. **React-Native的样式与web有所不同**<br>
具体的样式指南参看[React-Native 样式指南](https://github.com/doyoe/react-native-stylesheet-guide#react-native-%E6%A0%B7%E5%BC%8F%E6%8C%87%E5%8D%97)

7. to be continue