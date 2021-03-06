---
layout:     post
title:      清除浮动的几种方法
subtitle:   感觉现阶段清除浮动的方式主要利用的是 `clear:both` 和 `overflow:auto||hidden||scroll` 这两个css属性，只不过利用上述两者的实现方式有所不同而已。
date:       2015-04-16
author:     elson
header-img: 
header-bg-color: 337ab7
catalog: true
tags:
    - CSS
---

### **利用 `clear:both` 清除浮动**

**1.`clear:both` + 多余的标签**

	`<div class="clear"></div>`
将以上div放在浮动元素父级的内部
``` css
.clear1 {
	display:block; /* 对内联元素使用clear无效 */
	clear:both;
	/* 以下属性估计是为了避免标签中有文本或图片内容显示出来而造成影响 */
	line-height:0;
	font-size:0;
	height:0;
	overflow:hidden;
}
```
**缺点**：要添加多余的无意义标签

**2.`clear:both` + 伪类**

同样将此类名添加在浮动元素的父级身上
与`1.`相比，是将伪类去替代`div`，感觉本质还是没变
``` css
.Clear2:after{
	content:'.';
	display:block;
	height:0;
	clear:both;
	overflow:hidden;
	visibility:hidden;
}
/*伪类在IE67中无法识别*/
/*IE67是因为用了zoom:1触发hasLayout才清的浮动*/
.Clear2 {zoom:1;}
```
**ps.** 如果IE67下，浮动元素父级具有width值(非auto)，是不需要清除浮动的。因为width已经触发了haslayout。

### **利用 `overflow` 清除浮动**
其实第一次看到可以用 `overflow` 清除浮动(其实我是拒绝的)，有一种很神奇的感觉，为什么这样可以清除浮动？！而这样的问题，也作为面试题被问过。

而其原因在于，`overflow(非visible值)` 可以触发 BFC(Block Formatting Context) 或者是 IE67中的 `hasLayout`，使之改变了排版的方式。

那问题又来了，什么是BFC，什么是 `hasLayout`?

关于 `hasLayout` 请戳[这里](http://riny.net/2013/haslayout/)，解释的蛮清晰的。

>####**什么是BFC**
>BFC（Block Formatting Context）直译为“块级格式化范围”，是 W3C CSS 2.1 规范中的一个概念，它**决定了元素如何对其内容进行定位，以及与其他元素的关系和相互作用。**
>####**BFC的几大用处**
>1. 防止margin折叠
>2. 清除浮动
>3. 不会环绕浮动元素
>####**BFC的特点**
>1. 形成独立的空间，对内部元素负责，隔离内部元素对外界的影响。
>2. 自身对外界表现正常
>3. 不会覆盖float元素，并且自适应的占据这一行剩下的宽度
>####**如何触发BFC**
>
* 使用 float，并且值不为 none
* 使用 absolute 定位的元素
* **使用 overflow,并且值不为 visible**
* 使用 display: table-cell / table-caption / inline-block
* 使用 position, 并且值不为 static / relative

>**资料**
>[Block Formatting Context In CSS](http://outofmemory.cn/wr/?u=http%3A%2F%2Fkkeys.me%2Fpost%2F68547473290)
>[关于Block Formatting Context](http://www.cnblogs.com/pigtail/archive/2013/01/23/2871627.html)

 以上资料说明，只要能触发 BFC 或者 `hasLayout` 的css属性均可以清除浮动，而 `overflow` 被广泛使用的原因，我想应该在于，在触发 BFC 或者 `hasLayout` 的同时，对元素自身的定位或者表现影响有限。

下面是对使用 `overflow` 的几个属性值来清除浮动时，它们之间的差异性。

参考：[http://www.quirksmode.org/css/clearing.html](http://www.quirksmode.org/css/clearing.html)
``` css
.container{
	border: 1px solid #ccc;
	overflow: auto;
	width: 100%;
}
```

以上使用的是`overflow:auto`，下面几点值得我们注意：

**1.** 使用除了 `overflow` 的默认值 `visible` 以外的值`auto` `hidden` `scroll` 均可清除浮动。当然，使用 `scroll` 的话滚动条是会一直显示的。

**2.**  在使用 `auto` 或者 `hidden` 时，需要保证容器的高度为自适应(即不显式定义height)；此外浮动元素的总宽度应该始终小于容器的宽度。否则，在清除了浮动的同时会带来另外的问题：超出容器部分的内容会被“切”掉，或者出现滚动条。

**3.** 在Explorer Mac中，设置 `auto` 会始终显示滚动条。（不懂mac 没测过）

**4.** 对于IE6，设置 `overflow` 并不能触发 `hasLayout`, （IE7可以！）因此需要设置其他属性，如`zoom:1` `width: 100%` 等。


### **以下是其他可以清除浮动的方法，但有很大局限性或兼容问题，因此不常用**

1. 让浮动元素的父级也跟着浮动起来，`float:left` or `float:right`

2. 为浮动元素的父级添加`display:inline-block`
3. 为浮动元素的父级添加`position:absolute`

 **不难看出，以上方法的目的都是为了触发BFC或者 `hasLayout`。**


### **总结**
从各种书籍和文章看来，清除浮动主要是从以下两种思路入手：

1. 利用 `clear` 属性
2. 触发BFC 或者 `hasLayout`

以上是对最近看到的有关清除浮动的资料，所做的思考总结。如理解有误，有望指出~
