---
layout: post
title: "APNs实现小记"
date: 2014-06-27 11:07:51 +0800
comments: true
categories: iOS
---

> 转自己以前写的博客，很早时候的学习笔记了，不过下面几个连接貌似都失效了。。。囧

推荐如下链接，前两个介绍APNs的设置流程，已经很详尽，虽然很详尽但是个人感觉有些晦涩难懂，把简单的流程搞的复杂。我下面的内容是我设置APNs的过：

<!--more-->
[iPhone消息推送服务实现 - vber的专栏 - 博客频道 - CSDN.NET]

[apple push notification service apple与Python结合推送 - iOS开发讨论区 - Tiny4Cocoa]

[编写push notification之获取device token | Marshal's Blog]

[偷窥iPhone Push Notification的幕后 - SLJ.me - 申力军](http://slj.me/2010/02/iphone-push-notification/)

[iPhone的Push(推送通知)功能原理浅析 - iPhone/iPad 进阶讨论区 - 悦Phone论坛 iPhone|iPad开发者、苹果开发者]
 

&emsp;第一次接触APNs，鼓捣了一下午终于有了点眉目。

&emsp;链接1,2的流程介绍都是从最基础的入门级介绍，其实在你开发一款iOS app时，你需要做的是申请开发者帐号，设置Apple ID，生成provisioning file导入到Xcode，而APNs只是在设置Apple ID的时候多选择了一个选项，即Apple ID 的configure页面的 Enable for Apple Push Notification service选项，勾选它，然后根据你的需要生成dev或者release版本的SSL证书（证书生成的过程apple已经给了详尽的提示）。这里要注意的是，勾选了enable for apns之后，其实你的provisioning file已经改变，所以要重新下载新的导入Xcode。

-------------------
此时，你的app已经支持apns了。

&emsp;之后需要做的是两个方面：

1. 客户端 在appDelegate中编写代码，为了阅读方便就把上面链接中的代码重新贴一次：

	    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {  
	      [[UIApplication sharedApplication] registerForRemoteNotificationTypes:(UIRemoteNotificationTypeAlert |    UIRemoteNotificationTypeBadge |UIRemoteNotificationTypeSound)];  
	    }  
	    - (void)application:(UIApplication *)app didFailToRegisterForRemoteNotificationsWithError:(NSError *)err {  
	      [self alertNotice:@"" withMSG:[NSString stringWithFormat:@"Error in registration. Error: %@", err] cancleButtonTitle:@"Ok" otherButtonTitle:@""];  
	    }  
	    - (void)application:(UIApplication *)app didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {  
	      NSLog(@"devToken=%@",deviceToken);  
	      [self alertNotice:@"" withMSG:[NSString stringWithFormat:@"devToken=%@",deviceToken] cancleButtonTitle:@"Ok" otherButtonTitle:@""];  
	    } 

&emsp;其实就是在appDidFinishLaunch的时候注册（register）apns，然后通过didRegisterForRemoteNotificationsWithDeviceToken得到DeviceToken。

接受服务器消息并改变客户端本地状态（如在app图表显示带数字的小红圈）的代码如下：

		－(void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
		
		    for (id key in userInfo) {
		        NSLog(@"key: %@, value: %@", key, [userInfo objectForKey:key]);
		    }    
		}

&emsp;到此为止，当你在真机上运行你的app时就会出现：是否允许app显示push notification的提示框。

2. 服务器端 服务器端可以用PHP，Python等实现，网上也有一些现成的库如开源库APNSWrapper。现在你会发现我理解的思路跟链接1,2的介绍流程稍有不同，同时你会发现我还没有使用Enable for Apple Push Notification service时生成的证书文件，现在回到Apple ID configure页面下载生成的 Development (or Production) Push SSL Certificate 文件，关于这个文件要多说几句：

>这个文件的作用是让apple的APNs服务器确认服务器端具有push notification的权限。将这个证书作一定的处理（＊至于是什么处理为了不影响文章的连贯性会在下面介绍），生成一个ck.pem文件（这个名字可以自定义）。

&emsp;然后就要涉及到APNs的工作机制，我的总结如下：当一个device（苹果设备）启用apns功能（就是通知功能）的时候会和apple的APNs服务器建立TLS连接，APNs服务器根据device的UDID和加密密钥新建一个DeviceToken（它的作用是让服务器端指定device，唯一）返回给device，然后device会把device token给你的app客户端，这时会调用刚才写的didRegisterForRemoteNotificationsWithDeviceToken方法，你可以在这个方法中将device token返回给服务器端。此后，如果服务器端push了一个notification给客户端，需要将这个device token和之前处理的ck.pem文件同时发送给APNs服务器，APNs服务器会验证你的ck.pem文件然后根据device token指定的设备推送消息给客户端）。当客户端接受到push的通知时调用didReceiveRemoteNotification方法，你可以在这个方法中放入你需要的操作。

>现在再回到Apple ID configure页面下载生成的 Development (or Production) Push SSL Certificate 文件，对它的处理链接1,2都有介绍，就是：

1. 选择“我的证书”, 选定推送服务证书(Apple Development Push Services*),导出到桌面,保存为Certificates.p12。

2. 在终端中运行如下命令:

    	openssl pkcs12 -clcerts -nokeys -out cert.pem -in Certificates.p12  
    	openssl pkcs12 -nocerts -out key.pem -in Certificates.p12  
    	openssl rsa -in key.pem -out key.unencrypted.pem  
   	 	cat cert.pem key.unencrypted.pem > ck.pem  

		ck.pem文件则是PHP推送消息时所使用的证书文件。

&emsp;你剩下的任务就是编写服务器端的代码，push notification，这部分只要记住发送device token和证书文件就可以了。

（以上就是我理解学习APNs的过程，许多理解都不深入，所以描述也不准确，包涵～）


##更新：

**开发中遇到的问题汇总**：

1. 正常的iPhone刷系统之后，是没有设备证书和密钥的。这就是为什么iPhone会需要连接到 iTunes上进行激活——激活过程中，Apple会分配给每台iPhone独一无二的设备证书(device certificate)和密钥(key)。(如果没有意识到这个会一直收到 "未找到应用程序的"aps-environment" 的权利字符串)

2. 如果生成appleID的时候没有enable aps，那么enable之后一定要删掉原来的dev和rc证书重新生成，这样才能保证开启新功能，否则仅仅是刷新这张证书仍然会没有aps，提示（ "未找到应用程序的"aps-environment" 的权利字符串）

3. APNs证书如果是系统证书那么最好拷贝到登陆证书中，这样可以直接导出成p12

 

 

 