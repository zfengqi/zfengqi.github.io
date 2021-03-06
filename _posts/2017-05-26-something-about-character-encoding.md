---
layout:     post
title:      字符编码的那些事
subtitle:   字符编码相关知识的梳理和总结
date:       2017-05-26
author:     elson
header-img:
header-bg-color: 337ab7
catalog: true
tags:
    - 字符编码
    - 计算机基础
---

## 前言
之前看到ES6中对String扩展了不少新特性，字符串操作更加友好，比如"\u{1f914}"，codePointAt()，String.fromCodePoint()。其中涉及到不少字符编码的知识，为了更好理解这些新特性，本文对字符编码相关知识做一个较全面的梳理和总结。

以下内容包括：字符集和字符编码的关系以及编码规则，JS的字符编码，HTML的转义序列。

首先，在前言里面回顾一下位与字节(小b和大B)的最最基础知识。

- 1bit = 1个二进制位 = 0 或 1


- 8bit = 8个0或1（2^8=256个组合）= 1字节Byte

值得一提，在计算带宽大小(bps)的时候要注意是以bit作为单位。




## 一、字符集与字符编码

Unicode、ASCII、GB2312、GBK、BIG5都属于是**字符集(character set)**，每个字符集包含的字符种类和数量都不一样，每个字符有各自的编号作为唯一标识。

那么它们是通过什么方式进行编号(以下都称为**码点**)的呢？不同的字符集有不同的方案，对于ASCII、GB2312、GBK、BIG5来说，实行“垄断”政策，即只允许使用它规定的编码方案，也可以认为它即是字符集也是字符编码。而Unicode实行“百家争鸣”政策，提供了UTF-8/UTF-16/UTF-32几种备选的字符编码方案，所以这时Unicode仅仅是字符集，UTF-X才是**字符编码**。

