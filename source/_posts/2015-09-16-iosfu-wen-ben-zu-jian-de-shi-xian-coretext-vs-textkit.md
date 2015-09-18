---
layout: post
title: "iOS富文本组件的实现-CoreText vs TextKit"
date: 2015-09-16 18:00:44 +0800
comments: true
categories: iOS, CoreText
---

#iOS富文本实现

&emsp;在iOS7之前，系统展现文字的方式只有UILabel、UITextView这样的简单控件，如果要实现复杂的富文本展现，则需要开发者自己调用CoreText去实现，开发的复杂程度非常大。

&emsp;iOS7之前最优秀的实现应该是开源项目：[Cocoanetics/DTCoreText](https://github.com/Cocoanetics/DTCoreText)，通过解析HTML和CSS文件最终用CoreText绘制。（UIWebView应该不会有人用吧 = .=）基于JavascriptCore应该也可以解析渲染出元素丰富的界面，但是没有HTML和CSS来的直观。

#DTCoreText
>有关这个项目bang神的blog写的很赞。

* [iOS富文本组件的实现—DTCoreText源码解析 数据篇 « bang’s blog](http://blog.cnbang.net/tech/2630/)
* [iOS富文本组件的实现—DTCoreText源码解析 渲染篇 « bang’s blog](http://blog.cnbang.net/tech/2729/) 
* 基于DTCoreText实现的富文本项目Demo演示

#TextKit
>iOS7上终于等来了TextKit，有关的介绍和吐槽都在这里：

* [Getting to Know TextKit · objc.io](https://www.objc.io/issues/5-ios7/getting-to-know-textkit/)

#总结
&emsp;综合来讲TextKit已经具备了富文本展现的一切必备功能：整段缩进、截断（加省略号）、连字符、对齐、文本样式、文本效果、图片视频附件，但就多媒体附件这一块似乎支持的还没有DTCoreText那么灵活，只能插入固定的类型而不是插入一个View。


