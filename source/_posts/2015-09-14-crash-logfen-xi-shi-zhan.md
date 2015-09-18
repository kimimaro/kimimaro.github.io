---
layout: post
title: "iOS Crash Log分析实战"
date: 2015-09-14 15:56:22 +0800
comments: true
categories: iOS
---

>事情的缘起是收到了一个用户的线上Crash反馈，QA无法复现，但是作为体量大开发质量又追求极致的应用通常还得解决这样的问题。
>
>通过各种努力终于拿到了用户的Crash Log，结果符号化后的崩溃线程堆栈里面竟然一条有用的信息都找不到，WTF！
<!--more-->
崩溃日志如下：（出于公司隐私考虑去掉了部分符号）

	{"name":"XXXApp","bug_type":"109","os_version":"iPhone OS 7.1.1 (11D201)","bundleID":"com.xxx","version":"6.7.1.0 (6.7.1)","app_name":"XXXApp"}
	Incident Identifier: EC2DC883-C0DB-4FD6-81FB-237626598366
	CrashReporter Key:   d5c091bc35732df06fcaf41e5786833159b04ea5
	Hardware Model:      iPhone5,2
	Process:             XXXApp [1912]
	Path:                /var/mobile/Applications/4E3E131C-5847-4B61-A0E0-D350830C1693/BaiduBoxApp.app/XXXApp
	Identifier:          com.xxx
	Version:             6.7.1.0 (6.7.1)
	Code Type:           ARM (Native)
	Parent Process:      launchd [1]
	
	Date/Time:           2015-08-28 00:37:32.658 +0800
	OS Version:          iOS 7.1.1 (11D201)
	Report Version:      104
	
	Exception Type:  EXC_CRASH (SIGSEGV)
	Exception Codes: 0x0000000000000000, 0x0000000000000000
	Triggered by Thread:  0
	
	Thread 0 Crashed:
	0   libsystem_kernel.dylib        	0x39c636d8 __kill + 8
	1   Foundation                    	0x2f901c1e __NSThreadPerformPerform + 382
	2   CoreFoundation                	0x2eee3fec __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 12
	3   CoreFoundation                	0x2eee34b2 __CFRunLoopDoSources0 + 202
	4   CoreFoundation                	0x2eee1ca2 __CFRunLoopRun + 626
	5   CoreFoundation                	0x2ee4c764 CFRunLoopRunSpecific + 520
	6   CoreFoundation                	0x2ee4c546 CFRunLoopRunInMode + 102
	7   GraphicsServices              	0x33db96ce GSEventRunModal + 134
	8   UIKit                         	0x317ab88c UIApplicationMain + 1132
	9   BaiduBoxApp                   	0x0006d13e main (main.m:17)
	10  libdyld.dylib                 	0x39badab4 start + 0
	
	Thread 1:
	0   libsystem_kernel.dylib        	0x39c51804 kevent64 + 24
	1   libdispatch.dylib             	0x39ba0050 _dispatch_mgr_invoke + 228
	2   libdispatch.dylib             	0x39b9a2de _dispatch_mgr_thread + 34
	
	Thread 2 name:  com.apple.NSURLConnectionLoader
	Thread 2:
	0   libsystem_kernel.dylib        	0x39c51a50 mach_msg_trap + 20
	1   libsystem_kernel.dylib        	0x39c51848 mach_msg + 36
	2   CoreFoundation                	0x2eee3624 __CFRunLoopServiceMachPort + 152
	3   CoreFoundation                	0x2eee1d44 __CFRunLoopRun + 788
	4   CoreFoundation                	0x2ee4c764 CFRunLoopRunSpecific + 520
	5   CoreFoundation                	0x2ee4c546 CFRunLoopRunInMode + 102
	6   Foundation                    	0x2f88c23c +[NSURLConnection(Loader) _resourceLoadLoop:] + 316
	7   Foundation                    	0x2f901a0a __NSThread__main__ + 1058
	8   libsystem_pthread.dylib       	0x39ccb956 _pthread_body + 138
	9   libsystem_pthread.dylib       	0x39ccb8c6 _pthread_start + 98
	10  libsystem_pthread.dylib       	0x39cc9ae4 thread_start + 4
	
	Thread 3 name:  com.apple.CFSocket.private
	Thread 3:
	0   libsystem_kernel.dylib        	0x39c64434 __select + 20
	1   CoreFoundation                	0x2eee751e __CFSocketManager + 482
	2   libsystem_pthread.dylib       	0x39ccb956 _pthread_body + 138
	3   libsystem_pthread.dylib       	0x39ccb8c6 _pthread_start + 98
	4   libsystem_pthread.dylib       	0x39cc9ae4 thread_start + 4
	
	...

