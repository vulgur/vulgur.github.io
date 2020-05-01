---
title: Input - Issue 1
date: 2019-05-02 11:40:47
tags:
- react
- iOS
- mongodb
- node
- tools
- libraries
categories:
- input

---

如果用计算机程序来类比人生的话，那么一个人的生活从最简单的角度去看，无非就是不断的输入和输出。

输入就是人的五感带来的信息，即所见所闻。输出就是由我们产生的东西，即所想所为，可具体可抽象，具体如言语、绘画、产品等，抽象如各种思考的衍生物。

<!-- more -->

 只谈编程开发和技术学习的方面，我的输入不算少。我订阅了许多 newsletter 的 email，也有不少 blog 的 RSS，再加上平时搜索得到的 blog 和文章，其实在这方面的阅读量并不少，虽然不一定每周都及时读完每个文章，但也基本会抽出一段时间集中读完。

但是，我一直有以下几点的困扰：

1. 阅后即忘。小到一篇 blog，大到一整本，过了一段时间以后我都有可能完全忘掉。这也就导致了重复搜索和学习，时间和精力被大大消耗了。
2. 有时候的阅读只是肤浅的读了一遍，并没有完全理解或者没有上手实践，导致印象不深，和第一点很像。
3. 脑中始终是混乱的，熵增现象越来越明显。

学习的方法论里称最好的学习是教会其他人，这其实就是完全的输出了，然而我能力有限，我能自己学会了就已经不错了。我退一步，先做到总结，也算是简单模式的输入了。

所以，我打算不定期（希望能做到定期）记录自己一段时间内的输入，并附上简单的总结，这个系列就是**Input Issue**。我并不是要发布一个 newsletter，一来做不到定期，二来做不到一手性。只是我个人简单的记录，也方便以后的查找。



### Problems

- [解决 iOS View Controller Push/Pop 时的黑影](https://imtx.me/archives/1933.html)，最近在写的一个 app 也遇到了这个问题，而这个问题我之前也遇到过，但是完全忘记了。最后的解决方案也不是图拉鼎写的那个，而是文章评论里的。

### Tutorials

- [React Tutorial](https://reactjs.org/tutorial/tutorial.html)，这个放在这可能有充字数的嫌疑……但我真的是又跟着上手写了一遍（是「又」噢）。正好碰到实现一个非常简单的 web demo 的机会，于是就想着用 React 了。 也算是技能树又多了一项，可以多接活了！
- [MongoDB Quick Start Node.js Edition](http://mongodb.github.io/node-mongodb-native/3.1/quick-start/quick-start/)，还是和上面提到的那个 demo 有关，不想用 MySQL 了，换个味道。
- [Material-UI](https://material-ui.com/)，依旧是和上面提到的 demo 有关，自己写 CSS 是不可能的，不过还没有完整看完，只是看了需要的几个控件。和 Element UI 相比，要额外写好多的 import。

- [UISearchController Tutorial: Getting Started](https://www.raywenderlich.com/472-uisearchcontroller-tutorial-getting-started)，以前我都是自己用 UITextField 和 UITableView 来实现搜索栏，都不知道 iOS 早就有整套的 solution 了。



### Blogs & Articles

- [Designing Swift APIs](https://www.swiftbysundell.com/posts/designing-swift-apis) ，这篇 blog 里的 API 是广义上的，即函数签名。文章重点就是如何用 Swift 语法中的一些特性来使得函数签名既简单又明确，还可以达到灵活化，这些特性包括参数标签（argument label）、参数默认值、参数类型严格化以及包装器（wrapper）的使用。

下面这三篇是一口气看的，由表入里。

- [This is why we need to bind event handlers in Class Components in React](https://medium.freecodecamp.org/this-is-why-we-need-to-bind-event-handlers-in-class-components-in-react-f7ea1a6f93eb)，初学 React，这是遇到的第一个坑。
- [Losing .bind(this) in React](https://medium.com/@nikolalsvk/loosing-bind-this-in-react-8637ebf372cf)，如何去掉丑陋的`bind(this)`，其实就是用 arrow function，不过里面提到了 arrow function 的一些缺点以及 ES6 中 class 的内存分配。
- [Arrow Functions in Class Properties Might Not Be As Great As We Think](https://medium.com/@charpeni/arrow-functions-in-class-properties-might-not-be-as-great-as-we-think-3b3551c440b1)，这篇解释了 arrow function 被 babel 转译后的结果，还有 arrow function 在类继承上的一些坑，最后还给出了 arrow function、bound function 和普通 function 的性能对比。

### Libraries & Tools

- [SwiftCharts](https://github.com/i-schuetz/SwiftCharts)，app 中有折线图的需求，于是就用了这个 swift 中 start 最多的一个 library。没想到啊，巨难用！搞了两天才用自己的数据跑通并正确显示出来，虽然支持许多样式的图表，但是并不支持我设计的那种，而且定制起来太麻烦了，需要大量的魔改，准备换一个方案了。
- [Tabman](https://github.com/uias/Tabman)，也是许多 star，但是我已经弃用了。原因一是这个控件适合全屏都作为一个 tab 的情况，如果只是页面中的部分作为 tab 容器的话需要魔改，我这人懒，喜欢开箱即用的东西。还有一个原因是每个 tab 下的 child view controller 的大小不能控制，也和第一点有关，在我的 app 里出现了 frame 大小诡异的 bug。
- [csv-parser](https://www.npmjs.com/package/csv-parser)，之前的一个项目需要将 MySQL 中的旧数据导出，再转换成新格式的数据导入到新表中，于是就搜到了这个 package，就是将 csv 转换成 JSON。
- [country-list](https://www.npmjs.com/package/country-list)，demo 项目需要用到 country code。
- [react-flag-icon-css](https://www.npmjs.com/package/react-flag-icon-css)，根据 country code 来提供对应的国旗图标，和上面的 country-list 配合使用的。
- [request-ip](https://www.npmjs.com/package/request-ip)，获取请求的 IP 地址。
- [geoip-country](https://www.npmjs.com/package/geoip-country)，获取 IP 地址所在的国家，配合上面的 request-ip 使用的。
- [random-ipv4](https://www.npmjs.com/package/random-ipv4)，生成随机 IPv4 地址，和上面的 geoip-country 一起用来做测试的。
- [mongoose](https://www.npmjs.com/package/mongoose)，MongoDB 的 ORM，不多说了
- [react-router]([https://github.com/ReactTraining/react-router)，React 没有官方的 Routing library，于是就选了这个貌似是使用的最多的，我的 demo 只有两个页面，所以一些复杂的路由配置还没用上。

