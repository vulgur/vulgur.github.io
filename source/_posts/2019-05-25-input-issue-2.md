---
title: Input - Issue 2
tags:
  - 日语
  - 投资
  - 经济学
  - swift
  - react
  - book
categories:
  - input
date: 2019-05-25 13:23:44
---


本来这篇 issue 的大纲都写好了差不多将近一个月了，但是最近各种杂事，代码写得不多，也疏于整理 blog 了。好在该忙的基本都忙完了，接下来的日子就是安心 I/O 了。

<!-- more -->

### Books

* **新文化日本语 初级1**。最近在多抓鱼上卖书卖上瘾了，整理书的时候发现大学时的日语教材和韩语教材一大堆，就这么直接卖了感觉对不起之前的学习，再加上一直有重新学习日语的念头，于是就打算自己再学一遍后再卖掉。一天一课，目前已看完，待总结。
* **新文化日本语 初级2**。这个初级系列一共是4本，可惜第2本丢了，于是只好在多抓鱼上买了一本，目前还差一课读完。
* **穷查理宝典**。被无数人推荐的一本书，可是我看起来感觉非常琐碎，再加上投资经历过短，实在是无法理解大神语录的精髓。
* **通向财务自由之路**。书名很吸引人，但是前提储备知识不足的我还是读起来太困难了，目前只读完了前两章。
* **薛兆丰经济学讲义**。评价很高的一本书，也是在 得到 里知道了这个作者和这本书。每一章里有若干讲，每一讲都对应一个案例和理论。目前读完了前三章，30讲，感觉这30讲读下来在脑中还是一个个被割裂的片段，所以目前一边整理脑图一边读。

{% asset_img xzf.png %}

### Tutorials

* [UITableView Infinite Scrolling Tutorial](https://www.raywenderlich.com/5786-uitableview-infinite-scrolling-tutorial)，原来 iOS 10 新加入了`UITableViewDataSourcePrefetching` 这个协议专门用来预加载，我之前一直是靠判断 scroll view 的滚动距离来实现的……

### Blogs & Articles

* [The power of UserDefaults in Swift](https://www.swiftbysundell.com/posts/the-power-of-userdefaults-in-swift)，本以为在存储上有什么高级用法，结果并不是。但是还是学到了两种用途：一是在 app group 中分享数据，二是修改启动参数。

* [Launch arguments in Swift](https://www.swiftbysundell.com/posts/launch-arguments-in-swift)，这个是👆那篇引用的 blog，对于启动参数的有效使用对 debug 和 test 都非常有用。

* [Feature flags in Swift](https://www.swiftbysundell.com/posts/feature-flags-in-swift)，这个又是👆那篇引用的 blog，Sundell 的 blog 就是这样，总会引用自己之前相关的 blog，像维基百科一样，一篇篇互相连接起来，读起来就没完没了。这篇的内容就是如何设置静态和动态的 feature flags。

* [Using @autoclosure when designing Swift APIs](https://www.swiftbysundell.com/posts/using-autoclosure-when-designing-swift-apis)，还是 Sundell 的一篇，讲的就是如何用关键字 `@autoclosure` 来简化 API 的调用。

* [每周 Swift 社区问答：@autoclosure](https://swift.gg/2016/04/06/swift-qa-2016-04-06/)，和👆一起服用的，关于 `@autoclosure` 的更详细的介绍。

  

### Tools

* [Network Link Conditioner](https://nshipster.com/network-link-conditioner/)，苹果官方有一个开发工具包 Additional Tools for Xcode，其中的 Network Link Conditioner 是用来模拟各种网络环境的，比如带宽、延迟和丢包率。当然还有许多其他方面的工具，比如图形、音频和 HomeKit 等等等等。
* [Moya](https://github.com/Moya/Moya)，用 Swift 实现的网络抽象层，久闻大名但是一直没机会使用。这次趁着重构 app 网络层的机会就尝试了一下 Moya，感觉和之前自己写的分层区别不大，都主要是对 API 中 endpoint 的封装。
* [emoji-flags](https://www.npmjs.com/package/emoji-flags)，上一篇 Input 中提到的那个 react-flag-icon-css 被替换成了这个。
* [react-infinite-scroller](https://www.npmjs.com/package/react-infinite-scroller)，无限滚动加载，简单好用。