>于是只好把各种exception type和C signal分析了一遍，希望能找到一些蛛丝马迹。

#Crash Log分析实例

###1. 首先还是来看看 EXC_CRASH (SIGSEGV)

**Crash Log分析的标准姿势**一般是这样的：

1. 在用户的Crash日志文件查看应用版本，找到发布版本时的Archive包；
2. 其实我们只需要Archive包中的.app文件或者dSYM文件，二者中的一个即可；
3. 符号化日志中的内存地址，符号化出来的日志文件中的地址会被转换成对应的代码及行数，**符号化的方法**有如下几种：
	- mdimport dSYM文件，步骤如下：
		1. 找到对应版本的dSYM文件
		2. 导入到Xcode中，注意要用绝对路径，运行代码：
			
				mdimport /Users/kimimaro/Desktop/log/XXXAPP.app.dSYM/
		3. 在Xcode中运行Re-symbolicate后查看结果
		4. 顺便吐槽一下新版Xcode越来越不好用，Xcode6还没找到从外部导入crash文件之后执行Re-symbolicate的方法。真机里面的crash log用Xcode打开后，Window -> Devices -> 找到对应设备 -> View Device Logs -> 右键对应Log条目 -> Re-symbolicate。
	- 利用symbolicatecrash工具（**因为方便所以更为常用**）
		1. 找到Symbolicatecrash文件（Symbolicatecrash文件独立于Xcode，可以拷出来使用）;
	
				/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/PrivateFrameworks/DTDeviceKit.framework/Versions/A/Resources/
			
			或运行如下命令找到
			
				find /Applications/Xcode.app -name symbolicatecrash -type f

		3. 用命令将symbolicatecrash拷贝到桌面的crash文件夹里面，与.app和.app.dSYM放一起，将Crash文件也拷到当前文件夹里面;
		
				cp /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/PrivateFrameworks/DTDeviceKitBase.framework/Versions/A/Resources/symbolicatecrash /Users/kimimaro/Desktop/crash
		
		5. 终端中输入命令，命令格式：Symbolicatecrash + 崩溃日志 + APP对应的.dSYM文件 + > + 输出到的文件;
				
				Symbolicatecrash .crash .dSYM > a.log
		6. 将终端完成以后，在crash文件夹里面会多出一个文件a.log，这个就是最终的文件，可以查看bug所在；
		7. 如果提示"DEVELOPER_DIR" is not defined;
		 
				在终端中输入： export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer
	- 直接执行atos命令（symbolicatecrash里面也是用了这个工具）
		1. 计算symbol address，iOS某个版本之后新的crash日志的地址计算方式发生了改变，所以需要我们先手动计算一下（当然也可以写脚本算哈~）
		
				slide: 0x00001000 
				stack address: 0x0006da48 
				load address: 0x5c000
				
				symbol address = slide + stack address - load address = 00012a48
		
		2. 找到symbol address对应的代码行
		
				atos -arch [arch] -o [dwarf]   [address]
				eg. atos -arch armv7 -o Joke.app/Joke 00012a48
				
				atos命令的一些参数如下：atos -o EXECUTABLE -arch ARCHITECTURE -l LOAD_ADDRESS ADDRESS
		3. 再啰嗦一句，如果依赖atos写脚本的话就用：
		
				xcrun atos -arch armv7s -o [dwarf]   [address]
4. 在符号化后的日志中找到：
		
		Triggered by Thread:  0

	根据所指示的找到崩溃的线程，然后happy happy的在里面找到崩溃的函数；
	
&emsp;但是往往当我们找到崩溃线程的时候发现不能happy的解Bug了，因为对应的线程里面竟然全都是系统调用，一个应用内调用的方法都没有，ORZ。

>接下来的分析可能就根据对应问题不同分析的手段也不同，但是总得来说还是有一些规律可寻的。

###2. Exception Type & C signle

&emsp;上面Log里面的可以找到这样两行：

	Exception Type:  EXC_CRASH (SIGSEGV)
	Exception Codes: 0x0000000000000000, 0x0000000000000000

&emsp;第一行是这个崩溃日志的类型，由"Mach Exception"（以"EXC_"开头的）和"UNIX Signal"（如 SIGSEGV, SIGBUS 等）两部分组成。对于某些exceptions，也会附加一个处理器特定的Exception Code / Exception Subtype，包含和exception有关的信息。比如：“EXC_BAD_ACCESS”下面经常会看到“KERN_INVALID_ADDRESS at 0x80000010”这样的exception code；“EXC_RESOURCE”下面会看到“WAKEUPS”这样的exception subtype。

