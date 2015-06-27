---
layout: post
title: "Java统计程序运行时间（转）"
date: 2015-06-27 12:05:57 +0800
comments: true
categories: 
---

代码如下：
第一种是以毫秒为单位计算的。

	long startTime = System.currentTimeMillis();    //获取开始时间
	
	doSomething();    //测试的代码段
	
	long endTime = System.currentTimeMillis();    //获取结束时间
	
	System.out.println("程序运行时间：" + (endTime - startTime) + "ms");    //输出程序运行时间

第二种是以纳秒为单位计算的。

	long startTime=System.nanoTime();   //获取开始时间  
	
	doSomeThing(); //测试的代码段  
	
	long endTime=System.nanoTime(); //获取结束时间  
	
	System.out.println("程序运行时间： "+(endTime-startTime)+"ns"); 



