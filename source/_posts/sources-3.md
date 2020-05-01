---
title: Sources 开发日记三：Repo页面
date: 2016-07-01
tags:
    - app
    - swift
    - mvvm
    - github
categories:
- Sources
---

>Code Reader 改名为 Sources，已经上架  
>App Store: http://itunes.apple.com/app/id1125732186。
>同时 Sources 也在 Github 上开源了，地址是：https://github.com/vulgur/Sources。

这篇写展示 Repo 信息的页面，也是 MVVM 应用最多的一个页面。

<!-- more -->

{% asset_img repo.png Repo Page %}

## Model

Model 用的依然是上一篇提到的 `Repo` 类，是在 `SearchRepoViewControler` 通过 segue 传过来的。

{% codeblock SearchRepoViewControler lang:swift %}
extension SearchViewController: UITableViewDelegate {
    func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
        let repo = viewModel.repos[indexPath.row]
        let repoViewModel = RepoViewModel(repo: repo)
        performSegueWithIdentifier("ShowRepo", sender: repoViewModel)
        tableView.deselectRowAtIndexPath(indexPath, animated: true)
    }

}
{% endcodeblock %}

{% codeblock Segue lang:swift %}
override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
    if segue.identifier == "ShowRepo" {
        if let repoViewModel = sender {
            let destVC = segue.destinationViewController as! RepoViewController
            destVC.viewModel = repoViewModel as! RepoViewModel
        }
    }
}
{% endcodeblock %}

## View Model

作为 `RepoViewController` 的 view model, RepoViewModel 定义了所有需要和 Repo 绑定的属性，并且通过 Repo 的实例来初始化。除此之外，上一篇说到 watchers 的获取需要另一个 API，也就是 https://api.github.com/repos/{owner}/{repo}/subscribers，获取 watchers 数量的方法也定义在这个 view model 中。

{% codeblock RepoViewModel lang:swift %}
import Foundation
import Bond
import Alamofire

class RepoViewModel {
    let avatarImageURLString = Observable("")
    let name = Observable("")
    let description = Observable("")
    let stars = Observable(0)
    let watchers = Observable(0)
    let forks = Observable(0)
    let owner = Observable(Owner())
    let createdDate = Observable("")
    let updatedDate = Observable("")
    let language = Observable("")
    let ownerName = Observable("")
    let fullName = Observable("")
    let size = Observable(0)

    init(repo: Repo) {
        avatarImageURLString.value = (repo.owner?.avatarURLString)!
        name.value = repo.name ?? ""
        description.value = repo.description ?? ""
        stars.value = repo.starsCount ?? 0
        watchers.value = repo.watchersCount ?? 0
        forks.value = repo.forksCount ?? 0
        owner.value = repo.owner!
        createdDate.value = repo.createdDate ?? ""
        updatedDate.value = repo.pushedDate ?? ""
        language.value = repo.language ?? "Unknown"
        ownerName.value = repo.owner!.name ?? ""
        size.value = repo.size ?? 0
        fullName.value = repo.fullName ?? ""
    }


    func fetchWatchers() {
        let url = String(format: "https://api.github.com/repos/%@/%@/subscribers",
                         owner.value.name!, name.value)
        print(url)
        Alamofire.request(.GET, url)
        .responseJSON { (response) in
            if let watchers = response.result.value as? NSArray {
                self.watchers.value = watchers.count
            }
        }
    }
}
{% endcodeblock %}


## View

Repo 的 view 由两部分组成：storyboard 和 `RepoViewController`。

{% asset_img repo_design.png Repo UI %}

这个页面是目前项目里信息最多，布局也是最复杂的。以中间的按钮为分界线，上半部分大量使用了 Stack View，下半部分用一个 Web View 来显示 README 的内容。

因为这个页面的高度是不确定的，所以所有控件都是放在一个 content view 中，而这个 content view 又是放在一个 scroll view 中，结构如下图：

{% asset_img view_hierarchy.png View Hierarchy%}

