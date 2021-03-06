---
layout: post
title: "Python 编码那些事"
excerpt: "简单介绍编码的问题以及python中的处理"
categories: 程序技术
tags: [python, encoding, hack]
comments: true
share: true
---

### 似曾相识？

如果你经常使用Python的话，对下面这种情况应该很熟悉：

    UnicodeDecodeError: 'ascii' codec can't decode byte 0xef in position 1: ordinal not in range(128)

你当然知道这是编码错误，也能根据提醒知道默认的`ASCII`码无法解析这里的数据，但是却不知道到底为什么会出现这种情况，并且应该怎么解决。那么这篇文章就是为你而准备的，请继续往下读。

### 认识编码

计算机为什么需要编码呢？因为：**计算机内存储的都是0和1构成的字串。** 计算机无法认识文字、图像、视频等任何人类看起来很容易的东西，它只理解0和1。这样，我们就需要把上述的内容转化成计算机能理解的东西，这就是编码的过程。

其实早在计算机出现以前，编码就已经无处不在了。结绳记事就是一种编码：把某个时间和事情用绳子的状态来变现；文字本身也是一种编码：把自然界的动物和名字一一对应；另外一个很著名而且和计算机相近的编码是`摩斯码`。

> 摩斯码用两种符号来对字母和数字进行编码：点和线。它是一种变长的编码方式，一般来说，使用频率较高的字母的长度较小。需要注意的是：空格在摩斯码中非常重要，在字母与字母之间，单词和单词之间都用空格来分割。

### 几种重要的编码

+ ASCII码

    英语只有26个字母，加上键盘上所有的符号`{~\|/?<>,.'";:[!,@,#,$,%,^,&,*,(,),_-,=,+]}`也不到一百个字符。即使加上计算机内部的控制字符，七个比特就足够了！但其实ASCII码是用8个字符表示的，因为计算机内部最小的存储单位是字节，这样读取速度会更方便和快捷，增加的一位也可以扩展更多的字符。

    ![ASCII Code]

    在上图中可以看出英文的可打印字符只占据从32到126的位置，所以在正则表达式中`[ -~]`能匹配ASCII 码所有的可打印字符。大体来说ASCII表可以分为三个部分：控制码、可打印码、特殊字符码。需要注意的是大小写字母并不是紧密排在一起的，中间被6个特殊字符隔开。

+ Unicode编码

    有了ASCII码，英文国家的人很满意，所有的问题都解决啦！但是世界上那么多国家，字符集远不止这么多。你可能会想：每个国家的人都解决自己的语言字符不就完事啦？情况没那么简单：不同国家／地区对不同的字符编码相同怎么办？比如`ASCII`码中的`A`是65，但是65在另外一个国家被用来编码`＊`怎么办？当然也会出现对同一些字符（比如数字）编码不同的情况，因为互联网是无国界的（在中国这一点不是很明确），这些问题就会造成很大的麻烦。

    一个简单的思想是：用一种编码把世界上所有人类发明的字符都编码出来。你或许觉得这很笨，但是这其实是最聪明的办法，这就是`Unicode`。准确来说Unicode不是一种编码方式，只是一种编码的标准：它把每个字符都对应成一个四字节（32比特）的数字，具体怎么存储还需要看具体的编码方法。比如，简体中文`人`的unicode值是`U+2F08`，但是怎么在计算机里存储它还需要编码方式。我们来看一下unicode可以存放多少编码吧：

        2^32 = 4,294,967,296

    四十多亿，好吧，够用的。这么多数字都需要人工来逐个编码，不用担心，世界上没那么多符号，最新版本的unicode也只有一百多万个符号的编码而已，这样工作量就小得不少。

