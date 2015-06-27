---
layout: post
title: "SIGABRT错误的调试办法"
date: 2015-06-27 10:50:39 +0800
comments: true
categories: iOS,Debug
---

iOS经常会遇到一个头疼的error就是在main函数上显示“ Thread 1: signal SIGABRT ”这个错误，终于在stackoverflow上找到了调试的办法：

 

[原文链接](http://stackoverflow.com/questions/9782621/i-have-an-error-in-main-m-thread-1-signal-sigabrt-how-can-i-fix-this)

 

重点就是 **Set an exception breakpoint.**

 之前我们遇到的在main函数上的那个崩溃信息，如果想要调试就加入一个 an exception breakpoint ，它会在exception 被 cathc 的时候停下来，这样就可以追踪到造成 exception 的代码了。 

加入一个exception breakpoint的方法就是：在navigator的断点页面，点击左下角的加号就能看到 exception breakpoint；

加入的时候可以设置，默认是 all，也可以选择针对 oc 还是 c 的断点。

 

>原文：
>
>When you get SIGABRT on that line of main, it means that your program is raising an exception. The stack trace shows where the exception is being caught, >not where it's being raised. Usually this is not helpful. To debug the problem, you can do two things:  
>
>1.  Click the “Continue Program Execution” button in the debugger control bar, or choose Program > Debug > Continue from the menu bar. This will let the program continue the exception-raising process. It will print a message to the debugger console that will help you understand what's wrong. (You may have >to continue execution a couple of times before it actually prints messages.) Read the messages carefully! They usually continue helpful information.    
>
>2. Set an exception breakpoint. This will make Xcode stop your program at the point where the exception is being raised, so you can see the code and the stack trace that is causing the problem.