这个页面里除了 web view，其余控件的高度都是确定的，想要确定整个 scroll view 的高度就需要确定这个 web view 的高度。我以前的做法，就是要在代码中改变 scroll view 的 contentSize，不过按照[苹果给出的方法](https://developer.apple.com/library/ios/technotes/tn2154/_index.html)通过 Autolayout 就可以实现 scroll view 按照内容自动调整大小（不用管 scroll view 的 contentSize 了，但是 web view 的高度还是要计算的，这个下面会说。）。所以就是给 scroll view 添加四个约束，上下左右贴紧 root view。

再来看 content view 的约束：
{% asset_img content_view_constraints.png Content View Constraints%}

除了设置了与 scroll view 的约束关系，还设置了和 root view 的约束关系，也就是宽和高。这个高度的约束设置成了 Remove at build time，因为这个高度在后面是要重新计算的，这里只是个 placeholder，不让 storyboard 出警告。

## View Controller

`RepoViewController` 里面主要还是做的就是通过 view model 绑定 UI 和设置 UI。
绑定 UI 就是把各个 UILabel 和 view model 中对应的属性绑定，因为都只是在 UILabel 中显示的 text，所以都是单向绑定：

{% codeblock UI Binding lang:swift %}
private func bindViewModel() {

        viewModel.name.bindTo(repoNameLabel.bnd_text)
        viewModel.ownerName.bindTo(navigationItem.bnd_title)
        viewModel.description.bindTo(repoDescriptionLabel.bnd_text)
        viewModel.stars.map {"\($0)"}.bindTo(starsLabel.bnd_text)
        viewModel.forks.map {"\($0)"}.bindTo(forksLabel.bnd_text)
        viewModel.watchers.map {"\($0)"}.bindTo(watchersLabel.bnd_text)
        viewModel.createdDate.map{ $0.componentsSeparatedByString("T").first }.bindTo(createdDateLabel.bnd_text)
        viewModel.updatedDate.map{ $0.componentsSeparatedByString("T").first }.bindTo(updatedDateLabel.bnd_text)
        viewModel.size.map {  String(format: "%.2fMB" , Float($0)/1024) }.bindTo(sizeLabel.bnd_text)
        viewModel.language.bindTo(languageLabel.bnd_text)

        avatarImageView.kf_setImageWithURL(NSURL(string: viewModel.avatarImageURLString.value)!, placeholderImage: UIImage(named: "user_avatar"))

        //RecentsManager.sharedManager.currentRepoName = viewModel.name.value
        //RecentsManager.sharedManager.currentOwnerName = viewModel.ownerName.value
    }
{% endcodeblock%}

设置 UI 的工作中主要讲讲 web view 的高度是如何设置的（web view 内容的获取以及展示放到后面和代码页面一起讲）。
如何得到 UIWebView 加载完成后的高度，搜到的结果大多数通过JS解决的。但是我通过这个方法得到的高度不准确，总是要高于实际高度，而且如果内容越长得到的高度就越大于实际高度，导致页面下方出现大片的空白。好在我搜到了一个方法完美解决了这个问题（[就是这个问题中的最佳答案](http://stackoverflow.com/questions/3936041/how-to-determine-the-content-size-of-a-uiwebview)）：

{% codeblock WebView Height lang:swift %}
extension RepoViewController: UIWebViewDelegate {
    func webViewDidFinishLoad(webView: UIWebView) {

        var frame = webView.frame
        frame.size.height = 1
        webView.frame = frame
        let fitSize = webView.sizeThatFits(CGSizeZero)
        frame.size = fitSize
        webView.frame = frame
        self.view.layoutIfNeeded()

        let webViewHeight = frame.size.height
//        let webViewHeight = (webView.stringByEvaluatingJavaScriptFromString("document.body.offsetHeight;")! as NSString).floatValue
//        let webViewHeight = (webView.stringByEvaluatingJavaScriptFromString("document.body.scrollHeight")! as NSString).floatValue
//        let webViewHeight = (webView.stringByEvaluatingJavaScriptFromString("document.height")! as NSString).floatValue
        let contentViewHeight = CGFloat(webViewHeight) + webView.frame.origin.y
        self.contentView.addConstraint(NSLayoutConstraint(item: self.contentView, attribute: .Height, relatedBy: NSLayoutRelation.GreaterThanOrEqual, toItem: nil, attribute: .NotAnAttribute, multiplier: 1, constant: contentViewHeight))
        self.view.layoutIfNeeded()
        EZLoadingActivity.hide()
    }
}
{% endcodeblock %}
这个方法看起来貌似奇技淫巧，`sizeThatFits` 这个方法的注释是这么写的：**return ‘best’ size to fit given size. does not actually resize view. Default is return existing view size**。我也不清楚为什么传入 CGSizeZero 就会返回正确的大小，还请知情人士告知，谢谢！这段代码中注释的部分就是用JS来获取高度的方法，这三句都不行。得到正确的高度后就可以设置 content view 的高度了，给 content view 添加一个高度的约束，这样 scroll view 就可以自动调整 contentSize 了。