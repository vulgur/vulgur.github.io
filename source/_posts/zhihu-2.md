---
title: 仿写知乎日报：主页面补遗
date: 2016-01-08
tags:
- 仿写
- CALayer
categories:
- 知乎日报
---

{% asset_img home.png %}
这篇是仿写知乎日报系列的第二部分，内容是上一篇的补遗。
之前做得主页面还有以下几点没有完成：

1. 目前这个主页只做了展示，点击没有任何效果。
2. TableView 滑上再滑下的时候，topView 不会完全消失，可能会有淡淡地残留，这点还没有优化。
3. 原版的轮播图片底部和顶部有黑色阴影的渐变，这样在纯白的图片下，按钮和文字标题都可以清晰显示出来。
4. TableView 的内容会根据日期来分组。

因为主要做的是 UI 上的仿写，所以本篇仍然没有实现上述的第一条。

<!-- more -->

## 渐变效果

这里主要讲的是如何在 UIView 上实现一个渐变效果的 Layer，以及如何改变 Layer 的位置和大小。
如果轮播图片是一个深色的图片，那么一切都是和谐的。如果是一个顶部和底部都是白色的图片，那么就会出现顶部的按钮、状态栏以及标题文字都显示不出来的问题。就像下面一样：

{% asset_img banner_without_gradient.png 无渐变图层%}
{% asset_img banner_with_gradient.png 有渐变图层%}

增加一个渐变图层用的就是 CAGradientLayer，在初始化 bannerView 的时候新建一个 gradientLayer 并加入到 bannerView 的 layer 中：

{% codeblock lang:objc %}
self.bannerImageView.contentMode = UIViewContentModeScaleAspectFill;

_gradientLayer = [CAGradientLayer layer];
_gradientLayer.frame = self.bannerImageView.bounds;
_gradientLayer.colors = @[
                      (id)[UIColor colorWithWhite:0.2 alpha:0.6].CGColor,
                      (id)[UIColor clearColor].CGColor,
                      (id)[UIColor clearColor].CGColor,
                      (id)[UIColor colorWithWhite:0.2 alpha:0.6].CGColor
                      ];
_gradientLayer.locations = @[ @0.0, @0.4, @0.7, @1.0 ];
[self.bannerImageView.layer addSublayer:_gradientLayer];
{% endcodeblock %}

colors 就是指定渐变的颜色数组，locations 就是每个渐变颜色停止的位置（0.0 到 1.0）。一开始我以为三个颜色就可以了，但是试验后发现还是四个效果更好。

这时候就会出现一个问题，那就渐变图层的大小和位置是固定的，即使 UIView 的大小变了这个 layer 也不会变，就像下面这个样子：

{% asset_img layer_position.gif 渐变图层位置不动 %}


CALayer 是不支持 AutoLayout 的，所以不能像 UIView 那样设定约束。为了解决这个问题，只能是在 UIView 改变时同样改变 layer，所以就在 layoutSubviews 方法中做手脚：

{% codeblock lang:objc %}
- (void)layoutSubviews {
    [super layoutSubviews];

    self.gradientLayer.frame = self.bannerImageView.bounds;
}
{% endcodeblock %}

这样图层就会随着 view 的大小而同时改变了，你以为问题解决了？不，这样写的结果就是下面这样：

{% asset_img layer_delay.gif 渐变图层变化有延迟 %}

