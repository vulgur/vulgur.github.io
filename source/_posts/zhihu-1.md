---
title: 仿写知乎日报：主页面
date: 2016-01-06
tags:
- 仿写
categories:
- 知乎日报
---

失业后发现自己的项目经验太少了，除了公司的 App 和自己的游戏之外就几乎为零了，所以必须要增加自己的实战经验。之前写 UI 都是纯代码，为了熟悉 Storyboard，特地选择了知乎日报来练手。

在之前这个[仿知乎日报的 iOS App](https://github.com/hshpy/HPYZhiHuDaily) 已经很出名了，我也是借鉴（就是抄）了部分的实现，图片和 API 则是完全照搬。当然我也有我自己的不同之处，我的 UI 是尽可能采用 Storyboard+xib 来实现，另外也在一些细节上更贴近正版知乎日报。

<!-- more -->

这篇主要讲一下首页的实现，以下是动图。

{% asset_img zh-home.gif 首页效果 %}

## 首页结构

首页主要由以下几部分组成：

1. 顶部的图片轮播
2. 下面的 TableView
3. 顶部的其他小东西：展开侧边栏的按钮，刷新控件，「今日新闻」的标题，和一个随着 TableView 上滑出现的 View

## 上滑效果

先说一下 TableView 的实现。首先自定义一个 UITableViewCell，按照原版的把大小和位置设定好，这个不复杂，如下图：

{% asset_img story_cell.jpg 自定义 Cell %}

接下来弄 TableView，这个 TableView 是和父视图同样大小的，也就是充满屏幕（注意，TableView 的父视图不是 HomeViewController 的 UIView，而是其下的 Subview，轮播视图以及其他控件都是放在这个 View 中的，至于为什么不直接放在 HomeViewController 的 View 里面，下一篇讲侧边栏实现的时候再解释……）。

在视觉上，第一感觉这个 TableView 好像应该是放在轮播图片的下面的（也就是 TableView 的 top 贴着轮播图片的 bottom），最开始我也是这样做的。
但是后来做上滑效果的时候才发现这样不行，因为上滑的时候需要 Cell 和轮播图片同时向上移动，这样 TableView 的 origin 就会改变，contentOffset 就不好计算了，而轮播图片的移动全靠这个 offset 来决定。

我也试过将 TableView 的初始 contentOffset 设为轮播图片的下面，但是滑上去就下不来了……所以，最后的解决办法是将 TableView 铺满屏幕，上面加一个和轮播图片同样高度的 Header，完美！

{% asset_img home_view.jpg 首页的 Storyboard %}


上面说过，上滑效果全靠 TableView 的 contentOffset 来实现。HomeViewController 要实现 UIScrollViewDelegate 中的 scrollViewDidScroll: 这个方法。 在这个方法里面，加入以下代码：

{% codeblock lang:objc %}
CGFloat offsetY = scrollView.contentOffset.y;
if (offsetY > 0) {
    if (!self.topView) {
        self.topView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, kScreenWidth, 64)];
        [self.homeView insertSubview:self.topView belowSubview:self.showSideMenuButton];
    }
    CGFloat alpha = offsetY / 64;
    self.topView.backgroundColor = [UIColor colorWithRed:60.f / 255.f
                                                   green:198.f / 255.f
                                                    blue:253.f / 255.f
                                                   alpha:alpha];
    self.carouselViewTop.constant = -offsetY;

}
{% endcodeblock%}

这段代码做了这些：

1. 首先判断 contentOffset.y，如果大于零那就是在向上滑动。
2. 如果 topView 不存在的话，就新生成一个，并且 topView 的背景颜色随着滑动距离变化。
3. 最后设置轮播图片距离父视图 top 的约束，这个变量是从 Storyboard 中拖过来的，将这个约束设为 -offsetY 就可以实现轮播图片和 Cell 一起向上滑动的效果了。

还需要注意的是，展示侧边栏的按钮，还有刷新控件和「今日新闻」的 UILabel，必须在层级上高于这个 topView，不然就会被 topView 盖住。

有一个小细节的地方，困扰了我好久，就是 TableView 的第一个 Cell 和上面的轮播图片始终有一段距离。最后各种尝试和搜索后才找到解决方法：在 Storyboard 中选中 HomeViewController，在 Attributes Inspector 中把 **Adjust Scroll View Insets** 这个选项勾掉。

{% asset_img adjust_scroll_view_insets.jpg 设置 ViewController 的属性 %}

## 图片轮播

这部分在实现思路上基本完全借鉴了上面提及的那个仿作，整个控件的容器是一个 UIScrollView，里面并排摆放所有的图片，还有一个 UIPageControl 来显示对应的索引。

{% asset_img carousel_view.jpg 轮播容器的 Storyboard %}

自定义一个 BannerView，用来显示每一个轮播的图片以及标题。上面的容器里装的就是这个 View。

{% asset_img banner.jpg 每一个轮播的 View %}


重要的轮播逻辑是这样的：通过 API 获取的轮播个数是5个，但是容器中的 View 是7个（5+2）。这一排的 BannerView 按照序号是这样排列的，5-1-2-3-4-5-1，也就是把第一个和最后一个复制一份添加到数组的尾和头。

而 ScrollView 的初始 offset 是数组的第二个（也就是序号为1的）。这样，1在右划的时候会在左面显示5，5在左划的时候会显示1。如果 ScrollView 的 contentOffset 停留在数组的第一个（5），那么就把 contentOffset 设为数组的第6个（正确顺序的5）。同理，如果 ScrollView 的 contentOffset 停留在数组的最后一个（1），那么就把 contentOffset 设为数组的第2个（正确顺序的1）。这样就实现了一个可以无限循环的轮播。

