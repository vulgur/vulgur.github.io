---
title: What's New in Xcode 11 - WWDC19 Session 401
tags:
  - ide
  - xcode
  - wwdc19
categories:
  - wwdc
date: 2019-06-10 10:32:45
---


下面是 Session 401 提到的 Xcode 11 中的新功能和改进，我列出来大概其中90%的内容。

<!-- more -->

### Editors

* 增加了一个 Editor Option 的按钮，把原来的 Assistant Editor 整合在其中，还有新的 Canvas 和 Minimap 等功能
* 增加了一个分割 Editor 的按钮，可以横向也可以纵向添加 Editor，每个 Editor 都可以放大到占满整个 Xcode，也可以恢复到分割后的大小
* Editor 的分割是等分的，即使分割后手动调整了大小，zoom in 再 zoom out 后，还是会恢复成等分的大小，不知道这是不是一个 bug

{% asset_img xcode11.jpg %}

### Minimap

* Minimap 里的颜色和语法着色是一致的
* 鼠标 hover 在 Minimap 中会显示对应的属性和方法
* 搜索文件中的内容，会在 Minimap 中突出显示搜索的字符串
* Minimap 会显示代码中的 MARK 注释，这一点超级棒，尤其是对于超大型的 View Controller

### Code Editing

* 可以根据方法签名生成方法注释
* 可以统一修改方法（包括注释）中的参数名
* 可以在同一个 Editor 中显示代码的 diff

### Swift Package Manager

* Swift Package Manager 被整合进了 Xcode 11，一条龙服务 
* 可以只搜索 Swift Package 中的代码内容

### Source Control

* 可以 stash 和 cherry-pick 了
* 可以在 inspector 中看 git history 了

### Design Tools

* Size Class 中的 Device 增加了 mac
* Assets 可以添加自定义的 Symbol
* Assets 可以本地化了
* Inspector 选择 Image 可以看到缩略图了，不仅仅是图片名
* Image 可以为 dark mode 定制一个和 light mode 不一样的内容
* 新增了一个 Environment Overrides 的选项，用来改变模拟器的外观属性

### Debugging

* Device Condition 可以在 Xcode 中指定了，比如网络，不需要以前的 additional tools 了

### Simulator

* Apple Watch 的模拟器可以独立运行了
* Metal 可以运行在模拟器里了
* 绘制、CPU 使用、启动都得到了大幅提升



