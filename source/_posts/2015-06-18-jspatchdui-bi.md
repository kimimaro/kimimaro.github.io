---
layout: post
title: "JSPatch与Wax对比分析"
date: 2015-06-18 19:19:06 +0800
comments: true
categories: iOS,JSPatch
---
> [JSPatch](https://github.com/bang590/JSPatch)与[Wax](https://github.com/probablycorey/wax)作为iOS平台上两大热更新框架使用不同的语言、基于相同的原理（ObjC动态绑定）一直备受iOS开发者的青睐，Wax出现较早且已经不再维护，但是也有了各种各样的分支版本；JSPatch作为后起之秀得益于iOS7苹果开放了JavascriptCore.framework的API，相比以前的Wax框架有了很多更新，下面就根据以往的开发经验简单比较一下二者的优势劣势，如果想对两个框架有更多的了解，请移步上文的官方网站和相关文档。

<!--more-->

1. JSPatch相比于Wax的优势
2. JSPatch相比于Wax的劣势

---

#JSPatch相比于Wax的优势

内存管理
---
* JSPatch中JSValue负责维护被引用OC对象的生命周期，如果JS有变量引用时，这个OC对象引用计数就加1 ，JS变量的引用释放了就减1，如果OC上没别的持有者，这个OC对象的生命周期就跟着JS走了，会在JS进行垃圾回收时释放。
* Wax在引用OC对象需要打上waxRetain标记，然后跑一个定时的GC查看这个对象的retainCount，当发现一个不再需要引用的对象retainCount大于1并且waxRetain为YES时（实际上的判断逻辑要复杂一些），就释放这个对象。依赖于retainCount，苹果官方是不推荐的，也不安全。

类型转换
---
* JSPatch使用的是系统提供的类型转换，JSValue类内部可以看到完整的JS类型与OC的映射关系，同时支持NSArray和NSDictionary等类型的嵌套解析;

        @textblock
           Objective-C type  |   JavaScript type
         --------------------+---------------------
                 nil         |     undefined
                NSNull       |        null
               NSString      |       string
               NSNumber      |   number, boolean
             NSDictionary    |   Object object
               NSArray       |    Array object
                NSDate       |     Date object
               NSBlock (1)   |   Function object (1)
                  id (2)     |   Wrapper object (2)
                Class (3)    | Constructor object (3)
        @/textblock

* Lua中调用OC对象需要使用toobjc方法，否则会Crash，还有一个问题是NSDictionary、NSArray使用时是被copy的，也就是说你无法直接更改OC内存中的那个对象。

多线程
---
* 由于JSCore的支持，JS脚本实现的方法在多线程中调用没有任何问题，同时JSPatch也针对GCD提供了封装；
* Lua语言本身是不支持多线程的，多个线程同时调用Lua就等于同时操作同一张Lua元表，可能出问题；同时Wax调用异步的话只能依赖performSelectorInBackground:了。

Block
---
* JSPatch：天然支持，JS的Function类型会对应转换成NSBlock执行；
* 个人认为Wax最受诟病的一点，无法支持Block使得方法替换和扩展都受到很大限制。

类型扩展
---
* JSPatch提供了CGRect、CGPoint、NSRange支持
* Lua调用OC对象时需要toobjc，NSDictionary、NSArray只支持copy，不支持retain

参数传递
---
> 为了替换方法实现，将方法传递到脚本语言的实现过程中必须知道当前方法的参数类型列表，传统的方法是用va_list得到，但是arm64上va_list的实现更改无法根据内存位置取出参数，对于这个坑也是困了好久才解决。但是个人认为bang牛给了一个更好的解法。

* JSPatch为了解决arm64上va_list拿不到参数的问题，使用forwardInvocation方式，支持各种类型参数枚举；
* Wax框架维护期间还没有出现arm64所以也就一直平静的使用va_list，当然现在升级的话也可以使用JSPatch的这种方式；

#JSPatch相比于Wax的劣势

消息转发
---
* JS语言不支持消息转发，对于一个对象如果不响应方法就直接崩溃了，所以bang牛用正则替换了方法调用，然后通过桥接方法 __c() 来实现；
* Lua解析后的方法、变量都放到元表中，支持消息转发，算是一个非常好的特性。

iOS6支持
---
* JSPatch基于JSCore，仅支持iOS7+
* Wax理论上支持所有iOS版本。
