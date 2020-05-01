---
title: Sources 开发日记五：代码展示页面 
date: 2016-08-28
tags:
- syntax highlight
- swift
- mvvm
- github
categories:
- Sources
---

好久没写 blog 了，上一篇距离现在居然有一个半月了，而上一篇关于 Sources 的开发日记竟然相隔快两个月了。得赶快把 v1.0 的最后两篇写完，然后全身心进入下一个版本的开发和记录。

这一篇讲的部分应该是整个 App 最出彩的地方（颜色很多），展示代码，而且是带有语法高亮的展示。

<!-- more -->

## 语法高亮的实现方案

如何给代码在 iOS 上进行语法着色显示，这是这个 app 开发前我能想到的最大的一个难题。我看过几篇关于利用 Core Text 来给文本中不同的部分来设置不同颜色的 blog，但是前提是要知道哪些部分是哪些类型。关键字，字符串，数字，自建类型，注释……要把这些东西从一个代码文件中分析出来，实现一个语法分析器，对于现在的我来说简直不可能。

于是在 Google 里搜索「code highlight」，找到了我的解决方案，[https://highlightjs.org](https://highlightjs.org)。这个网站就是在 Web 端给各种语言（目前是166种）提供语法高亮，而且包含了多种主题（目前是77种）。只需要在网页中使用这个网站提供的 js，并指定代码段的 class 就可以实现梦想中的语法高亮！

所以在 iOS 端如何实现也就确定了，web view。

{% asset_img code.png Syntax-highlighten Code %}

## 加载代码

### HTML Template

既然是 web view，就需要跟 HTML 打交道了。根据 Github API 下载得到的代码是 plain text，想要语法高亮就要按照 highlightjs 的教程加载 js 到 HTML，所以要自定义一个 HTML 模板用来加载 js 和 代码。

{% codeblock HTML Template lang:html %}
<!DOCTYPE html>
<html lang="en">
<head>
    <title>#title#</title>
    <link rel="stylesheet" href="#theme#.css">
    <meta name='viewport' content='initial-scale=1.0; maximum-scale=2.0;'>
    <script src="highlight.pack.js"></script>
    <script>hljs.initHighlightingOnLoad();</script>

</head>
<body>
<pre><code class="hljs">

#code#

</code></pre>
</body>
</html>

{% endcodeblock%}

以上代码中的 `#title` `#theme` `#code` 都是 placeholder，在获取代码后用文件名、选择的主题和代码文本来替换。

在 `CodeViewController` 中下载代码之前需要获得这个模板的字符串：

{% codeblock Get the string of html template lang:swift %}
private func htmlTemplateString() -> String? {
    let path = NSBundle.mainBundle().URLForResource("template", withExtension: "html")!
    let str: String?
    do {
        str = try String(contentsOfURL: path)
    } catch {
        str = nil
    }
    return str
}

{% endcodeblock %}

### 下载代码

我用的是 WKWebView。因为 WKWebView 目前还无法在 Storyboard 中使用，所以只能在 code 中进行设置。

{% codeblock WKWebView configuration lang:swift %}
override func viewDidLoad() {
    super.viewDidLoad()

    navigationItem.title = file.name

    let config = WKWebViewConfiguration()
    config.preferences.javaScriptEnabled = true
    webView = WKWebView(frame: view.bounds, configuration: config)
    view.insertSubview(webView, belowSubview: favoriteButton)
    self.theme = NSUserDefaults.standardUserDefaults().stringForKey("default_theme") ?? "default"
    downloadSourceCode()
}
{% endcodeblock %}

语法高亮的主题默认是 default，如果用户有选择其它主题就会存入 User Defaults 中，这样 `CodeViewController` 在每次加载后都会设置为上一次选择的主题。

下载代码的逻辑是这样的：

1. 先获取 HTML 模板和下载API
2. 根据 API 去下载代码
3. 如果 API 对应的文件是代码文件（文本文件）就将代码字符串进行转义、placeholder 替换，用 web view 加载
4. 如果不是代码文件，就提示用户，并直接返回文件列表

{% codeblock Download Code lang:swift%}
private func downloadSourceCode() {
    if let template = htmlTemplateString(), downloadURLString = file.downloadURLString {

        let url = NSURL(string: downloadURLString)!

        EZLoadingActivity.show("loading source", disableUI: true)

        Alamofire.request(.GET, url)
            .responseData(completionHandler: { (response) in
                EZLoadingActivity.hide()
                self.setFavoriteButton()
                if let htmlData = response.data {
                    if let dataString = String(data: htmlData, encoding: NSUTF8StringEncoding) {
                        let escapeString = dataString.stringByReplacingOccurrencesOfString("<", withString: "&lt;")
                            .stringByReplacingOccurrencesOfString(">", withString: "&gt;")
                        self.contentString = escapeString
                        let htmlString = template.stringByReplacingOccurrencesOfString("#code#", withString: escapeString)
                            .stringByReplacingOccurrencesOfString("#title#", withString: self.file.name ?? "")
                            .stringByReplacingOccurrencesOfString("#theme#", withString: self.theme)
                        dispatch_async(dispatch_get_main_queue(), {
                            self.webView.loadHTMLString(htmlString, baseURL: NSBundle.mainBundle().bundleURL)
                        })
                    } else {
                        // not a text file, show alert and then pop back

                        let alertController = UIAlertController(title: "", message: "This file is not a source code file", preferredStyle: .Alert)
                        let alertAction = UIAlertAction(title: "OK", style: UIAlertActionStyle.Default, handler: { ( _ ) in
                            self.navigationController?.popViewControllerAnimated(true)
                        })
                        alertController.addAction(alertAction)
                        RecentsManager.sharedManager.recents.removeFirst()
                        self.presentViewController(alertController, animated: true, completion: nil)
                    }
                }
            })
    }
}

{% endcodeblock %}

## 语法高亮主题列表

其它的代码查看 App 关于语法高亮主题的选择上都只是列出个名称列表而已，通过名称并不能直观的知道该主题是个什么样子的，必须设置后才能一窥究竟。
我在这部分做了一点微小的工作：将主题的整体配色加入到列表中。

{% asset_img theme_list.png Theme List %}

用户点击一个 theme 后，会返回到 CodeViewController 并重新加载上面的 HTML string，具体代码就不贴了，可以到我的 github 中查看。
弄这个 list 纯是个手工活，因为这个 list 并不是动态加载的，而是我将各个主题的配色都手工提取出来做成了一个数组：

{% asset_img theme_array.png Theme array %}

要问我为什么要手工搞这个数组，下面就是原因。

### Theme CSS

highlightjs 中每一个 theme 其实是一个 css，就是定义了 HTML 中各个 class 的颜色，以 default 为例：

{% codeblock Default theme lang:css %}
...

.hljs {
  display: block;
  overflow-x: auto;
  padding: 0.5em;
  background: #F0F0F0;
}


/* Base color: saturation 0; */

.hljs,
.hljs-subst {
  color: #444;
}

.hljs-comment {
  color: #888888;
}

.hljs-keyword,
.hljs-attribute,
.hljs-selector-tag,
.hljs-meta-keyword,
.hljs-doctag,
.hljs-name {
  font-weight: bold;
}


/* User color: hue: 0 */

.hljs-type,
.hljs-string,
.hljs-number,
.hljs-selector-id,
.hljs-selector-class,
.hljs-quote,
.hljs-template-tag,
.hljs-deletion {
  color: #880000;
}

.hljs-title,
.hljs-section {
  color: #880000;
  font-weight: bold;
}

.hljs-regexp,
.hljs-symbol,
.hljs-variable,
.hljs-template-variable,
.hljs-link,
.hljs-selector-attr,
.hljs-selector-pseudo {
  color: #BC6060;
}


/* Language color: hue: 90; */

.hljs-literal {
  color: #78A960;
}

.hljs-built_in,
.hljs-bullet,
.hljs-code,
.hljs-addition {
  color: #397300;
}

...
{% endcodeblock %}
每一个 css 中的 class 都不尽一样，不过大部分还都是定义了四五个颜色，分别对应背景、关键字、自定义类型、字符串、数字和注释等等。因为 css 没有一个统一的格式标准，所以想靠动态读取来获得这些颜色还是要比搞个固定的数组麻烦许多，毕竟这个数组也不是太大，才70多个元素。
