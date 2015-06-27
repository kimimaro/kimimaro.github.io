---
layout: post
title: "NSURLConnection的同步与异步"
date: 2015-06-27 12:15:00 +0800
comments: true
categories: iOS
---

从这篇文章中收益良多：http://www.cocoabyss.com/foundation/nsurlconnection-synchronous-asynchronous/

写下体会：（时间所限，比较杂乱，见谅！）

***
NSURLConnection的同步发送就不用说了，很简单，上面链接也说了，

异步在iOS5之前都是作为一个非正式的协议出现的，默认initWithRequestxxx方法发送的就是异步请求，然后通过delegate中的方法可以操作，之前我一直错误在发送异步请求的时候也开一个新的线程，看了文章之后才想明白这真的是没有必要，在其他线程跑conn的原因就是为了避免阻塞主UI线程，而现在cocoa使用了delegate帮助你避免了这个问题，所以根本不需要再开一个新的线程去请求，直接请求然后在delegate中handleData就可以了。

而回到对同步conn的使用，同步conn使用我认为是一定要在新线程中，这样避免了阻塞主线程的缺点（如文中所说），但是相比与aSync conn，sync还有其他缺点，如不能在请求同时操作数据，不能cancel等等，所以还是aSync要好用。

***

iOS4之前如果想在新的线程使用异步发送的conn，最好的方法应该就是文中教的trick，而

iOS5之后添加了 NSURLConnectionDataDelegate 把之前非正式协议的一部分拿了过来变成了正是协议（formal protocol），里面包括了数据处理等代理方法，还增加了在线程中发送异步请求的方法：

	+ (void)sendAsynchronousRequest:(NSURLRequest *)request
	
	                          queue:(NSOperationQueue*) queue
	
	              completionHandler:(void (^)(NSURLResponse*, NSData*, NSError*)) handler NS_AVAILABLE(10_7, 5_0);

这样我认为就不需要像上文作者那样trick了吧？

***)
另外还加入了NSURLConnectionDownloadDelegate，主要作用就是讲数据直接存入文件中，缓存的时候估计有用吧？