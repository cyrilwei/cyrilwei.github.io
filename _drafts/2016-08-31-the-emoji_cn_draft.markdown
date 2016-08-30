---
layout: post
title:  "原来你是介样的emoji⁉️"
date:   2016/08/31 16:53:00 +0800
categories: Swift, iOS, Unicode
published: false
---

##### _Note: Because some browsers are not support all of those emoji yet, all the emoji in this article are presented with picture from Apple iOS 10 Beta version and some emoji in the code maybe not displayed correctly. 如果你好奇它们在你的浏览器中应该是什么样子的，请访问 [Full Emoji Data, v4.0 — Beta][4] 查看各个emoji在不同系统中的实现。_

在iOS 10最新的Beta版中，我发现Apple重绘了不少emoji，并且增加了很多性别相关的emoji，比如我很常用的这个<img src="/assets/images/2016/08/emoji/1F64B.png" class="emoji">动作，现在增加了男性版本<img src="/assets/images/2016/08/emoji/1F64B+2642.png" class="emoji">。

于是我很好奇的发给了iOS 9的用户，非常有意思的是，在iOS 9中，该emoji显示为'<img src="/assets/images/2016/08/emoji/1F64B_iOS9.png" class="emoji"><img src="/assets/images/2016/08/emoji/2642.png" class="emoji">'两个符号

...

在Swift中，`String`类型和`Character`类型保存的都是Unicode。所以我知道，这肯定跟Unicode有关。

经过简单的google，我在Unicode Consortium的Blog中发现了如下内容：

