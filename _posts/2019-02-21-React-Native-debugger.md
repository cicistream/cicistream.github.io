---
layout: post
title: "React-Native调试大法"
subtitle: "React-Native的调试与web端的‘直接接触’非常不同"
date:  2019-02-21
author: "cici"
header-img: "img/post-bg-js-module.jpg"
tags:
  - React-Native
  - debug
  - Mobile
---

react-native巨坑，相关插件也巨坑，版本更新慢，维护不及时。
我手头使用的是RN目前最新版本0.57.8，以下均为此版本环境下遇到的问题，在解决问题时配合开发调试方法，事半功倍。

----------


我使用的技术栈是：react-native（0.57.8）+ [react-navigation](https://reactnavigation.org/zh-Hans/) + react-redux + ant-design + axios

react-native调试不友好，不像web端可以直接定位到具体组件，还可以选择相应的代码块。
<br>
主要调试工具可以配合使用Android Studio，Chrome，终端，真机提示等，我用的IDE是VS Code，一般使用Developer Menu中的debug in remotely，根据打印在Chrome控制台的信息进行开发。
<br>
这篇详细的关于Developer Menu的文章，比官网描述详细，值得一看
[React Native调试技巧与心得
](https://www.jianshu.com/p/30e1d7d99a9e)<br>

### 补充几点：

1. 在虚拟机上进行调试可以通过双击R键进行reload<br>

2. 调试时只能有且只有**一个**机器（虚拟机或真机）连接可以使用`adb devices `来查看当前连接设备。<br>
在Android5.0以上设备上，将手机通过usb连接到你的电脑，然后通过adb命令行工具运行如下命令来设置端口转发
`adb reverse tcp:8081 tcp:8081`<br>
而且，除了摇晃手机，还可以使用`adb shell input keyevent 82`来调出Developer Menu<br>
如果你的终端无法识别adb命令，应在终端或者iTerm界面运行如下命令：<br>
`open ~/.bash_profile`<br>
这样就开了配置文件，然后在zshrc文件里面添加如下配置：<br>
`export ANDROID_HOME=/Users/*****/Library/Android/sdk（后面配置的是你系统存放SDK的目录）`<br>
然后使用下面的命令保存并应用文件<br>
`source ~/.bash_profile（这个表示默认把系统的配置文件拿过来了）`<br>
然后就OK了，小伙伴们可以愉快的玩耍了<br>

3. 有时候**Debug JS Remotely**无法连接，此时可以先检查Chrome中打开了几个调试页面，只保留其中一个并检查网址是否是` http://localhost:8081/debugger-ui` <br>
因为有时候**localhost**会被替换成ip地址而无法连接，如果还不能连接，保证该网页开启的情况下，再重新跑`react-native run-android/ios`命令<br>

4. 如果你想在Chrome的network面板看到应用网络信息，可以在**react-native\Libraries\Core\InitializeCore.js** -> 搜索 XMLHttpRequest，注释掉 `polyfillGlobal('XMLHttpRequest', () => require('XMLHttpRequest'));`
再在真机或模拟器reload  查看Network面板，此时就可以查看到我们的fetch请求了<br>
需要注意的是，当我连接公司内网并需要在请求中加入token的时候，注释掉这行代码会导致错误。

 