> 各个字符集的具体编码方案可以看[这里](http://www.cnblogs.com/skynet/archive/2011/05/03/2035105.html)

正因为这个原因，经常会听到说ASCII编码、GB2312编码，甚至Unicode编码，这种叫法很容易混淆字符集和字符编码的关系。

弄清楚字符集与字符编码的关系之后，我们可以知道如果某个字符想从UTF-8编码转成GBK编码的话，那就必须先将其unicode码点换算成GBK码点，再进行GBK编码。

下面我们主要看看ASCII和Unicode这两种字符集(编码)。



## 二、ASCII字符集及编码

ASCII是最古老原始的字符集和编码，主要是满足英语字符的需要，毕竟计算机是从人家老美那诞生的。

每个字符用一个字节(8bit)来储存，一共定义了128个字符，前32个字符是非打印控制字符(回车换行等)。虽然一个字节最多可以定义256种字符，但是ASCII只用了1个字节的后面7位，最前面统一都为0。

- 空格"SPACE"码点：十进制32，十六进制20，二进制00100000
- 大写的字母A码点：十进制65，十六进制41，二进制01000001


### Extended ASCII

ASCII只有128个字符，其他语言不够用了，怎么办？

别忘了ASCII只用了后面7位，利用空闲的最高位，这样可以扩容到256个字符，成为**扩展ASCII码(EASCII)**。所以0-127码点表示的字符是一样的，不一样的只是128-255码段。

由于只能多扩容128个字符，而各国语言中的字符又各不相同，为了满足不同地区的更多字符的需求，所以扩容字符的含义不可能都一样。这里就会出现如ASCII码表“阿拉伯字符(ASMO-708)码”扩展ASCII，“泰语(Windows)码”扩展ASCII。

而对于中文而言，1个字节256个字符显然不够，因此中文只能单独制定如GB2312、GBK、GB18030、BIG5字符集了。关于GBXXX编码可以看[这里](http://www.cnblogs.com/skynet/archive/2011/05/03/2035105.html)。

> [ASCII码表](http://www.haomeili.net/Code/ASCII?key=windows-874)

看到这里，有种贵圈真乱的感觉，各国都自行一套字符集及编码，这就不利于沟通交流阿。直到Unicode出现。



## 三、Unicode

Unicode解决了各国自行一套的问题，将世界上所有的符号都纳入其中。它符提供了唯一码点，不论是什么平台、不论是什么程序、不论是什么语言。

- **码点code point**范围从 0x0 - 0x10FFFF，共分为17个Plane，每个Plane中有65536个字符，共可容纳：` 17*(16*16*16*16)= 1114112 `个字符。
- 第一个平面称为**基本多语言平面（Basic Multilingual Plane, BMP）**。其他平面称为**辅助平面（Supplementary Planes, SP）**，或astral Plane。
- <u>BMP内，从U+D800到U+DFFF之间的码位区块是永久保留不映射到Unicode字符</u>。后面介绍的UTF-16就利用保留下来的0xD800-0xDFFF区段的码位来对辅助平面的字符的码位进行编码。

前面说到Unicode只是字符集，具体码点怎么储存，可选择UTF-8、UTF-16或者UTF-32编码方式。

> 这里插播一个名词：**code units 码元**
>
> 码元是各编码方式的基本单位，长度单位是bit。一个码点有可能只需要一个码元，也有可能需要多个码元。
>
> UTF-x等编码方式中的数字其实就规定了此编码方式下的码元长度。如UTF-8的码元长度为8bit.......
>
> 当一个码点太大，一码元长度没法储存时，这时就需要其分解成两个或以上码元来储存。
>
> 如**0x10437**码点UTF-16会分解成**D801 DC37**两个码元（每个码元16bit），UTF-8会分解成f0 90 90 b7四个码元（每个码元8bit）

> [中日韩汉字unicode编码表](http://www.chi2ko.com/tool/CJK.htm)
>
> [Unicode code converter](https://r12a.github.io/apps/conversion/)

### 1. UTF-8

1. 是在互联网上使用最广的一种Unicode的实现方式，
2. 是一种变长的编码方式。可以使用1~4个字节存储一个字符，根据不同的符号而变化字节长度。

#### UTF-8的编码规则

| Unicode码点范围               | UTF-8编码方式（二进制）                       | UTF-8编码所需大小 | 规则                              | 备注                                    |
| ------------------------- | ------------------------------------ | ----------- | ------------------------------- | ------------------------------------- |
| U+0000 0000 - U+0000 007F | 0xxxxxxx                             | 1byte       | 字节的第一位设为0；后面7位为这个符号的unicode码。   | 英语字母，UTF-8编码和ASCII码是相同的               |
| U+0000 0080 - U+0000 07FF | 110xxxxx  10xxxxxx                   | 2byte       | 第一个字节前两位是1，第三位是0；后面字节的前两位一律设为10 |                                       |
| U+0000 0800 - U+0000 FFFF | 1110xxxx  10xxxxxx 10xxxxxx          | 3byte       | 第一个字节前三位是1，第四位是0；后面字节的前两位一律设为10 | 汉字(U+4E00 - U+9FA5)在UTF-8里的编码都是 3 个字节 |
| U+0001 0000 - U+0010 FFFF | 11110xxx  10xxxxxx 10xxxxxx 10xxxxxx | 4byte       | 第一个字节前四位是1，第五位是0；后面字节的前两位一律设为10 |                                       |

![UTF8编码例子](https://ws1.sinaimg.cn/large/006tNc79gy1fpvbxjpf4oj30m8032aa5.jpg)

### 2. UTF-16

#### 2个或4个字节存储一个字符

- 2字节：从0x0 - 0xFFFF的码段(BMP)，编码后的数值和unicode对应的码点一致
- 4字节(两个双字节)：从0x10000 - 0x10FFFF的码点(SP，已经超过了BMP平面)，会根据规则，编码成一对16bit长的码元：如**0x10437**码点会编码成**D801 DC37**，它们叫做代理对(surrogate pair)

#### 4字节代理对的原理

从上面D801 DC37的例子可以发现，这两个码点都落在了BMP平面的码点范围之内，<u>并且都属于U+D800到U+DFFF码段</u>。没错，就是通过一系列规则，把超出BMP平面的码点(U+10437)转换成两个属于BMP平面的码点——U+D800到U+DFFF码段之间(U+D801和U+DC37)。

##### 大致解说

1. SP平面中的码点范围是从U+10000到U+10FFFF，共计FFFFF个，即2^20=1,048,576个，需要20位来表示。
2. 如果用两个双字节长的码点组成的序列来表示，第一个码点（称为高位代理）要容纳上述20位的前10位，第二个码点（称为低位代理）容纳上述20位的后10位。前后分别需要2^10=1024个码点来代理。而BMP平面的U+D800到U+DFFF码段正好有2048个码点，足以满足高位代理与低位代理的需要。
3. 因此需要将U+D800 - U+DFFF分为两段
 - 一段为高位代理初始值U+D800：U+D800 到 U+DBFF 之间的保留码点用于高位代理（leading  surrogates）
 - 一段为低位代理初始值U+DC00：U+DC00 到 U+DFFF 之间的保留码点则用于低位代理（trailing  surrogates）
 - 两段之间间隔2^10=1024，刚好各自能够满足前后10位

##### 例如U+10437编码（?）

- 0x10437<u>减去0x10000</u>，结果为0x00437，二进制为0000 0000 0100 0011 0111
- 分区它的上10位值和下10位值（使用二进制）:**0000000001** and **0000110111**
- 添加0xD800到上值，以形成**高位**：0xD800 + 0x0001 = 0xD801。
- 添加0xDC00到下值，以形成**低位**：0xDC00 + 0x0037 = 0xDC37。
- 得出它的UTF-16编码为**D801 DC37**

##### js实现

``` javascript
function findSurrogatesPair(codePoint) {
	var offset = codePoint - 0x10000;
	var lead = 0xd800 + (offset >> 10);
	var tail = 0xdc00 + (offset & 0x3ff); // 和1023 位与 把前十位都置为0

	return [lead.toString(16) + tail.toString(16)];
}

// example
var str = "?";
var cp = str.codePointAt(0);
utf16Encode(cp); // return ["d83e", "dd14"]
console.log('\ud83e\udd14'); // ?
```



#### UTF-16是UCS-2的父集

在没有辅助平面字符前，UTF-16与UCS-2所指的是同一的意思。但当引入辅助平面字符后，就称为UTF-16了。也就是说，UCS-2编码不能支持在UTF-16中超过2字节的字集。



## 四、JS字符编码
阮老师的ES6教程[字符串的扩展](http://es6.ruanyifeng.com/#docs/string)里面的第一小节**字符的unicode表示法**中提到：

> ......
>
> 有了这种表示法之后，JavaScript 共有6种方法可以表示一个字符。
>
> ```
> '\z' === 'z'  // true
> '\172' === 'z' // true
> '\x7A' === 'z' // true
> '\u007A' === 'z' // true
> '\u{7A}' === 'z' // true
> ```

这里面的`\u007A`看起来JS好像是UTF-16编码，但是平时加载一个JS时指定的charset又好像是UTF-8或者GBK？那JS到底是以什么来编码的？

这个问题我一直都有点懵逼，但实际上对于JS的编码问题应该分成两个不同的部分看待：

1. 内部：JS引擎是如何解析的？
2. 外部：浏览器是以什么编码来解析JS脚本的？

### 1. 内部：JS引擎解析源码

引擎会把所有源码当做是一连串的UTF16码元，也就是内部是以UTF-16进行编码的。

``` javascript
var f\u006F\u006F = 123;
console.log(foo); // 123
console.log(\u0066\u006F\u006F); // 123
var foo = '12\u0033'; // 123

// 中文
var 腾 = '123';
console.log(\u817e); // 123

// 4字节字符
var bar = '?';
console.log('\uD842\uDFB7'); // '?'

```

上面的例子可以看到，无论是字符串还是变量，无论是BMP还是SP上的字符，都可以使用UTF-16码元来表示。

那ES6中的大括号表示法呢？看起来并不需要UTF-16编码，直接用大括号包裹码点就好了。

```javascript
'\u{20BB7}' === '\uD842\uDFB7' // 竟然全等
```

但实际上只是语法糖，而这个语法糖很赞，ES6内部对大括号内的码点进行了UTF-16编码，不需要自己换算成代理对。

此外，上面还有三种表示法看起来怪怪的

``` javascript
'\z' === 'z'  // true
'\172' === 'z' // true 八进制
'\x7A' === 'z' // true 十六进制
```

1. `\z`实际上是用于转义特殊字符，如\r \n \t \" \'等，而`\z`这种非特殊含义字符则等于它本身
2. 八进制表示法，反斜杠后的取值范围是0-377（十进制的0-255），官方说法是用来表示Latin-1编码字符
3. 十六进制表示法，取值范围是00-FF，和上面的八进制表示目的是一样的

#### 迷之String.prototype.length

其实了解了上面的知识以后，对于字符串的length就不难理解了。

对于JS引擎来说，所有的字符串都是一系列的UTF-16码元，length指的是码元的个数（也可以理解为两个字节等于1个length），而不是字符个数。当某个字符是4个字节的UTF-16编码时，这时一个字符的length就为2。但是中文的length却始终为1，这是因为中文的码点范围U+4E00 - U+9FA5，都在BMP平面内，UTF-16编码只需要2个字节。

来看例子~

``` javascript
// 还是拿这个字
'?' === '\uD842\uDFB7';
'\uD842\uDFB7' === '\u{20BB7}';
var foo = '?';
var bar = '\uD842\uDFB7';
var baz = '\u{20BB7}';
console.log(foo.length, bar.length, baz.length); // 2 2 2
```

**而我们需要获取“正确”的length值该怎么办？**

ES6轻松解决：`Array.from(str).length`

不是ES6也不要灰心，只要识别两个相邻的码元是否形成代理对的关系（原理在上面有讲~），是的话把它们视为一个整体。

``` javascript
function getRealLen(str) {
	var reg = /[\ud800-\udbff][\udc00-\udfff]/g; // /[高位代理][低位代理]/g
	return str.replace(reg, 'i').length;
}

getRealLen('?'); // 1
getRealLen('?????'); // 5
```

### 2. 外部：浏览器解析JS脚本

我们可以以不同的编码方式来保存源码，但如果浏览器解码方案和源码保存时的编码方案不同，就会导致乱码。

当浏览器在加载一个`<script>`时，是通过以下优先级来确定其编码方式：

1. 如果文件开头有BOM(byte order mark)，那么它肯定是UTF编码的其中之一，而又因为不同编码下的BOM不一样，所以可以从BOM来确定编码方式
2. 通过响应头Content-Type: application/javascript; charset=utf-8
3. `<script>`标签中的charset属性
4. 页面中的<meta charset="UTF-8">

**不同编码下的BOM**

| BOM（U+FEFF） | 编码方式|
| ----------- | --------------------- |
| 00 00 FE FF | UTF-32, big-endian    |
| FF FE 00 00 | UTF-32, little-endian |
| FE FF       | UTF-16, big-endian    |
| FF FE       | UTF-16, little-endian |
| EF BB BF    | UTF-8                 |


## 五、HTML的转义字符

最后来总结一下前端经常能看到如`&nbsp;`,`&#x4e2d;`这样的转义字符。它们是HTML、XML等 SGML 类语言的转义序列（escape sequence），浏览器遇到他们会自动渲染成文字。

它一共有三种表达方式：

1. **&#dddd;**  &#后接十进制数——称之为实体编号
2. **&#xhhhh;** &#x后接十六进制数——称之为实体编号
3. **&name;**  &后接预先定义好的实体名称——称之为实体名称

其中前两种方式的数字**取值为目标字符的unicode码点**，与文档编码无关。

> [HTML特殊转义字符列表](http://zqdevres.qiniucdn.com/data/20120726145112/index.html)

| 字符实体 | 实体编号                                    | 实体名称    |
| ---- | --------------------------------------- | ------- |
| 中国   | `&#x4e2d;&#x56fd;` 或 `&#20013;&#22269;` | 无       |
| &    | `&#38;`                                 | `&amp;` |

### 如何在JS对这些转义字符进行解析？

对于实体编号，可以使用ES6中的String.fromCodePoint(codePoint)

``` javascript
String.fromCodePoint(20013); // 中
String.fromCodePoint(0x4e2d); // 中
```

或者借助dom

``` javascript
var $dom = $('<div>&#x4e2d;&#x56fd;</div>');
$dom.html(); // 中国
```



# 总结

自此，对字符编码有了一个较为全面的了解，这时回过头去看那些String新特性时，更容易理解了，印象深刻了不少。本来还有一些关于emoji表情的内容，还是后面另起一篇再说吧。

## 参考资料

- [字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)

- [字符集和字符编码(Charset & Encoding)](http://www.cnblogs.com/skynet/archive/2011/05/03/2035105.html)

- [JavaScript特殊字符]( https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types#String_literals)

- [Unicode and JavaScript](http://speakingjs.com/es5/ch24.html)

- [ES6入门-字符串的扩展](http://es6.ruanyifeng.com/#docs/string)

- [UTF-8 UTF-16 UTF-32 & BOM](http://unicode.org/faq/utf_bom.html)
