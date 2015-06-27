---
layout: post
title: "UIImageView和UIButton拉伸效果的实现"
date: 2015-06-27 11:49:22 +0800
comments: true
categories: 
---

UIImageView中Image拉伸效果的实现：


    UIImageView *strechTest = [[UIImageyiView alloc] initWithImage:[UIImage imageNamed:@"test.png"]];
    [strechTest setContentStretch:CGRectMake(0.5f, 0.5f, 0.f, 0.f)];
    CGRect frame = strechTest.frame;
    frame.size.width += 100;
    strechTest.frame = frame;

 

但是虽然UIButton和UIImageView都是继承于UIView，但是二者实现方式不同，UIImageView没有subView，它 的content就是image，UIBotton不同，the way it works is a private implementation detail。

因此UIButton实现背景拉伸，即图片两端不拉伸中间拉伸的办法有如下两种：

第一种方法很简单而且使用性更广。做法就是直接拉伸想要setBackgroundImage的image，代码如下：

	UIImage*image =[UIImage imageNamed:@"image.png"];
	
	image = [image stretchableImageWithLeftCapWidth:floorf(image.size.width/2) topCapHeight:floorf(image.size.height/2)];
	
	image =[image stretchableImageWithLeftCapWidth:floorf(image.size.width/2) topCapHeight:floorf(image.size.height/2)];

设置了左端帽之后，rightCapWidth = image.size.width - (image.leftCapWidth + 1); 也就是说图片中间的一像素用来拉伸。垂直方向同上。

设置之后无论把image放到什么控件中都可以自动拉伸了。

第二种方法是在UIButton中加入一个UIImageView，拉伸imageView，然后将button的背景设为clearColor等等。把imageView放入button中，并且sendToBack，得到效果。

代码如下：

	//刚才imageView拉伸的代码

    UIImageView *strechTest = [[UIImageyiView alloc] initWithImage:[UIImage imageNamed:@"test.png"]];
    [strechTest setContentStretch:CGRectMake(0.5f, 0.5f, 0.f, 0.f)];
    CGRect frame = strechTest.frame;
    frame.size.width += 100;
    strechTest.frame = frame;

	//把imageView放入button中，并设置为back


    [temp_button addSubview:backgroundImageView];
    [temp_button sendSubviewToBack:backgroundImageView];
    [temp_button setBackgroundColor:[UIColor clearColor]];

 

button不能设置背景图片，这样就可以实现拉伸的图片作为背景并且背景上可以放置title。