---
title: Great Developer Habits - WWDC19 Session 239
tags:
  - wwdc19
  - xcode
  - craft
categories:
  - wwdc
date: 2019-06-13 23:31:15
---


看这个 Session 的标题本以为是讲一些宽泛的通用的方法论，其实还是围绕苹果的开发环境来提出一些 best practice，当然部分内容也可以适用于其他开发的领域。

{% asset_img 239_cover.jpg %}

<!-- more -->

值得一提的是，这个 session 是我目前看过的所有 WWDC19 的 session 中最舒服的一个了。从头至尾只有一个演讲者，中间没有 demo 演示，单枪匹马说了半个小时。虽然也是不断看提词器，但是没有奇奇怪怪的口音，吐字清晰，语气、语调和节奏都堪称完美，举止也很优雅得体（没有其他工程师那种你懂的形象）。

整个 Session 从七个部分来给出开发中的各个需要注意和提高的细节，分别是：Organize、Track、Document、Test、Analyze、Evaluate、Decouple 和 Manage。

{% asset_img 239_habits.jpg %}

### Organize

这一部分讲的都是 Xcode 中对于项目的一些 best practice。

{% asset_img 239_organize.jpg %}

* 根据功能划分使用文件的 group
* 精简项目的结构和文件的结构
* 将庞大的 storyboard 分解成一个个小的 storyboard
* 采用最新的构建系统
* 扔掉无用的代码（因为有 source control）
* 解决最根源的 warnings

### Track

这一部分讲的是如何利用 source control 来管理代码。

{% asset_img 239_track.jpg %}

* 使用源码管理系统
* 尽量保持 commit 既小又独立
* 好好写 commit 的信息
* 利用分支来管理 bug 和 feature

### Document

这一部分讲的是文档和注释。

演讲者说「自解释」的代码是不能说服人的，因为这些代码只能说明做了什么，不能说明做的原因。

{% asset_img 239_document.jpg %}

* 注释对于在未来对代码的理解至关重要
* 好的注释可以提供代码的背景和原因（Xcode 11 可以方便地给方法自动生成注释 placeholder 了）
* 使用描述性的变量名和常量名（除非特殊场合，不要使用单字母名字）
* 写文档

### Test

这一部分讲的是单元测试。

{% asset_img 239_test.jpg %}

* 一定要写单元测试
* 在提交代码之前一定要先跑单元测试
* 建立持续集成

### Analyze

这一部分讲的是如何使用 Xcode 来模拟网络环境、调试和分析 app 的各项性能指标。

{% asset_img 239_analyze.jpg %}

* 用 Network Link Conditioner 模拟差的网络环境（Xcode 11 中已经集成这个功能了）
* 使用 sanitizers 和 checkers 来进行内存调试
* 用 Debug Gauges 进行性能和效率的测量
* 用 Instruments 来调查各种问题

### Evaluate

这一部分讲的是代码评估。

* 将 code review 作为实践的一部分
* 理解每一句代码
* 构建代码
* 跑测试
* 针对代码风格、拼写和语法进行校对

### Decouple

这一部分讲的是如何不要重复造轮子以及如何分享代码。

{% asset_img 239_decouple.jpg %}

* 将业务代码分离出来
* 通过 package 在不同的 app 中共享代码
* 通过扩展来提高代码效率
* 在社区分享你的作品
* 文档至关重要

### Manage

最后这个部分讲的是如何管理代码的依赖。

{% asset_img 239_manage.jpg %}

* 以认真负责的态度使用开源社区和开源项目
* 彻底理解所有的依赖
* 尊重隐私
* 准备一个依赖消失或者不再维护的 B 计划