> "A proposed update of UTR #51, Unicode Emoji (Version 4.0) is available for public review and feedback. This new version covers a total of 2,243 emoji, an increase from the 1,788 in Version 3.0.
>
> There are several important changes in the proposed update. Three existing symbols have been newly classified as emoji: U+2640 FEMALE SIGN, U+2642 MALE SIGN, and U+2695 STAFF OF AESCULAPIUS. These are used in sequences to represent additional professions and to make gender distinctions among emoji."
>
> Excerpt From: Unicode.org, [Proposed Update UTR #51, Unicode Emoji (Version 4.0)][1]

那么这个'<img src="/assets/images/2016/08/emoji/2642.png" class="emoji">'就应该是`U+2642`啦，The male sign。

那看来这个男性版本的emoji好像真的是两个Unicode字符组成的。所以一个`Character`再也不是**一个**字符啦？

#### **Character**

`Swift`中的`Character`定义如下，它保存一个'Extended Grapheme Cluster'。

{% highlight swift %}
public struct Character : ExpressibleByExtendedGraphemeClusterLiteral, Hashable {
  // ...
}
{% endhighlight %}

> "Every instance of Swift’s Character type represents a single extended grapheme cluster. An extended grapheme cluster is a sequence of one or more Unicode scalars that (when combined) produce a single human-readable character."
>
> Excerpt From: Apple Inc. “The Swift Programming Language (Swift 3 beta).” iBooks. [https://itun.es/us/k5SW7.l][TSPL]

下面我就简单介绍一下Unicode下的emoji究竟是一种怎样的存在（你眼中的字符串将再也不是以前的字符串了）。

#### **Unicode**

> "Unicode is a computing industry standard for the consistent encoding, representation, and handling of text expressed in most of the world's writing systems."
>
> Excerpt From: Wikipedia.org, [Unicode][7]

那么Unicode是如何表示字符的呢？

#### **Code Point**

Unicode使用code point来表示字符。每一个code point都是一个数值，表示a single character。使用U+hex的形式表示，比如`U+0061`表示'a'，`U+01F64B`表示'<img src="/assets/images/2016/08/emoji/1F64B.png" class="emoji">'。目前这个code point的大小已经达到了21bits，从`U+000000`到`U+10FFFF`。

#### **Sequence**

除了用单一的code point表示字符的方式外，Unicode还使用被称为'sequence'的'组合'方式拼装字符。

在sequence中，各个code point都有自己的职责，主要分为Base，Variation，Modifier，ZWJ，Combining Marks。

**Base**

Base就是可以接受后续code point的修改的字符，不同的字符可以接受修改的能力不同。

**Variation**

有些字符可以用两种方式表达，一种看起来像emoji，另一种看起来像字符。比如<img src="/assets/images/2016/08/emoji/2618+FE0E.png" class="emoji">和<img src="/assets/images/2016/08/emoji/2618+FE0F.png" class="emoji">其实是同一个字符的不同presentation。

{% highlight swift %}
"\u{2618}\u{FE0E}" ==> "☘︎"
"\u{2618}\u{FE0F}" ==> "☘️"
{% endhighlight %}

这里就是一个sequence，每一个\u{}里都是一个code point，后面的code point决定这个字符的显示方式，含义如下：

> U+FE0E VARIATION SELECTOR-15 (VS15) for a text presentation  
> U+FE0F VARIATION SELECTOR-16 (VS16) for an emoji presentation
>
> Excerpt From: Unicode.org, [Unicode® Technical Report #51][2]

而<img src="/assets/images/2016/08/emoji/2618+FE0F.png" class="emoji">这种很漂亮的就是所谓的emoji presentation。

**Modifier**

目前的Emoji modifier主要是Emoji Fitzpatrick Type Modifier

Fitzpatrick type是一种描述肤色的scale。这个modifier的作用就是实现我们现在常见的，emoji里的不同肤色。<img src="/assets/images/2016/08/emoji/1F64B.png" class="emoji"><img src="/assets/images/2016/08/emoji/1F64B+1F3FB.png" class="emoji">

在代码中就是这样的（其中\u{}是swift中Unicode scalar的表示方式）

{% highlight swift %}
"\u{01F64B}"           ==> "🙋"
"\u{01F64B}\u{01F3FB}" ==> "🙋🏻"
"\u{01F64B}\u{01F3FC}" ==> "🙋🏼"
"\u{01F64B}\u{01F3FD}" ==> "🙋🏽"
"\u{01F64B}\u{01F3FE}" ==> "🙋🏾"
"\u{01F64B}\u{01F3FF}" ==> "🙋🏿"
{% endhighlight %}

没错，这些不同肤色的emoji也并不是像大多数人想象中的那种‘不同的emoji’。这些肤色不同的emoji，其实是同一个字符，通过不同的modifier来修饰得到的。

负责表示该emoji的Unicode只有一个`U+01F64B`，而通过后面的modifier让它呈现出不同的肤色版本。所以，每一个这样的emoji都有可能是由两个Unicode code point组合而成的，而负责呈现的系统，则要处理好这种组合，并展示适当的emoji图像。

在不支持这些modifier的系统里，看到的将是类似这样的

{% highlight swift %}
"\u{01F64B}\u{01F3FD}" ==> "🙋‌🏽"
{% endhighlight %}

Fitzpatrick type modifier被显示成色块，这个色块的颜色与它所表达的skin scale是对应的。所以以后如果看到这样的组合，那么表示“作者本来想表达这种肤色的emoji，奈何当前系统／软件不支持该modifier，所以显示成这样了。”

Unicode委员会定义，所有能看到皮肤的emoji，都需要支持Fitzpatrick type modifier。这些支持modifier的emoji被称为Emoji Modifier Base，也就是可以被modify的emoji。这也是为什么输入emoji的时候，只有这些emoji可以点出不同的变种。

**ZWJ**

ZWJ指的是 `U+200D ZERO WIDTH JOINER`，是一个特殊的Unicode code point。它会将前后两个元素连成一个单一的符号。并且可以在一个emoji中使用多次，well, 一个打破你的三观的例子是最好的。

{% highlight swift %}
"\u{01F468}\u{200D}\u{002764}\u{00FE0F}\u{200D}\u{01F48B}\u{200D}\u{01F468}"
 ==> "👨‍❤️‍💋‍👨"

"\u{01F468}\u{200D}\u{01F469}\u{200D}\u{01F467}\u{200D}\u{01F466}"
 ==> "👨‍👩‍👧‍👦"

"\u{01F469}\u{200D}\u{01F680}"
 ==> "👩‍🚀"

"\u{01F469}\u{200D}\u{01F3A8}"
 ==> "👩‍🎨"

"\u{01F46E}\u{200D}\u{2642}\u{FE0F}"
 ==> "👮‍♂️"

"\u{01F46E}\u{200D}\u{2640}\u{FE0F}"
 ==> "👮‍♀️"
{% endhighlight %}

它们被称为**ZWJ sequences**

是的，我知道，它们根本就不是什么‘字符’，他们是一种’描述用语言’。  
<img src="/assets/images/2016/08/emoji/1F468.png" class="emoji_middle"> + <img src="/assets/images/2016/08/emoji/2764.png" class="emoji_middle"> + <img src="/assets/images/2016/08/emoji/1F48B.png" class="emoji_middle"> + <img src="/assets/images/2016/08/emoji/1F468.png" class="emoji_middle"> = <img src="/assets/images/2016/08/emoji/Kiss.png" class="emoji_middle">  
<img src="/assets/images/2016/08/emoji/1F468.png" class="emoji_middle"> + <img src="/assets/images/2016/08/emoji/1F469.png" class="emoji_middle"> + <img src="/assets/images/2016/08/emoji/1F467.png" class="emoji_middle"> + <img src="/assets/images/2016/08/emoji/1F466.png" class="emoji_middle"> = <img src="/assets/images/2016/08/emoji/Family.png" class="emoji_middle">  
(注意这里的<img src="/assets/images/2016/08/emoji/2764.png" class="emoji">是in emoji presentation的. 还记得Variation中的`U+FE0F`么，在这里也有用，因为我们在组合一个emoji)

and furthermore, 我们也可以有一些职业描述  

<img src="/assets/images/2016/08/emoji/1F469.png" class="emoji_middle"> + <img src="/assets/images/2016/08/emoji/1F680.png" class="emoji_middle"> = <img src="/assets/images/2016/08/emoji/1F469+1F680.png" class="emoji_middle">  
<img src="/assets/images/2016/08/emoji/1F469.png" class="emoji_middle"> + <img src="/assets/images/2016/08/emoji/1F3A8.png" class="emoji_middle"> = <img src="/assets/images/2016/08/emoji/1F469+1F3A8.png" class="emoji_middle">  

还有我们文章开始提到的`U+2640`和`U+2642`，也出现在这里，是通过ZWJ来连接并修改emoji的。  

<img src="/assets/images/2016/08/emoji/1F46E.png" class="emoji_middle"> + <img src="/assets/images/2016/08/emoji/2642.png" class="emoji_middle"> = <img src="/assets/images/2016/08/emoji/1F46E+2642.png" class="emoji_middle">  
<img src="/assets/images/2016/08/emoji/1F46E.png" class="emoji_middle"> + <img src="/assets/images/2016/08/emoji/2640.png" class="emoji_middle"> = <img src="/assets/images/2016/08/emoji/1F46E+2640.png" class="emoji_middle">  

and you can image more.

所以，基本上这个ZWJ创造了一种emoji描述语言，随着Unicode的发展，我们可以期待更多更多的emoji在不增加code point数量的情况下，被‘描述’出来。而越来越多的传统符号或者普通emoji将拥有修饰其他emoji的能力。

**Combining Marks**

多数combining marks都是Diacritical Marks，具体可以参考[Combining character][8]。

{% highlight swift %}
"\u{65}"             ==> "e"
"\u{0301}"           ==> " ‌́"
"\u{65}\u{0301}"     ==> "é"
{% endhighlight %}

这个原则上来说不算是Emoji相关的元素，不过因为Unicode允许他们被apply到任何一个字符上，所以其中一些可以用来和emoji组合出很好玩的效果，比如

{% highlight swift %}
"\u{01F436}"         ==> "🐶"
"\u{20E0}"           ==> "‌⃠"
"\u{01F436}\u{20E0}" ==> "🐶⃠"
{% endhighlight %}

<img src="/assets/images/2016/08/emoji/1F436+20E0.png" class="emoji">...<img src="/assets/images/2016/08/emoji/1F914.png" class="emoji">

#### **Counting a String, a.k.a 4 + 1 = 4**

OK, 所以，现在是不是更能理解为什么现代语言都不期望你通过占用内存的大小来计算字符串里的字符数量了哈。

'The Swift Programming Language'中有一个关于the count of the characters in a string 的note：

> “Extended grapheme clusters can be composed of one or more Unicode scalars. This means that different characters—and different representations of the same character—can require different amounts of memory to store. Because of this, characters in Swift do not each take up the same amount of memory within a string’s representation. As a result, the number of characters in a string cannot be calculated without iterating through the string to determine its extended grapheme cluster boundaries. If you are working with particularly long string values, be aware that the characters property must iterate over the Unicode scalars in the entire string in order to determine the characters for that string.
>
> The count of the characters returned by the characters property is not always the same as the length property of an NSString that contains the same characters. The length of an NSString is based on the number of 16-bit code units within the string’s UTF-16 representation and not the number of Unicode extended grapheme clusters within the string.”
>
> Excerpt From: Apple Inc. “The Swift Programming Language (Swift 3 beta).” iBooks. [https://itun.es/us/k5SW7.l][TSPL]

在Playground里简单的测试一下

{% highlight swift %}
let cafe = "cafe"
let accent = "\u{0301}"

cafe.characters.count   // 4
accent.characters.count // 1

let café = cafe + accent
café.characters.count   // 4 + 1 = 4

{% endhighlight %}

当然，很奇怪的是，使用ZWJ组合的emoji被认为是多个character（虽然显示的时候只占用一个字符的位置）。不知道是不是因为用的是Xcode 8 beta版的缘故。

{% highlight swift %}
"👨‍❤️‍💋‍👨".characters.count   // 4
{% endhighlight %}

#### **还有很多...**

Unicode很复杂，其实还有很多很好玩的东西，比如emoji里的国旗，其实是用专表区域的字母拼写出来的该国家的代码。比如<img src="/assets/images/2016/08/emoji/CN.png" class="emoji">表示<img src="/assets/images/2016/08/emoji/1F1E8+1F1F4.png" class="emoji">，<img src="/assets/images/2016/08/emoji/US.png" class="emoji">表示<img src="/assets/images/2016/08/emoji/1F1FA+1F1F8.png" class="emoji">等等。

鉴于篇幅，这里就不详细介绍了，感兴趣的可以看文后的参考链接。其中[Unicode® Technical Report #51][2]是关于emoji的比较全面的资料。

<br />

* * *

<br />

Reference:

1. [Proposed Update UTR #51, Unicode Emoji (Version 4.0)][1]
2. [Unicode® Technical Report #51][2]
3. [Unicode® Emoji Charts v4.0 — Beta][3]
4. [Full Emoji Data, v4.0 — Beta][4]
5. [Text vs Emoji, v4.0 — Beta][5]
6. [Draft data files for UTR #51 Unicode Emoji, Version 4.0][6]
7. [Wikipedia: Unicode][7]
8. [Wikipedia: Combining character][8]
9. [The Swift Programming Language (Swift 3 beta)][TSPL]

[1]: http://blog.unicode.org/2016/08/proposed-update-utr-51-unicode-emoji.html "Proposed Update UTR #51, Unicode Emoji (Version 4.0)"
[2]: http://www.unicode.org/reports/tr51/tr51-8.html "Unicode® Technical Report #51"
[3]: http://www.unicode.org/emoji/charts-beta/index.html "Unicode® Emoji Charts v4.0 — Beta"
[4]: http://www.unicode.org/emoji/charts-beta/full-emoji-list.html "Full Emoji Data, v4.0 — Beta"
[5]: http://www.unicode.org/emoji/charts-beta/text-style.html "Text vs Emoji, v4.0 — Beta"
[6]: http://www.unicode.org/Public/emoji/4.0/ "draft data files for UTR #51 Unicode Emoji, Version 4.0"
[7]: https://en.wikipedia.org/wiki/Unicode "Unicode"
[8]: https://en.wikipedia.org/wiki/Combining_character "Combining character"
[TSPL]: https://itun.es/us/k5SW7.l "The Swift Programming Language (Swift 3 beta)"