+ UTF-32和UTF-16

    最直接的编码方式就是直接存储Unicode的四个字节，UTF-32就是这样做的。这样做的优点是简单，方便读取，但是会浪费很大的空间。虽然实际上存在的字符很多，但是经常用到的却没有那么多，比如中文的汉字有上万（四五万的样子），但是经常使用的还不到十分之一。对与UTF-32编码也是这种情况：像`☀`这样的特殊符号在Unicode中所占的比例很大，但是被使用的几率却很小；常用的英文字母和数字一直用32个字节来表示的话，确实是一种浪费。编码方式影响最大的程序是**网页**，它需要面对世界上任何角落的编码方式，如果都用UTF32存储的话，网络流量的浪费可想而知。

    所以就有了UTF-16编码，它用两个字节来编码常用的65536个字符，其他的字符用两个UTF-16来表示就行了。那么能否有更好的办法呢？

+ UTF-8编码

    如果你是完美主义者，也许会想到`huffman编码`：对每个字符出现的频率进行统计，然后据此得到最优化的huffman编码。不错的注意，但是基本上不可行。为什么呢？因为这么多字符统计是件很复杂的事情，而且不同国家和地区对字符的使用频率不同。

    现在最常用的编码方式为UTF-8编码。UTF-8编码是变长编码方式，就是说不同字符的编码长度是不同的。0-127之间的字符都是一个字节的编码，128以上的编码才会使用2，3甚至6个字节（没错，就是6）！它最大的特点就是：能够和ASCII码兼容。简单介绍一下，UTF-8是怎么读数据的（也怎么怎么判断下一个数据是几个字节长度的）。

```
        ＊ 0000 - 007F -> 0xxxxxxx
        ＊ 0080 - 07FF -> 110xxxxx 10xxxxxx
        ＊ 0800 - FFFF -> 1110xxxx 10xxxxxx 10xxxxxx
```

  上表是UTF-8的编码表，0-127只需要一个字节，128-2047需要两个字节，2048-65535需要四个字节。因为每个字节前几位用来判断字节数，当读入字节流的时候，只需要读入第一个字节，判断下面需要继续读入几个字节就行了。所以UTF-8不存在大小端的问题！

    下图是互联网上不同编码的比例，可以看出utf-8已经成为了最流行的编码方式。需要注意的是，GB2312编码的比例基本不变，这说明其实中国的大部分网站还是停留在GB2312编码上，而这种编码和UTF-8是不兼容的。

    ![UTF-8]

### Python中的编码

扯了那么多，终于要讲python啦：这些编码在python是怎么实现的？会出现什么问题？python又是怎么解决的呢？

python3.X对编码的改变很大，下面就先说说2.X版本的编码问题。（下面提到的python没有特殊说明均为2.X版本。）

- python 的默认编码方式为ASCII码，这就是文章的开头错误出现的一个重要原因：试图用ASCII编码来读取其他编码的值，超过128部分的将无法处理。
    你可以用`sys.setdefaultencoding('utf-8')`来改变默认编码。

- python编码原则
    + 趁早解码
    + 过程中使用unicode
    + 最后再编码

  怎么理解呢？因为计算机不能自动判断编码（它可以去猜，但是准确率不令人满意），必须人为地告诉它内容的编码方式。这样在刚使用文本内容的时候，就把它解码成unicode，然后在程序处理过程中一直如此，直到需要存储或者打印等的时候，才把unicode编码成需要的格式。这样的原则会减少很多不必要的错误！

### 参考资料

- [ASCII，Unicode，UTF-8，GB2312一些关于编码的理解](http://space.itpub.net/23071790/viewspace-704585)
- [字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)
- [The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!)](http://www.joelonsoftware.com/articles/Unicode.html)
- [Unicode In Python, Completely Demystified](http://farmdev.com/talks/unicode/)
- [Wikipedia: Morse Code](http://en.wikipedia.org/wiki/Morse_code)
- [Wikipedia: Unicode](http://en.wikipedia.org/wiki/Unicode)


[Morse Code]: http://pad2.whstatic.com/images/thumb/7/7d/International_Morse_Code_150.png/300px-International_Morse_Code_150.png
[ASCII Code]: http://people.revoledu.com/kardi/resources/Converter/Image/ASCII.jpg
[UTF-8]: http://www.w3.org/QA/2008/05/utf8-growth-google
