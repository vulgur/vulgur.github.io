---
title: 仿写知乎日报：侧边栏菜单
date: 2016-01-22
tags:
- 仿写
- CALayer
- UIGestureRecognizer
categories:
- 知乎日报
---

这一篇 blog 是专门介绍如何仿写知乎日报的侧边栏菜单，主要介绍如何构建 UI 以及一个渐隐的图层效果，和如何控制侧边栏的显示和隐藏。
{% asset_img sidebar.gif 侧边栏菜单 %}
<!-- more -->

##界面

首先是画 UI。侧边栏的 UI 比较简单，由上到下分别是：

1. 头像和登录按钮
2. 收藏、消息和设置按钮
3. 专栏的列表
4. 离线下载和切换主题的按钮

用 xib 做出来就是这样子：

{% asset_img sidebar_xib.png xib %}

值得一提的是专栏列表的底部，在滑动的时候会有一个渐隐的效果，如图所示：

{% asset_img list_bottom.gif 列表底部的渐隐 %}

实现这个效果的第一个想法是在 UITableView 的底部改一个带渐变效果的透明 UIView，但是原版中处于渐隐效果的底部 Cell 也是可以点击的，这样子遮罩的 UIView 还要处理一下点击事件，麻烦，弃用！第二个想法就和 Part 2 中一样，给 UITableView 设置一个 CAGradientLayer，专门在底部加一个渐变的效果，但是试验了几次，发现渐变的效果十分不理想，所以也放弃了。

最后的解决方案，也是我无意中发现的。UIView 的 layer 有个属性是 mask, 根据文档里面写的，这也是个 CALayer，用来给 layer 的 alpha 通道设置遮罩。但是，这个属性必须在滚动的时候不断重新设置，不然遮罩会固定在一个地方。所以更新的代码就在 scrollViewDidScroll: 里：

{% codeblock lang:objc %}
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    UIColor *backgroundColor = [UIColor colorWithRed:0.106 green:0.125 blue:0.141 alpha:1];
    CAGradientLayer *gradientLayer = [CAGradientLayer layer];
    gradientLayer.frame = self.menuTableView.bounds;
    gradientLayer.colors =
    	@[ (id)[UIColor clearColor].CGColor, (id)backgroundColor.CGColor];
    gradientLayer.endPoint = CGPointMake(0.5, 0.8);
    gradientLayer.startPoint = CGPointMake(0.5, 1);
    self.menuTableView.layer.mask = gradientLayer;
}
{% endcodeblock %}

这段代码实现的就是每次滚动的时候，都新建一个 CAGradientLayer，设置好大小、渐变的颜色和位置，然后将其赋给 UITableView 的 layer 的 mask。但是这样肯定会有性能问题，可我也没想到其他的方法，如果有更好的解决方案还请赐教！

## View 层次

弄好了 UI，接下来就是交互操作了。显示侧边栏有两个方式：一是点击主页面左上角的菜单按钮，二是向右滑动。同样隐藏侧边栏也有两个方式：一是点击主页面的任意区域，二是向左滑动。两个的根本区别就是，一个是 tap，一个是 slide。

这里先说一个 Part 1 中遗留的东西，就是主页面的 view （以下简称 main view）是 HomeViewController 的 view（下面简称 root view） 的一个 subview。为什么要多套这一层呢？我最开始做的时候是没有这一层的，也就是 root view 就是主页面的 View，侧边栏的 view （以下简称 side view）是作为 root view 的 subview 放在 root view 的左侧。移动两个 view 的动画代码就是这个样子的（半伪代码，示意而已）：

{% codeblock lang:objc %}
[UIView animateWithDuration:kSideMenuAnimationDuration
                     animations:^{
                     	self.view.left = 225;
                     	self.sideMenuVC.view.left = 0;
                     }];
{% endcodeblock %}

就是 root view 和 side view 同时向右移动一段距离。但是这样做的话，root view 向右移动后，side view 也无法显示出来，只是一个同样大小的黑色区域。我用 Reveal 看了一下，移动后 root view 的 frame 也右移了，也就是 origin 不在屏幕的左上角，而 side view 仍然是在 root view 的 bounds 之外（我猜想这是无法显示的原因）。