图层的变化居然有延迟（这是我的第一想法）？？？我也试过了 KVO，同样还是有延迟。百般折磨后，终于让我找到了答案（[猛戳此处](http://stackoverflow.com/questions/11896663/calayer-resize-is-slow)），原来 CALayer 的 bounds 是个 animatable 的属性，改变这个属性就会触发动画……不是延迟！

## 加载更多

Part1 中的主页只加载了最新的数据，并没有以往的数据，而原版的在滑动的时候就会加载以往的条目数据（我猜，懒得抓包分析）。我是在即将快滑到 UITableView 的底部时，就进行前一天数据的加载，这个逻辑是在 scrollViewDidScroll: 实现的：

{% codeblock lang:objc %}
// if scroll to bottom, then fetch the previous day
CGRect bounds = scrollView.bounds;
CGSize contentSize = scrollView.contentSize;
UIEdgeInsets inset = scrollView.contentInset;
float y = offset.y + bounds.size.height - inset.bottom;
float h = contentSize.height;
float reload_distance = 10;
if (y > h + reload_distance) {
    if (self.isLoading) {
        return;
    } else {
        NSLog(@"------> load more");
        [self fetchData];
    }
}
{% endcodeblock %}

最后实现的效果就是这样：

{% asset_img load_more.gif 滑动中加载 %}

## 内容分组

原版的 TableView 中的数据都是根据日期来分组的，也就是 Section。第一个 Section 的标题就是「今日新闻」，下面的 Section 的标题就是对应的日期。那么对应这样的 UI，存储新闻的数据结构就是一个二维数组，数组中的数组就是每一天的新闻组数，而且索引是按照日期排序的，也就是说 data[0] 就是今天的新闻数组，data[1] 就是昨天的新闻数组，data[2] 就是前天的新闻数组，以此类推。

所以要根据 section 的索引来新建对应的 Section Header，实现 UITableViewDelegate 中 的 tableView:viewForHeaderInSection:：

{% codeblock lang:objc %}
-(UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section {
    if (section == 0) {
        return nil;
    }
    UIView *header = [[UIView alloc]initWithFrame:CGRectMake(0, 0, kScreenWidth, 44)];
    header.backgroundColor = [UIColor colorWithRed:0.0667 green:0.478 blue:0.804 alpha:1];
    UILabel *dataLabel = [[UILabel alloc]initWithFrame:header.bounds];
    dataLabel.text = [WSDDataUtils dateStringBeforeDays:section];
    dataLabel.textColor = [UIColor whiteColor];
    dataLabel.textAlignment = NSTextAlignmentCenter;
    [header addSubview:dataLabel];

    return header;
}
{% endcodeblock%}

还有一个交互效果是，Section Header 在上滑的时候会一起悬停在顶部，并且下面的 Header 会替换掉上面的。除了昨日的 Header 替换「今日新闻」外，其他的替换都是 Plain UITableView 的默认效果（我以为是知乎自己实现的，还尝试了好久，自定义 Header 并试图拿到每一个 Section Header 的位置，并且通过 delegate 来自定义动画）。

Part1 中提过，显示「今日新闻」的其实是一个自定义的 topView，根据滑动 offset 来控制是否显示和透明度的。这里就要多加一个逻辑，就是今日的新闻都滑出屏幕时（也就是 offset.y 等于今日所有新闻的累计高度），隐藏这个 topView，好让昨日的 Section Header 不被盖住（我一开始就是因为被盖住了，才没有发现 UITableView 自带的悬停和替换的效果……），代码如下所示：

{% codeblock lang:objc %}
if (offsetY > 0) {
    if (offsetY >= 90 * [self.data[0] count] + 200) {
        self.todayTitleLabel.hidden = YES;
        self.topView.hidden = YES;
    } else {
        self.todayTitleLabel.hidden = NO;

        if (!self.topView) {
            self.topView =
            [[UIView alloc] initWithFrame:CGRectMake(0, 0, kScreenWidth, 64)];
            [self.homeView insertSubview:self.topView
                            belowSubview:self.showSideMenuButton];
        }
        self.topView.hidden = NO;
        CGFloat alpha = offsetY / 64;
        self.topView.backgroundColor = [UIColor colorWithRed:0.0667 green:0.478 blue:0.804 alpha:alpha];
    }
    self.carouselViewTop.constant = -offsetY;

} else {
    if (self.topView) {
        [self.topView removeFromSuperview];
        self.topView = nil;
    }
	...
}
{% endcodeblock%}

topView 的高度要和 Section Header 的高度一样，不然替换的时候 Header 的高度会变一下，不完美！另外， 这段代码还修复 Part1 中提到的 topView 没有完全隐藏的 bug。最后的实现就是下面这样：

{% asset_img section.gif 成品效果 %}
代码请戳 [github](https://github.com/vulgur/WSDZhihuDaily)。

