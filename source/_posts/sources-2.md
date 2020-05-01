---
title: Sources 开发日记二：搜索
date: 2016-06-03
tags:
- search
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

这一篇写搜索 Repo 功能的实现。
输入 Repo name 关键字，返回搜索结果。说起来简单，细节其实还不少。

<!-- more -->
{% asset_img repo_search.gif Repo Search%}

## 架构 & 第三方库

这个 App 我第一次尝试使用 MVVM 架构来实现。框架我选择的是 Bond ，之所以没选 ReactiveCocoa，原因很简单：怎么学也学不会……RAC4 的源代码我读了三四遍，Demo 我也看了几个，但是无奈资质愚笨，败在了如何使用 Action 和 UI binding 上面（RAC4 不像 RAC2，缺少了 UI binding 的 extension，需要自己实现，不过他们正在考虑把 Rex 加入到 RAC 中）。选择 Bond 的原因是看了这个教程 {% asset_link https://www.raywenderlich.com/123108/bond-tutorial Bond Tutorial: Bindings in Swift%}，发现这个 framework 和 RAC 比较相似，但是使用起来更简单明了 ，作者也说了这是一个轻量级的 binding 框架（妈蛋，写这篇 blog 的时候发现 Bond 作者推荐 ReactiveKit，说是更快更高更强……过两天再折腾）

以下是项目中目前为止用到的第三方库，因为项目是用 Swift 写的，所以选用的库也都是 Swift 写的：

* Alamofire
* AlamofireObjectMapper
* ObjectMapper
* Kingfisher
* Bond
* EZLoadingActivity

## Model

先说 Model。搜索结果是 Repo 的列表，而 Repo 又包含着 Owner，所以先写这两个 model。这两个类的属性就是参照 Github API 返回的 JSON 字段来设计。这里选用 ObjectMapper 作为 JSON 和 Model 对象的转换器。

{% codeblock Owner.swift lang:swift %}

import ObjectMapper

class Owner: Mappable {
    var name: String?
    var ownerId: String?
    var avatarURLString: String?

    required init?(_ map: Map) {

    }

    init() {

    }
    // Mappable
    func mapping(map: Map) {
        name            <- map["login"]
        ownerId         <- map["id"]
        avatarURLString <- map["avatar_url"]
    }
}
{% endcodeblock %}

{% codeblock Repo.swift lang:swift %}
import ObjectMapper

class Repo: Mappable {

    var repoId: String?
    var name: String?
    var fullName: String?
    var owner: Owner?
    var description: String?
    var size: Int?
    var starsCount: Int?
    var watchersCount: Int?
    var language: String?
    var forksCount: Int?
    var createdDate: String?
    var pushedDate: String?

    required init?(_ map: Map) {

    }

    // Mappable
    func mapping(map: Map) {
        repoId          <- map["id"]
        name            <- map["name"]
        fullName        <- map["full_name"]
        owner           <- map["owner"]
        description     <- map["description"]
        size            <- map["size"]
        starsCount      <- map["stargazers_count"]
        language        <- map["language"]
        forksCount      <- map["forks"]
        createdDate     <- map["created_at"]
        pushedDate      <- map["pushed_at"]
    }
}
{% endcodeblock %}

这里注意一下，Repo 的 JSON 里面没有 watchers，尽管搜索 Repo 的 API 的返回结果里面有个「watchers_count」的字段，但是这并不是 Github 中的 watchers，这只是一个过时的字段，取代它的新字段就是「stargazers_count」，也就是 stars，这两个字段的值是一样的。至于真正的 watchers，其实是在另一个 API 中，这个以后再说。

## View Model

接下来是 ViewModel。`SearchRepoViewModel` 中的负责绑定的属性是搜索时的各种参数以及搜索结果列表，另外还负责「搜索」和「加载更多」这两个动作。网络库用的是著名的 Alamofire。
这里只贴加载下一页的代码片段，方法中的参数一个是加载完成后的处理（主要是刷新 tableView），一个是出错时的处理（显示 alert），这两个 closure 都在 view controller 中传入。

{% codeblock SearchRepoViewModel.swift lang:swift%}
func loadMore(completion completion: ()->(), errorHandler: ((String) -> ())? = nil) {

        currentPage += 1
        let urlParams = [
            "q": keyword,
            "sort" : sortType.rawValue,
            "page" : "\(currentPage)"
        ]
        // Fetch Request
        Alamofire.request(.GET, "https://api.github.com/search/repositories", parameters: urlParams)
            .responseJSON { (response) in
                switch response.result {
                case .Success:
                    if let statusCode = response.response?.statusCode{
                        switch statusCode{
                        case 200..<299:
                            if let items = response.result.value!["items"], results = Mapper<Repo>().mapArray(items) {
                                    self.repos.appendContentsOf(results)
                                    completion()
                            }
                        default:
                            self.currentPage -= 1
                            if let message = response.result.value!["message"], errorHandler = errorHandler {
                                errorHandler(message as! String)
                            }
                        }

                    }
                case .Failure(let error):
                    self.currentPage -= 1
                    if let errorHandler = errorHandler {
                        errorHandler(error.localizedDescription)
                    }
                }
            }
    }
{% endcodeblock %}

## View

最后是 View。这部分的代码最多，既包括 views 也包括 view controllers。搜索这部分有三个文件：

* SearchViewController.swift
* SearchRepoCell.swift
* SearchRepoCell.xib

## 搜索

关于 Cell 的两个文件就不详述了，没什么特别的。重点说说 SearchViewController。
首先是输入关键字进行搜索。UI 中用的是 `UISearchBar`，而不是自定义的 `UITextField`，所以在输入完点击键盘上的「搜索」按钮的动作需要实现 `UISearchBarDelegate` 的 `searchBarSearchButtonClicked` 这个方法：

{% codeblock SearchViewController.swift lang:swift %}
extension SearchViewController: UISearchBarDelegate {
    func searchBarSearchButtonClicked(searchBar: UISearchBar) {
        print("Search for: ", searchBar.text!)
        searchBar.endEditing(true)

        viewModel.keyword = searchBar.text!

        EZLoadingActivity.show("Searching...", disableUI: true)

        viewModel.searchRepos(completion: {
            self.tableView.reloadDataWithAutoSizingCells()
            EZLoadingActivity.hide()
            }, errorHandler: self.errorHandler)
    }
}
{% endcodeblock %}

点击「搜索」后，首先是隐藏键盘，然后将搜索框中的文本赋给 view model，利用 EZLoadingActivity 显示提示框，同时阻止其他 UI 操作，最后执行 view model 中的搜索。这里传给搜索方法的两个 closure，第一个就是成功搜索后刷新 table view 并隐藏提示框，第二个是搜索出错后的错误处理。

`reloadDataWithAutoSizingCells` 是我给 UITableView 增加的自定义的方法，为的是解决 table view 的一个 UI bug：就是 table view 第一次加载 Autolayout 的动态高度的 cell，会出现高度不正确的 bug。这个方法的实现是：

{% codeblock UITableView extension lang:swift%}
extension UITableView {
    func reloadDataWithAutoSizingCells() {
        self.reloadData()
        self.setNeedsDisplay()
        self.layoutIfNeeded()
        self.reloadData()
    }
}
{% endcodeblock %}

处理错误的 errorHandler 执行的代码就是隐藏提示框并弹出一个 alert，alert 的内容就是 API 中返回的错误信息。

{% codeblock error handler closure lang:swift%}
errorHandler =  { [unowned self] msg in
            EZLoadingActivity.hide()
            self.isLoading = false
            let alertController = UIAlertController(title: "", message: msg, preferredStyle: .Alert)
            let action = UIAlertAction(title: "Okay", style: .Default, handler: nil)
            alertController.addAction(action)
            self.presentViewController(alertController, animated: true, completion: nil)
        }
{% endcodeblock %}

### 条件搜索

Github 的搜索可以按照「best match」，「stars」，「forks」和「updated」这四个条件排序搜索，这个 App 我默认都是降序。
在搜索框下面有一个 UISegmentedControl ，四个 segment 就对应着四个搜索条件。代码中先给这个 segmented control 添加一个 target，关联的 action 如下：

{% codeblock SegmentedControl Action lang:swift %}
func searchSortChanged(sender: UISegmentedControl) {
        switch sender.selectedSegmentIndex {
        case 0:
            viewModel.sortType = .Best
        case 1:
            viewModel.sortType = .Stars
        case 2:
            viewModel.sortType = .Forks
        case 3:
            viewModel.sortType = .Updated
        default:
            viewModel.sortType = .Best
        }

        EZLoadingActivity.show("Loading", disableUI: true)
        viewModel.searchRepos(completion: {
            self.tableView.reloadDataWithAutoSizingCells()
            let topIndexPath = NSIndexPath(forRow: 0, inSection: 0)
            self.tableView .scrollToRowAtIndexPath(topIndexPath, atScrollPosition: UITableViewScrollPosition.Top, animated: true)
            EZLoadingActivity.hide()
        }, errorHandler: self.errorHandler)
    }
{% endcodeblock %}

### 加载更多

这个功能可以说是数据列表中的必备功能之一，也有多种实现方式。一开始我是通过 UITableViewDelegate 中的 `tableView(tableView: UITableView, willDisplayCell cell: UITableViewCell, forRowAtIndexPath indexPath: NSIndexPath)` 这个方法来实现的，即在即将显示某个 cell （一般是 data source 中的倒数第X个）的时候加载下一页的数据。这是 SO 上面很多答案都推荐的方式，不过我发现这个方式有个问题，就是向下滑动过那个触发点 cell 后再向上滑的话，那么会再次执行一次加载动作（后来发现其实加一个是否在加载中的判断就可以了）。
最后我采用的方式是实现 UIScrollViewDelegate 中的 `scrollViewDidEndDecelerating(scrollView: UIScrollView)`。通过判断 scroll view 是否快滚到底来加载下一页数据，代码如下：