&emsp;**Mach Exception常见的有如下几种：**

Exception类型|描述|说明
:---------------|:---------------
EXC_BAD_ACCESS|Bad Memory Access|错误内存地址，访问的地址不存在或者当前进程没有权限都会报这个错。通常后面跟随的UNIX Signal是**SIGBUS**或者**SIGSEGV**
EXC_CRASH|Abnormal Exit|通常跟随的UNIX Signal是**SIGABRT**，当前进程被系统检测到有异常行为而杀死
EXC_BREAKPOINT|Trace / breakpoint Trap|通常跟随的UNIX Signal是**SIGTRAP**，一般来说代码中主动抛出异常时发生。
EXC_GUARD|Violated Guarded Resource Protection|侵犯了被监视资源的安全性，比如：确定的文件描述
EXC_BAD_INSTRUCTION|Illegal Instruction|非法或未定义的指令或操作数
EXC_RESOURCE|Resource Limit|达到资源极限时的App Crash
00000020|Hexadecimal Exception Type|非“OS内核”异常

>完整的Mach Exception列表可以在[这里](http://fxr.watson.org/fxr/source/osfmk/mach/exception_types.h?v=xnu-2050.18.24)的源代码文件（sys/osfmk/mach/exception_types.h）中找到。

&emsp;**UNIX Signal常见的有如下几种：**

UNIX Signal|解释说明
:---------------|:---------------
SIGSEGV|访问了无效的内存地址，这个地址存在，但是当前进程没有权限访问它。**属于硬件层错误**
SIGABRT|程序Crash，这个符号是由C函数abort()触发的。通常代表系统发现当前程序执行出错。**属于软件层错误**
SIGBUS|访问了无效的内存地址，与SIGSEGV的区别是：SIGBUS表示内存地址不存在。**属于硬件层错误**
SIGTRAP|Debugger相关
SIGILL|尝试执行一个非法的、未知的、没有权限的指令

&emsp;上面两个表格只能帮助理解区分，实际定位问题时需要更深入的理解。举个例子说，比如上述Crash Log实例中：

2. EXC_CRASH
	
	* EXC_CRASH is a mach exception that just means the application terminated abnormally. The parenthetical is the signal that caused the exception, in your case it's SIGABRT which almost always means that you have an un-handled exception somewhere or you have some code that is calling abort() for some reason (again, generally the un-handled exception handler calls this in the end).
	* 可能导致问题的原因：
		- unrecognized selector
	* 调试方法：
		- All Exception Point

2. EXC_BAD_ACCESS

	* 可能导致该问题的原因：
		- memory errors
	* 调试方法：
		- NSZombie
		
3. SIGSEGV
		
	* 在POSIX兼容的平台上，SIGSEGV是当一个进程执行了一个无效的内存引用，或发生段错误时发送给它的信号。SIGSEGV的符号常量在头文件signal.h中定义。因为在不同平台上，信号数字可能变化，因此符号信号名被使用。通常，它是信号#11。
	* SIG是信号名的通用前缀。SEGV是segmentation violation（段违例）的缩写。
	* 它會出現在當程式企圖存取CPU無法定址的記憶體區段時。
	* 可能导致该问题的原因：
		- 引用released对象
		- 引用从未init的对象
		- 数组越界
3. SIGABRT
	
	* 软件层（系统层）的错误
	* 可能导致该问题的原因：
		- try to free the same memory twice
		- raise Exception
3. 结合Exception Type和UNIX Signal可以看出：
	- 这个Log中显示的崩溃是一种比较异常的情况：（Mach Exception Type和C signal常识上不匹配）；
	- 程序崩溃的原因比较大的可能仍是**访问了无效的内存地址**。

###3. 崩溃线程中的系统调用
>接着Exception Type部分往下看，程序崩溃在了Thread 0，那么我们就来看一下Thread 0的内存堆栈。

&emsp;如上面所说崩溃线程中除了main.m方法属于App源码外，其他皆为系统调用。
	
	2   CoreFoundation                	0x2eee3fec __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 12
	3   CoreFoundation                	0x2eee34b2 __CFRunLoopDoSources0 + 202
	4   CoreFoundation                	0x2eee1ca2 __CFRunLoopRun + 626
	5   CoreFoundation                	0x2ee4c764 CFRunLoopRunSpecific + 520
	6   CoreFoundation                	0x2ee4c546 CFRunLoopRunInMode + 102
	7   GraphicsServices              	0x33db96ce GSEventRunModal + 134
	8   UIKit                         	0x317ab88c UIApplicationMain + 1132
	
上面这个部分的堆栈是很平常的系统调用堆栈，在main方法执行后通常都会看到它们。重点是再上面这一句：

	1   Foundation                    	0x2f901c1e __NSThreadPerformPerform + 382

这个方法告诉我们很大可能是**在我们调用performSelector:系列方法时程序发生崩溃**的。

	_NSThreadPerformPerform is used by the performSelector... family of methods. So look at your use of those methods. In particular, figure out if it's possible that you're asking an object to perform a selector that it doesn't support. That would throw an exception.

###4. 判断当前程序正在运行的时机
>其他线程虽然不是导致崩溃的线程，但是根据其他线程中的可识别代码，可以大致判断出当前程序所处于的时期（程序启动、程序运行期、前后台切换等等），缩小问题定位的范围。

&emsp;在上述Crash Log例子中通过其他线程正在执行的方法判断出此时程序正在启动（各种单例初始化的过程中），再从这部分逻辑中找到performSelector:相关的代码逻辑，虽然不是最好的办法，但是通常通过这种方式可以把问题的查找方位逐渐缩小，提高找到问题原因的可能性。

#常见Crash类型梳理（持续完善中...）
>之前每次查crash log都是查完就扔到一般，其实每种崩溃日志都有其特定的特点。下面列一些常见的Crash类型，也作为自己开发的经验积累，不断扩展中...

* 内存问题 or 方法调用（调用高版本API、未知方法）
* 数组越界
* 多线程（mutable array）
* unrecognized selector sent to instance （可能造成的原因：使用id指针、强制类型转换）
		
		id neverInit = [[Something alloc] init];
	    [neverInit methodNotOwn];
* memory errors
		
		viewController.list = [NSArray arrayWithObjects:@"One", @"Two"];
* Home键退后台后程序crash
	- 在app处于后台时，ios是不允许app调opengl的
	- [按Home键切换到后台后会触发libGPUSupportMercury.dylib: gpus_ReturnNotPermittedKillClient导致crash - Clin - 博客园](http://www.cnblogs.com/Clin/p/3405173.html)
	- 应用占用内存过高（通常UIWebView会出现），推到后台直接被杀死

#LLVM的使用
>顺带记录一下调试过程中查到的一些LLVM实用技巧，后续有机会也都总结出来。
9. po $exa
		
		The po command stands for “print object.” The symbol $eax refers to one of the CPU registers. In the case of an exception, this register will contain a pointer to the NSException object. Note: $eax only works for the simulator, if you’re debugging on the device you’ll need to use register $r0.

#参考资料
3. [SIGSEGV - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/SIGSEGV)
4. [SIGTERM, SIGSEGV, SIGINT, SIGILL, SIGABRT, SIGFPE - cppreference.com](http://en.cppreference.com/w/c/program/SIG_types)
5. [C signal handling - Wikipedia, the free encyclopedia](https://en.wikipedia.org/wiki/C_signal_handling)
6. [iphone - Exception Types in iOS crash logs - Stack Overflow](http://stackoverflow.com/questions/7446655/exception-types-in-ios-crash-logs)7. 
7. [My App Crashed, Now What? - Part 1 - Ray Wenderlich](http://www.raywenderlich.com/10209/my-app-crashed-now-what-part-1)
10. [Demystifying iOS Application Crash Logs - Ray Wenderlich](http://www.raywenderlich.com/23704/demystifying-ios-application-crash-logs)
1. sigaction: [Mac OS X Manual Page For sigaction(2)](https://developer.apple.com/library/ios/documentation/System/Conceptual/ManPages_iPhoneOS/man2/sigaction.2.html)

#My Question
>这次遇到的棘手问题也上传到了stackoverflow：

* [ios - A very interesting crash log with Exception Type:"EXC_CRASH (SIGSEGV)" - Stack Overflow](http://stackoverflow.com/questions/32480179/a-very-interesting-crash-log-with-exception-typeexc-crash-sigsegv)

#TODOs
1. Try Catch到的到底是什么？
2. Mach Exceptions深入研究
3. EXC_CRASH部分叫做Mach Exceptions，只有iOS和Mac才有，(SIGSEGV)是Unix Signal，所有基于Unix的系统都可以看到这样的符号？
4. ARC(retain, release, weak, strong, assign, unsafe_retained)
5. __kill 与 __pthread_kill 区别？
