---
title: Catalina beta 初体验
tags:
  - macos
  - bug
categories:
  - catalina
date: 2019-06-12 20:21:09
---


这是我第一次装 beta 版的 macOS，不在主力机上使用 beta OS 是我之前一直的原则。

其实这一次一开始我也没有打算动用主力机，我只是下载了 Xcode 11 beta 在主力机上，但是 SwiftUI 的 Canvas Preview   需要 Catalina beta 才可以。于是我把备用机（MD101）装上了 Catalina beta，但是我几乎把备用机的资料和 app 删得只剩一个系统了，还是没有足够的空间来安装 Xcode 11 beta。

无奈 SwiftUI 的诱惑力太大了，我就突破了自己的底线在主力机上装了 Catalina beta，于是我就开始了为期一周的体验，但是有些 bug 实在忍不了，再加上整个系统的卡顿现象十分严重，只好今天抹盘重装了。

因为资料都备份好了，我也不想用 Time Machine 恢复了，于是我现在用的是一个干干净净、速度飞快的系统了，MD 以后再也不折腾 beta OS 了。

下面就是我这一周内遇到的 Catalina 的一些问题，还有少量的惊喜。

<!-- more -->

{% asset_img catalina.jpg %}

### Changes

* Safari 字体变细了

* 右键点击 Safari tab 弹出的菜单中多了两个选项

  {% asset_img safari_tab.jpg %}

* 外观可以设为「自动」了，也就是自动切换 light mode 和 dark mode，night owl 可以卸载了

### Bugs

* Finder 的智能文件夹对于部分文件扩展名的搜索无效
* Dock 中有的 app 的图标在副屏上显示为空白（滴答清单）
* 网易有道词典无法打开
* 保存文件选择文件夹时卡到爆
* Safari 所有之前的扩展都不见了
* Safari 会偶尔超级超级卡，只能暂时用 Chrome 作为默认浏览器了
* Finder 也会偶尔卡死
* Podcast 里面「浏览」和「排行榜」的节目都是国外的
* 点击微信的非自带表情， 无效
* 开启 Popclip 会出现80%的点击拖放后会弹出一个「Learn」的菜单
* 音乐（原 iTunes）有时会无法播放歌曲，完全退出再打开就好了
* Ctrl + 左右方向键 切换桌面有时没有动画，黑屏一下就切换过去了
* Xcode 11 beta 各种崩溃退出（常规操作啦）
* Numbers 打开后崩溃
* Alfred 4 针对 App 的搜索有许多问题，要么重复，要么搜不到
* 弹出移动硬盘会导致五国

### Surprises

* 在文件夹点击右键弹出的菜单底部多了三个选项，Go2Shell 可以卸载了。(但不是每次都出现，目前还不知道一定出现的条件)

  {% asset_img folder_options.jpg %}

* 独立的 Podcast app 目前还没有出现跳集的现象。

  

  