尝试之后，我的解决方法就是，main view 和 side view 都作为 root view 的 subview，移动的只是 main view 和 side view， 而 root view 是固定不动的，这就是多了一层的原因。

## Tap 操作

首先说如何通过点击来显示和隐藏侧边栏。因为我所有的 UI 都是靠 AutoLayout 来控制的，所以侧边栏和主页面的位置变换的动画也是用 AutoLayout。

HomeViewController 在加载的时候把 side view 加入到 root view 中，设置好位置和大小，这里的代码就不贴了。下面是点击菜单按钮显示侧边栏的代码：

{% codeblock lang:objc %}
- (IBAction)showSideMenu:(UIButton *)sender {
    self.homeViewLeft.constant = 225;
    self.homeViewRight.constant = -225;
    [self.homeView setNeedsUpdateConstraints];
    [UIView animateWithDuration:kSideMenuAnimationDuration
                     animations:^{
                         self.sideMenuVC.view.left = 0;
                         [self.view layoutIfNeeded];
                     }
                     completion:^(BOOL finished) {
                         // setup transparent view for tapping to hide the side menu
                         self.tapView = [[UIView alloc] initWithFrame:self.homeView.bounds];
                         self.tapView.backgroundColor = [UIColor clearColor];
                         [self.tapView addGestureRecognizer:self.tapToHideSideMenu];
                         [self.homeView addSubview:self.tapView];
                         self.isShowSideMenu = YES;
                     }];
}
{% endcodeblock %}

代码中的 homeViewLeft 和 homeViewRight 是从 StoryBoard 中拖过来的约束，通过改变这两个约束的值来移动 main view。

原版中侧边栏滑出后，移到右侧的 main view 就不可以上下滑动内容了，只可以点击或滑动隐藏侧边栏菜单。所以在上面的代码中，动画结束后，新建一个 tapView 盖在 main view 上面，这样就无法直接操作 main view 了，然后给这个 tapView 添加一个 tap 的手势，这个手势就是用来隐藏侧边栏的。下面是隐藏的代码：

{% codeblock lang:objc %}
- (void)hideSideMenu {
    [UIView animateWithDuration:kSideMenuAnimationDuration
                     animations:^{
                         self.homeView.left = 0;
                         self.sideMenuVC.view.left = -255;
                     }
                     completion:^(BOOL finished) {
                         [self.tapView removeGestureRecognizer:self.tapToHideSideMenu];
                         [self.tapView removeFromSuperview];
                         self.isShowSideMenu = NO;
                     }];
}
{% endcodeblock %}

这个动画里没有使用 AutoLayout，还是用 frame 来操作位置。在隐藏动画结束后，顺便把 tapView 干掉。

## Slide 操作

另外一种交互就是向左向右滑动，这里就用到了 UIPanGestureRecognizer。

滑动的细节有两点：

1. side view 和 main view 随着滑动手势同步移动。
2. 如果向右滑动到某一阈值，side view 就自动完全显示出来。反之，side view 就完全隐藏。

具体实现就是给 main view 添加一个 ‘UIPanGestureRecognizer’，手势的 action 代码如下：

{% codeblock lang:objc %}
- (void)handlePanGesture:(UIPanGestureRecognizer *)recognizer {
    CGFloat offsetX = [recognizer translationInView:self.homeView].x;
    if (offsetX > 0 && offsetX < kSideMenuWidth) {
        self.sideMenuVC.view.right = offsetX;
        self.homeViewLeft.constant = offsetX;
        self.homeViewRight.constant = -offsetX;
        [self.homeView layoutIfNeeded];
    }
    if (recognizer.state == UIGestureRecognizerStateEnded) {
        
        if (offsetX >= kSideMenuWidth / 2) {
            [self showSideMenu:nil];
        } else {
            [self hideSideMenu];
        }
    }
}
{% endcodeblock %}

这段代码中，第一个 if 是用 AutoLayout 和 frame 根据滑动距离同时移动两个 view；第二个 if 就是手势结束时判断是要隐藏还是显示，如果超过侧边栏宽度的一半，就进行显示动画，反之就进行隐藏动画。
