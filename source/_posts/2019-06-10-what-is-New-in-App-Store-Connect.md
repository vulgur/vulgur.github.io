---
title: What's New in App Store Connect - WWDC19 Session 301
tags:
  - wwdc19
  - testflight
  - app store connect
categories:
  - wwdc
date: 2019-06-10 21:41:50
---


这个 Session 只有30分钟，但是干货也不少，下面是我总结的一些要点。

{% asset_img lifecycle.jpg %}

<!-- more -->

### Upload App

* Application Loader 被替换成了新推出的 Transporter

{% asset_img transporter.jpg %}

### App Store Connect

* 邮件通知增加了许多关于审核不通过的信息
* app 中增加了 build 历史列表和每一个 build 的详细信息
* app 中增加了关于 build processing 的通知

* web 增加了一个「过去24小时」的 dashboard
* web 增加了「app 被删除」的指标

 

### TestFlight

* TestFlight Feedback，可以通过对 app 截图发送反馈，也可以在 app 崩溃的时候发送 crash report
* 在 App Store Connect 中可以查看所有的崩溃报告和截图，还可以过滤和下载等操作，每个反馈都有详细的机型和系统的信息

{% asset_img testflight.jpg %}

> 说个题外话，讲解 TestFlight 这部分的是一个特别妖娆的小哥，我很好奇以后加入中文配音的话会不会也找一个妖娆小哥来，不然违和感太强了



### App Store Localization

* 增加了许多国家和语言的本地化
* 首次支持从右至左书写的阿拉伯语和希伯来语

{% asset_img localization.jpg %}