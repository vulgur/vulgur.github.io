---
title: input issue_3
draft: true
tags:
- podcast
categories:
- issue
---



### Problems & Solutions

* 升级到了 Catalina beta 后用 Xcode 10 打包的 app 审核被拒，理由是「二进制文件无效」，具体信息如下：

  > **ITMS-90111: Invalid Toolchain** - Your app was built with a beta version of Xcode or SDK. Apps submitted to the App Store must be built with the GM version of Xcode 9 and the SDK for iOS 11, tvOS 11, watchOS 4, or macOS 10.13 or later.

  解决方法：修改 app archive 中的 info.plist，具体操作请移步 [Solution](https://stackoverflow.com/questions/37823382/can-i-upload-xcode-builds-on-macos-10-12)
  
  
  
  
  
* Previous Content

  https://discussions.apple.com/thread/7755547

  

* iOS 证书已过期

* 重装 Mojave，提示

  >  安装需要下载重要内容。该内容此时无法下载。请稍后再试。


### Podcast

* 「某 C 的微信入职经验分享」 by **ggtalk**
* 「30岁财务自由，可能比你想的简单」by **WEB VIEW**，

### Blogs & Articles

* [Inside SwiftUI's Declarative Syntax's Compiler Magic](https://swiftrocks.com/inside-swiftui-compiler-magic.html)，主要围绕 Return-less single expressions、Property Delegates 和 Function Builders 这三个方面分析了 SwiftUI 背后实现的黑魔法，其实是使用了一些还没正式发布的 Swift 特性。

### Tools & Libraries

* [tiptap](https://github.com/scrumpy/tiptap), Vue.js 的富文本编辑器又有新选择了。
* [Combine](https://developer.apple.com/documentation/combine)，iOS 13 新出的一个 framework，针对数据处理提供一套声明式的 Swift API，主要由 Publisher 和 Subscriber 组成，和 ReactiveCocoa 类似的概念。