相关代码如下：

{% codeblock lang:objc %}
-(void)scrollViewDidScroll:(UIScrollView *)scrollView {
    CGFloat offsetX = scrollView.contentOffset.x;
    if (offsetX ==  6 * kScreenWidth) {
        _scrollView.contentOffset = CGPointMake(kScreenWidth, 0);
        _pageControl.currentPage = 0;
    } else if (offsetX == 0) {
        _scrollView.contentOffset = CGPointMake(5 * kScreenWidth, 0);
        _pageControl.currentPage = 4;
    } else {
        _pageControl.currentPage = offsetX/kScreenWidth - 1;
    }
}
{% endcodeblock %}

具体运行时的效果如下：

{% asset_img reveal.jpg Reveal 截图 %}

## 刷新动画

这里有两部分内容，一是刷新控件的实现，二是刷新控件的控制。

刷新控件的是由两部分组成的，一个 UIActivityIndicatorView 和由 CAShapeLayer 绘制的圆环。
定义一个 RefreshView，初始化中加入以下代码：

{% codeblock lang:objc %}
-(void)customInit {
    _indicatorView = [[UIActivityIndicatorView alloc]initWithFrame:self.bounds];

    _grayCircleShapeLayer = [CAShapeLayer layer];
    _grayCircleShapeLayer.lineWidth = 2.f;
    _grayCircleShapeLayer.strokeColor = [UIColor grayColor].CGColor;
    _grayCircleShapeLayer.fillColor = [UIColor clearColor].CGColor;
    _grayCircleShapeLayer.opacity = 0;
    _grayCircleShapeLayer.path = [UIBezierPath bezierPathWithOvalInRect:self.bounds].CGPath;

    _whiteCircleShapeLayer = [CAShapeLayer layer];
    _whiteCircleShapeLayer.lineWidth = 2.f;
    _whiteCircleShapeLayer.strokeColor = [UIColor whiteColor].CGColor;
    _whiteCircleShapeLayer.fillColor = [UIColor clearColor].CGColor;
    _whiteCircleShapeLayer.opacity = 0;
    _whiteCircleShapeLayer.path = [UIBezierPath bezierPathWithArcCenter:CGPointMake(self.width/2, self.width/2)
                                                                 radius:self.width/2
                                                             startAngle:M_PI_2 endAngle:M_PI * 5 / 2
                                                              clockwise:YES].CGPath;
    _whiteCircleShapeLayer.strokeEnd = 0;

    [self addSubview:_indicatorView];
    [self.layer addSublayer:_grayCircleShapeLayer];
    [self.layer addSublayer:_whiteCircleShapeLayer];
}
{% endcodeblock%}

圆环由一个灰色的背景圆环和一个表示进度的白色圆弧组成，下拉过程中更新白色圆弧的长度，到指定位置后，整个圆环消失，开始 UIActivityIndicatorView 的动画。
更新圆环进度的代码如下：

{% codeblock lang:objc %}
-(void)updateProgress:(CGFloat)progress {
    if (progress <= 0) {
        _whiteCircleShapeLayer.opacity = 0;
        _grayCircleShapeLayer.opacity = 0;
    } else {
        _whiteCircleShapeLayer.opacity = 1;
        _grayCircleShapeLayer.opacity = 1;
    }

    if (progress > 1) {
        progress = 1;
    }
    _whiteCircleShapeLayer.strokeEnd = progress;
}
{% endcodeblock %}

对刷新控件的控制其实和上滑的控制一样，也在 HomeViewController 中的 scrollViewDidScroll: 中，这部分逻辑就是 offsetY < 0 的那一部分。

{% codeblock lang:objc %}
self.carouselViewHeight.constant = 220 - offsetY;
if (offsetY <= -kRefreshOffsetY * 1.5) {
   self.tableView.contentOffset = CGPointMake(0, -kRefreshOffsetY * 1.5);
} else if (offsetY <= 0 && offsetY >= -kRefreshOffsetY * 1.5) {
   if (self.isRefreshing) {
       [self.refreshView updateProgress:0];
   } else {
       [self.refreshView updateProgress:-offsetY / kRefreshOffsetY];
   }
}
if (offsetY < -kRefreshOffsetY && !scrollView.isDragging) {
   [self.refreshView startAnimation];
   self.isRefreshing = YES;
   dispatch_after(
                  dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)),
                  dispatch_get_main_queue(), ^{
                      [self.refreshView stopAnimation];
                      self.isRefreshing = NO;
                  });
}
{% endcodeblock %}

这段代码的逻辑有：

1. 下拉时增加轮播图片的高度。
2. TableView 不是无限下拉的，只能下拉到一个指定的位置，超过的话，TableView 就不再下滑了。
3. 下拉一段后再上滑，如果进入了刷新状态，不显示圆环；如果没进入刷新状态，那么就根据 下拉距离/下拉阈值 来更新圆环进度。
4. 如果下拉距离达到了阈值并且松手了（没有拖动），那么就进入刷新状态。我这里做了个2秒刷新时间。

## 遗留

1. 目前这个主页只做了展示，点击没有任何效果。
2. TableView 滑上再滑下的时候，topView 不会完全消失，可能会有淡淡地残留，这点还没有优化。
3. 原版的轮播图片底部和顶部有黑色阴影的渐变，这样在纯白的图片下，按钮和文字标题都可以清晰显示出来，这点我也没做。

另外，代码请戳 [github](https://github.com/vulgur/WSDZhihuDaily)。
