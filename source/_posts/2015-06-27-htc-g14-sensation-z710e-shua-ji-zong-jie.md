---
layout: post
title: "HTC G14 Sensation Z710e 刷机总结"
date: 2012-06-27 12:20:35 +0800
comments: true
categories: Android
---


>几年前写的可能过时了，留个底。

>在下面一大堆废话之前先说有用的，推荐 RomManager 这个应用，只要你的手机已经获得了root权限，这个app能够让你 随心所欲的，方便的更新 recovery，下载国内国外好的ROM，同时也提供了 备份和还原 功能。有免费版和付费版两个版本。 有经验的朋友应该已经知道这款应用，大神级的app。

<!--more-->

&emsp;这不是一篇教程，但是也许会让你对刷机有一些整体的体会； 阅读这篇文章的前提是你已经有了一些刷机的经验，了解大体的步骤。推荐先了解几个概念： ROM，recovery，root，S-ON (SHIP S-OFF, END S-OFF)； 如果初次接触的话推荐 小米 的刷机教程（最新的 android 4.0 系统的 sensation 的教程），省去很多时间； 网上的刷机教程都太繁琐，只讲了流程，但是没有逻辑性，按照它的流程走通是没有问题，但是如果一旦遇到了与之流程不符的情况，就不知道如何处理了。 我写这篇教程只讲几个比较重要的软件和值得注意的点，希望能对遇到莫名其妙问题的同学有所助益。 刷机并不需要一个完整的傻瓜式教程，而是要理解其中的几个点，刷机主要有哪几个步骤，用到了哪些服务（或者说软件），它们都是做什么的，这样就ok了。 这些服务的官网上都有完整的说明，把这些说明看懂你才真正知道你在做什么，为什么要这么做 （不过这些网站都是国外的，所以如果英文不好，或者你根本就不care这些只care刷上了一个新系统的话，忽略我这篇文章） 由于时间有限所以有些服务没有链接，建议直接去官方网站上下载，顺便看看介绍。

* 两个快捷键 + 一个操作：
    音量下 + 电源 ：进入 HBOOT 模式，要进行 fastboot，recovery等操作都要在这个模式下；
    音量上 + 音量下 + 电源 ：硬重启，刷机会遇到这样那样的问题，偶尔会用到这个；
    拔电池，主要是为了确认真正关机了，等到你刷上的时候就会发现这个操作很有用，如果你没用到，我只能承认你确实很厉害。

* 其实刷机无非就几个步骤：
    获得 ROOT 权限；
    Recovery；
    Rom；

* 会用到这样一些服务，基本上都是国外的服务：
    - Revolutionary

        用来解锁的服务，官网上有很好的解释，如果希望简单了解具体的操作步骤参见其他教程（eg. http://bbs.anshouji.com/forum.php?mod=viewthread&tid=38512&extra=#pid582875）

    - ClockWorkMod

        Recovery 提供商，同时也是 RomManager 的作者。

    - ROM 可以就去网上找把，小米的，点心的，或者刚才提到的 ClockWorkMod 上面更多。
    - sensation 的解锁还要借助一个临时获得 ROOT 权限的软件： tacoRoot

* htc默认是不允许进行 recovery 操作的，所以需要解锁。

* 解锁，获取 ROOT 权限
    htc的手机需要先解锁，用到的工具就是 Revolutionary ，但是 sensation 的解锁还要麻烦一步，
    因为直接 Revolutionary 解锁会遇到 Zerging root... 然后直接就 failed 了。
    所以需要借助临时获得 ROOT 的软件： tacoRoot （具体的操作用adb，见：http://www.androidpolice.com/2011/12/30/exclusive-tacoroot- by-justin-case-and-reid-holland-a-new-temporary-root-exploit-for-all- htc-devices/）
    得到临时 ROOT 权限之后按照 Revolutionary 的操作步骤就可以把 S-ON 改为 S-OFF，但这只是一个 SHIP S-OFF，进一步可以改为 END-OFF，这里不做介绍。

* recovery
    ClockWorkMod 下载的 recovery 是 img 的。自己做成 PG58IMG.zip ，放在 SD 卡根目录，重启的时候 htc 会自动识别
    网上有很多人做了相应的 zip，搜 PG58IMG.zip 能找到一堆，还是推荐 小米 的。
    放在 SD 卡根目录然后重启，安装
    装好后记得 PG58IMG 要删掉否则每次 htc 都会提醒你。

**现在你可以进入 recovery 模式了**

* ROM
    下好的 ROM （.zip 的包，名字不限，位置不限）
    进入 recovery 模式，install 这个包

>再次声明，我写的不是教程！具体方法还是看 小米 的操作，但是这回你明白了 PG58IMG.zip 到底是什么东西，你为什么要这么做。 其实就是给手机刷一个新的 ROM，之前做的 解锁，root，recovery 都是因为 htc sensation 本身对手机做了一些限制。 顺便提一下我的htc刚开始刷了一个网上下的 PG58IMG.zip（recovery），结果导致无法关机，只有拔电池才有效，很烦；