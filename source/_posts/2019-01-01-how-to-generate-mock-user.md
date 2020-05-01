---
title: 如何生成模拟的用户数据
tags:
  - test
  - mock
  - package
categories:
  - Node
date: 2019-01-07 00:00:00
---


这是迟到了半年的一篇 blog，本来去年8月就想写了，可是当时工期比较紧，一直忙于赶工，等到做完了就拖延到了今天，那就作为2019年第一篇正式 blog 吧。

相关项目是去年做的一个社交类型的小程序，我负责的是后端。到了测试用户列表的时候，必须要生成一些模拟用户。像年龄、性别、标签、描述这样的数据还比较好模拟，就是预设一些合理的数据集合，然后在里面随机取值就行了。还有些属性就不能这么简简单单地模拟了，一些是构造数据比较麻烦（或者是浪费脑子，比如名字和日期），还有一些就是人工无法模拟出相对准确的数据（比如地理坐标），还有一些就是为了让测试数据看起来不那么假（比如头像）。

<!-- more -->

下面分别介绍针对名字、日期、地理位置和头像这四个用户属性使用的工具和方法。像这样随机生成模拟数据的包非常多，搜索关键字 `mock`、 `random`、 `fake` 即可。

### 模拟名字

我觉得可能有许多人（包括我）在生成测试数据的时候喜欢用`111` ，`12345` 和 `test` 这样的字符串，或者干脆胡乱敲一顿键盘，比如`asdfg` 或`oizxuvc` 这样的。这样的胡乱输入，第一看起来很假，第二因为字符很少会导致一些 UI 上的问题无法被发现。

我用的是[node-random-name](https://www.npmjs.com/package/node-random-name)，用起来非常简单，下面是官方的代码示例，一共才5行。

```javascript
var random_name = require('node-random-name');
console.log(random_name()); // -> "Brittny Kraska"
console.log(random_name({ first: true, gender: "male" })); // -> "Jean"
console.log(random_name({ last: true })); // -> "Kinsel"
console.log(random_name({ seed: 'Based on this' })); // -> "Garrett Scheets"
console.log(random_name({ random: Math.random, female: true })); // -> "Jo"
```

我用的这个包只能生成英文名，也有许多其他的包能生成其他语言的名字，还有一些各种动物、头衔、物品的名字。在 npm 里搜索就像开宝箱一样，总有一些让人惊喜的东西。

### 模拟日期

日期这个属性用在用户上主要就是生日了，还有一些操作的元数据。`moment` 是众所周知的处理日期的利器，而我选用的随机日期生成器就是基于`moment`的一个包，[moment-random](https://www.npmjs.com/package/moment-random)。

这个包也是非常简单，只有一个 API，默认返回的是过去的某个日期，当然也可以指定日期的范围。

```javascript
const momentRandom = require('moment-random');
 
momentRandom();
// > random date in the past
```



### 模拟地理位置

项目中需要根据用户的定位数据来显示位置信息和互相之间的距离，所以需要模拟一些坐标数据。我采用的方法是根据某个城市中心的经纬度，然后随机生成一定范围内的坐标。一开始我发现了[random-coordinates](https://github.com/mock-end/random-coordinates) ，但是这个包生成的数据选项中只有精度，而没有范围，最后选用了 [random-latitude](https://github.com/mock-end/random-latitude)和 [random-longitude](https://github.com/mock-end/random-longitude)这两个包，其实都是同一个作者的作品。拿北京的坐标举例，下面的代码就是生成了周边的坐标信息。

```javascript
const latitude = randomLatitude({
    min: 39.9,
    max: 40.07,
    fixed: 10
})
const longitude = randomLongitude({
    min: 116.3,
    max: 116.39,
    fixed: 10
})
```

### 模拟头像

前面的几个工具都是一句调用的事，生成的说到底就是数字或字符串。而头像这个东西就复杂一些了，生成的是图片文件。在进行了几个工具的对比后，我最终选择的是[avatar-generator](https://www.npmjs.com/package/avatar-generator/v/2.0.2)这个包，它能生成 8bit 风格的头像，而且还能够根据人物的性别以及指定的一些外观来生成头像（最新的2.x版本才支持的）。

{% asset_img avatar.png generated avatar%}

去年我用的时候这个包的版本还是1.0.8，而且还必须先安装 ImageMagick 才可以正常使用。现在升级到了2.0.x，负责图像处理的内部工具也改成了 sharp，也不必额外安装其他依赖了。旧代码我就不贴了，有兴趣的直接去官网看吧。





