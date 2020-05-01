---
title: What's new in iOS Design - WWDC19 Session 808
tags:
  - wwdc19
  - design
  - dark mode
categories:
  - wwdc
date: 2019-06-20 16:02:30
---


整个 Session 讲了 iOS 13 中新引入的三个设计上的改进，dark mode、modal presentations 和 contextual menus。

### Dark Mode

这一部分介绍了设计规范、颜色、材质以及控件等方面在 dark mode 下的具体实现。

{% asset_img dark_mode.jpeg %}

<!-- more -->

#### iOS Design System

首先，所有的新 app 都应该支持 light mode 和 dark mode，某些就是不出夜间模式的 app 看你们还怎么死扛，整个系统都黑了，就你还是白花花的，你好意思吗？说你呢，微信和即刻！

{% asset_img both_modes.jpeg %}

然后整体的设计目标是：

- 保持熟悉性，也就是不要把已经深入人心的习惯打破
- 保持所有平台的统一性
- 清晰的信息层级
- 易使用
- 保持简单的原则

#### Colors

{% asset_img colors.jpeg %}

* 颜色要有层级
* tint color 在 light mode 和 dark mode 下是不一样的
* light mode 和 dark mode 下要注意 tint color 和背景色的对比度
* elevated 的背景色要浅一些

{% asset_img tint_colors.jpeg %}

#### Materials

这里的 materials 就是指四种透明度，对应的翻译应该用「材质」更接近一些。

四种材质分别是：

* Thick
* Regular，默认
* Thin
* Ultrathin

{% asset_img materials.jpeg %}

Dark Mode 最后还有一些关于控件的设计规范，以及新推出的 SF Symbols，就不赘述了，想知道细节的就自行观看吧。

### Modal Presentations

这个东西的前身就是 sheet，现在 UI 上改成了卡片式，好处是可以 peek 到前一个页面的背景，这样不会因为页面的完全切换而失去上下文的关联。

Modal 意味着模式的切换，比如从全局切换到细节，从总览切换到编辑。

{% asset_img modal.jpeg %}

### Contextual Menus

这个东西其实就是 3D touch 的衍生物，也可以理解为 macOS 上各个 app 的右键弹出的菜单。这个菜单会根据使用场景的不同提供不同的 action，也支持用分组和颜色来区分各个 actions，还可以根据屏幕大小以及屏幕方向来调整出现的位置。

{% asset_img contextual_menus.jpeg %}