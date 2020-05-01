---
title: 用 Reveal 分析系统日历应用（碰到了一个大坑）
date: 2016-04-14
tags:
- reveal
- reverse
categories:
- iOS
---

这篇博客的起因是这样的，前些日子的一个电话面试，其中一个问题是：如何实现 iOS 系统中的日历应用。
<!-- more -->
{% asset_img calendar.png Calendar %}

我一边翻看手机中的 app，一边回答说有两种做法：一是用 UITableView+Cell，一是用 UIScrollView+UIView，在滑动的过程中计算即将显示的内容。

面试的结果是失败，当然不光是因为我上面的回答，但却激起了我的好奇心，这日历到底是怎么实现的？

于是我就打开了 Reveal，这个神器是我在初学 iOS 时就果断买的，当时还是折扣价，现在想想都觉得超值。如果没有这个东西我都无法想想我的第一个项目会遇到多大的困难，可能都无法完成交付。

## 用 Reveal 查看日历应用

关于使用 Reveal 查看任意 app 的方法，我就不再这里赘述了，网上有很多教程，在此给出一个搜索关键字：**Reveal Loader**。

以前用 Reveal 看其他 app 的 UI 是如何实现的时候，一看一个准，从未遇到任何问题。但是这次让我碰到了一个大坑……其他的 app 都可以用 Reveal，唯独这个日历 app 不可以（后来经过搜索，发现相机应用也是不可以的），Reveal 就是死活连不上，真是奇了怪了。

鼓捣了半天后还是无法解决，只好 Google 大法了。一开始用中文关键字搜索，都是不着边际的搜索结果。改用了英文关键字，第一个结果就是我想要的！

搜到的结果网页是 Reveal 的一个 Support 页面，[Unable to inspect Calendar (MobileCal)](http://support.revealapp.com/discussions/questions/50918-unable-to-inspect-calendar-mobilecal) 。这里面说貌似日历应用阻止了和 Reveal HTTP Server 的连接，而在倒数第二楼里面也给出了 SO 的一个链接，[Cannot bind() a socket inside Apple Calendar (dylib injection)](http://stackoverflow.com/questions/27126315/cannot-bind-a-socket-inside-apple-calendar-dylib-injection)，里面就是更详细的原因和解决方法。

对于沙盒机制和 dylib 注入什么的我基本不懂，我就拣我看得懂的说。原因就是，日历应用（MobileCal）采用的是自定义沙盒，app 中有一个 Entitlements.xml 文件，里面就是规定这个 app 的各种权限。这个文件里又有一个特殊的字段（seatbelt-profiles），导致阻止了所以得网络通信。回答者也给出了两个解决方案，而第一个难度系数太高，所以就第二个给出了实际的操作方法。

简单来说，这个方法就是把 app 的权限导出，编辑后再塞回给 app。为什么不直接修改这个 Entitlements.xml 呢？因为真正的权限存在于 app 的二进制文件中。整个操作所使用的工具是 **ldid**。操作的过程中我遇到的一个问题就是怎么在手机中找到 app 的文件位置，一开始我走了弯路，我误以为要拿到 ipa 文件，其实不是。网上搜到的都是说在 **/var/mobile/Applications/** 下面，还说所有的 app 都是以一些乱七八糟的标识符命名的文件夹，但是我的 4s 并没有这个目录。还是搜了半天才发现，真正的 app 都在/Applications/（好像是从 iOS 8.x 开始改的），而且所以得 app 目录就是以 **XXX.app** 命名的。

{% asset_img applications_list.png App Folders %}

接下来的就十分简单了，就是照着 SO 上面执行以下步骤：

1. 导出权限，`ldid -e MobileCal > entitlements.xml`
2. 手机上还没有 vi 和 vim，只好把这个 xml 复制到电脑中编辑，删掉那个 **seatbelt-profiles**，再传回手机

{% asset_img entitlements.png Entitlements.xml %}

3. 最后把权限导入，`ldid -Sentitlements.xml MobileCal`

这样就大功告成了，日历就可以在 Reveal 中一览无遗了。

{% asset_img reveal.png Calendar App %}

## 分析日历应用的实现

我主要看了月视图和年视图这两个页面的结构，发现这个应用的实现真是非主流啊。

{% asset_img month.png Month%}
{% asset_img year.png Year%}

月视图的页面主要是由 `CompactMonthWeekView` 和 `CompactMonthWeekDayOverlay` 组成的，这两个东西都是 `UIView` 的子类。诡异的是月视图中表示日期的1到31都看不到 `UILabel`，这些数字都是包含在 `CompactMonthWeekView` 中。而那些 `CompactMonthWeekDayOverlay` 也只是显示农历日期，看不到有任何的 subView。想不出这两个东西是如何封装的。当然，表示「今日」的控件是另外的一个类，可以看到其中的 `UILabel`。我把手机切换到英文，再看，果然 `CompactMonthWeekDayOverlay` 这个东西就没了。

{% asset_img month_en Calendar in English %}

再来看年视图，主要是由 `CompactYearMultipleMonthView` 和 `CompactYearMonthView` 组成。同样的是，数字都看不到 UILabel 的存在，也看不到 UIImageView，实在是奇怪。

还遇到一个奇怪的地方，4s 的月视图中点击日期除了「今天」都没有反应，而 iPhone 6 点击任意日期都可以进入日程的页面。

年视图和月视图可以无限上滑和下滑，在 Reveal 中可以看到对上一年（月）和下一年（月）都是有预加载的。年视图中不是整年的加载，大概是两行左右的预加载（在屏幕外）。月视图中下个月是整月加载，本月和上个月加载的不多，只有两三个星期。

这些控件应该不是固定的图片，理论上是可以通过算法计算出每个月视图和年视图中数字的排列的，但是在无限滑动中还能保持特别的顺畅，实在是费解。我只会用 Reveal 查看 app 的 UI 结构，对于逆向领域是一窍不通，如果有哪位有过研究的前辈还请告知，谢谢！
