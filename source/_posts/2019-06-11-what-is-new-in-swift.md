---
title: What's New in Swift - WWDC19 Session 402
tags:
  - swift5.1
  - api
  - wwdc19
categories:
  - wwdc
date: 2019-06-11 11:00:14
---




首先，Xcode 10.2 已经支持了 Swift 5，而 Xcode 11 会支持 Swift 5.1，其实 SwiftUI 的背后就有 Swift 5.1 的新 API。

关于 ABI 和 SPM 就不再赘述了，下面是 Session 中提到的一些 Swift 5.1 中的新内容和改进。

<!-- more -->

### Performance

{% asset_img swift_performance.jpg %}

* 代码大小缩小了 10%
* 如果开启了 Optimized for Size，代码大小会缩小15%
* `NSDictionary` 到 `Dictionary` 的桥接提高了1.6倍
* `NSString` 到 `String` 的桥接提高了15倍
* `String` 的默认编码由 UTF-16 改为了 UTF-8
* 基于 SwiftNIO 的 HTTP RPS 提高了 20%



### Swift Tooling and Open Source

{% asset_img swift_docker.jpg %}

* 提供了官方的 Swift Docker Image
* SourceKit，一个提供代码编译、跳转到定义和重构的工具集
* Language Server Protocol，配合 SourceKit 可以让任何遵守这个协议的编辑器都能编辑 Swift，比如 Vim



### Language and Standard Library

{% asset_img swift_se.jpg %}

* 增加了 implicit return from single expressions
* 增加了对成员变量的 synthesized default values
* 改进了针对向量运算的 API：SIMD
* String Interpolation 的速度提升了，而且可以增加 interpolation 的行为了，比如本地化
* 增加了一个关键字 `some`，用于修饰 Opaque Result Types，具体使用方法请看这篇 [泛型语法改进第一弹 —— Opaque Result Types](https://kemchenj.github.io/2019-05-05/)
* Property Wrapper Types，就是把具有相同逻辑的property 的读写包装起来，比如 UserDefaults 的操作，这样就可以减少许多冗余代码
* 可以用 Swift 定义 DSL 了，SwiftUI 的语法其实就是一种 DSL

