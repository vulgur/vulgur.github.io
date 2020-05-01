---
title: 用 Swift 刷 Leetcode No.917 - Reverse Only Letters
date: 2019-02-16 21:56:23
tags:
- algorithms
- string
- array
categories:
- leetcode

---

最近又又又开始刷 Leetcode 了。

整个过年期间一行代码都没写，自知编程也是个用进废退的技能，为了恢复手感，就先从一道道算法题来练手吧。另外一个原因是，刷 Leetcode 也算是成就之一，结合我刚刚建立的 BMS（Better Me System， 名字是我自己起的，是一个游戏化达到自我成就的方法集，具体内容可以参考这篇文章：[2018我的人生游戏指南-欲望与「成救」模式](https://sspai.com/post/52270)）。

这次刷题我用了 VSCode 的 [Leetcode 插件](https://marketplace.visualstudio.com/items?itemName=shengchen.vscode-leetcode)，真乃神器也，给作者一个👍。

No. 917 这道题就是只反转所有的字母字符，而不是数字和标点符号，所以关键点就在于判断一个字符是否是字母了。

<!-- more -->

### 如何判断一个字符是否是字母

在 Swift 中判断一个字符是否是字母（或是数字，或其他类型的字符），可以借助 `CharacterSet` 这个 struct，它有许多的属性来表示各种各样的字符集。

{% asset_img CharacterSet.png CharacterSet %}

其中的 `CharacterSet.letters` 就是所以字母字符的集合。但是这些属性所代表的集合并不是 `Set` 类型而是 `CharacterSet` ，判断某个字符是否在某个字符集里需要用到 `CharacterSet` 的 `contains` ，这个方法接受的参数类型是一个 `Unicode.Scalar`，所以还需要把 `Character` 转换一下才可以。判断的逻辑代码如下：

```swift
func isLetter(_ c: Character) -> Bool {
       let letters = CharacterSet.letters
       let unicodeScalar = c.unicodeScalars.first!
       return letters.contains(unicodeScalar)

}

```

如果遍历 String 用的是 `unicodeScalars` 就更方便一些了，因为遍历过程中元素的类型就是 `Unicode.Scalar` 。

还有一个判断是否是字母的方法，这个方法也是在学 C/C++ 时常用的方法，就是判断字符的 ASCII 码，这种方法在 swift 中是这样实现的：

```swift
func isLetter(_ c: Character) -> Bool {
       let letters = CharacterSet.letters
       let unicodeValue = c.unicodeScalars.first!.value
       if (unicodeValue > 64 && unicodeValue < 91) || (unicodeValue > 96 && unicodeValue < 123) {
           return true
       }
       return false
}
```



解决了判断的问题，接下来就是反转的问题了。

### 第一个想法，反转后插入符号

这是我脑中弹出的第一个想法，就是把整个字符串中所以非字母的字符及其位置记录下来，同时把所有字母过滤出来，然后把所有字母反转，最后按照记录的位置去插入其余的字符。

```swift
class Solution {
    func reverseOnlyLetters(_ S: String) -> String {
        let lettersSet = CharacterSet.letters
        var letters = [Character]()
        var notLetters = [(Int, Character)]()
        var i = 0
        for c in S.unicodeScalars {
            if lettersSet.contains(c) {
                letters.append(Character(c))
            } else {
                notLetters.append((i, Character(c)))
            }
            i += 1
        }
        var reversedLetters = Array(letters.reversed())
        for item in notLetters {
            reversedLetters.insert(item.1, at: item.0)
        }
        return String(reversedLetters)
    }
}
```



这个方法耗时16ms，只击败了57.50%。缺点就是需要额外存储字母字符和非字母字符及其位置。

### 对位填入反转后的字母

这是击败了97.50%的高分答案。我十分喜欢这个想法，就是把所有的字母入栈，然后再遍历一遍原字符串，遇到一个字母就将栈顶的字母弹出并替换，真是巧妙啊！

因为我并没有根据这个想法改写自己的代码，这里我就贴一下别人的代码吧：

```swift
class Solution {
    func reverseOnlyLetters(_ S: String) -> String {
        var alphas = [Character]()
        for c in Array(S) {
            if c.isAlpha() {
                alphas.append(c)
            }
        }
        
        var charArr = Array(S)
        for i in 0..<charArr.count {
            if charArr[i].isAlpha() {
                charArr[i] = alphas.popLast()!
            }
        }
        return String(charArr)
    }
}

extension Character {
    var ascii: Int {
        let charVal = self.unicodeScalars
        if let c = charVal.first {
            return Int(c.value)
        }
        return -1
    }
    func isAlpha() -> Bool {
        return ascii >= 97 && ascii <= 122 || ascii >= 65 && ascii <= 90
    }
}
```



### 前后遍历，交换字母

这个是效率最高的一个 solution，思想就是从头尾同时遍历字符串，遇到两个都是字母字符的情况就原地交换。

我用这个方法实现了一版，但是最快只有 12ms，而提交记录中最快的是 8ms。Leetcode 就是有这样的问题，同样的 solution 多次提交就会得到不同的 runtime，第一次提交有可能只击败了 45%，但第二次提交就有可能击败 99% 了。

```swift
class Solution {
    func reverseOnlyLetters(_ S: String) -> String {
        let letters = CharacterSet.letters
        var chars = Array(S)
        var left = 0
        var right = chars.count - 1
        while left < right {
            if !letters.contains(Unicode.Scalar(String(chars[left]))!) {
                left += 1
            } else if !letters.contains(Unicode.Scalar(String(chars[right]))!) {
                right -= 1
            } else {
                chars.swapAt(left, right)
                left += 1
                right -= 1
            }
        }
        return String(chars)
   }
}
```



### 结论

这道题给我最大的收获就是让我发现了 `CharacterSet` 这个好东西，另外遇到字符串和数组的反转问题优先考虑首位遍历交换元素这个方法。