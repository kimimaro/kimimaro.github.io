---
layout: post
title: "异步机制"
date: 2015-06-08 00:02:06 +0800
comments: true
categories: iOS
---

>1. 及时响应用户
>2. 实现多核调度

<!--more-->

###NSOperation
1. 封装了大部分异步相关的逻辑，基础类，使用的时候要继承它创建我们自己的子类；
2. 系统实现的具体的类：NSInvocationOperation，NSBlockOperation；
3. Dependency，Completion block，KVO，Priority，Cancel
5. non-concurrent
	* 重写main()
	* autoreleasepool
	* 不需要自己维护状态
4. concurrent
	* 实现start()
	* 实现异步操作
	* 自己维护isFinished，isExcuting，isCancelled（manual KVO）
6. 判断是否cancel
	* main/start
	* 循环开始处
	* 逻辑区分点

###dispatch queue
1. 将业务逻辑写在block里
	* 定义：	
		
			^return type(params){expression}	
			typedef return type(^name)(params)
	* 声明：
	
			self.block = ^(params){expression};
	* 方法中定义的block在stack上，当调用copy时，会copy到heap上
2. finalizer
3. Serial(private dispatch queue)
	* block间顺序执行
	* dispatch_queue_create()
	* 需要自己维护reference count:
			
			dispatch_release()
4. Concurrent(global dispatch queue)
	* block间并发执行
	* dispatch_get_global_queue()
	* normal, low, high
	* 系统维护reference count
5. main dispatch queue
	* 在主线程run loop执行
	* 更新UI/主线程作为同步
6. dispatch queue维护自己的autoreleasepool

