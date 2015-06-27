---
layout: post
title: "JSPatch初探"
date: 2015-06-18 11:56:50 +0800
comments: true
categories: iOS,JSPatch
---

---
1. JSPatch是什么？
2. Objective-C动态绑定机制
3. JSPatch原理
4. JSPatch实用举例
5. 与现有方案（Wax）的对比

---

> [JSPatch实现原理详解](http://blog.cnbang.net/tech/2808/)

一、JSPatch介绍
---
> 最直观的介绍莫过于官方文档

* [bang590/JSPatch](https://github.com/bang590/JSPatch)
* 利用OC Runtime桥接Objective-C和JavaScript
* 用处：
	- 添加模块
	- 修复线上Bug

二、OC动态绑定机制
---
> 一切iOS平台上实现动态BugFix和新增模块的技术都基于ObjC的动态绑定、动态加载、消息转发

* 所谓动态绑定就是利用OC Runtime可以做如下几件事情：
	- 给已有类增加方法
	- 替换现有类的方法指针，指向新的方法实现
	- 动态创建一个类
	
* 【类对象】动态绑定机制源于OC的动态类实现

			#import <objc/runtime.h>
			
	- 除基本类型外所有OC类都继承于NSObject类
	- NSObject对象有一个指向该实例所属类的isa指针
	- OC中的类也是一个对象的概念，因此我们可以动态的创建一个类对象
	- objc_allocateClassPair
	- class_addMethod
	- objc_registerClassPair
	
* 【消息】方法替换和新增源于消息转发机制
	- OC中方法调用起始是一个对象响应消息的过程
	- 可响应消息列表存在于对象isa指针所指向的类对象上
	- objc_msgSend
	- forwardInvocation:

三、JSPatch原理
---
> 通过forwardInvocation方法，将
	