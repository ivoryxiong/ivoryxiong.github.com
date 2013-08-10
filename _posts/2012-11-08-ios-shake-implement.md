---
layout: post
title: "ios的晃动检测"
description: "摇一摇随着微信的流行而为大家所知，之后的app中越来越多使用了晃动功能，本文详细介绍了ios中实现晃动检测的方法及使用场景。"
category: tutorial
tags: [ios, shake]
---
## 实现方式 ##
### 使用UIWindow+NSNotificationCenter实现 ###
首先Category扩展UIWindow，使UIWindow具有感知晃动的功能。如：UIWindow+IVAddtions.h 与 UIWindow+IVAddtions.m

``` objc
//UIWindow+IVAddtions.h
#import <UIKit/UIKit.h>
@interface UIWindow(IVAddtions)
-(void) motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event;
@end

//UIWindow+IVAddtions.m
#import "UIWindow+IVAddtions.h"
@implement UIWindow(IVAddtions)
-(void) motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event {
    if (event.subtype == UIEventSubtypeMotionShake) {
	[[NSNotificationCenter defaultCenter] postNotificationName:@"shake_event" object:nil]
    }
}
@end
```

现在假设IVProblemViewController需要使用晃动检测，则只需要该类在init的时候向消息中心添加观察者，在dealloc时从消息中心移除。代码如下：

``` objc
//IVProblemViewController.m
- (id) init {
	self = [super init];
	if (self) {
		[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(handleShake:) name:@"shake_event" object:nil];
		// other code ...
	}
	return self;
}

- (void) dealloc {
	[[NSNotificationCenter defaultCenter] removeObserver:self];
	[super dealloc];
}
- (void) handleShake:(NSNotification *) note {
	// handle code ...
}
```

### 在UIViewController的子类里实现 ###
  在需要使用晃动检测的UIViewController子类里分别实现motionEnded:withEvent:, becomeFirstResponser,并在view展示前发送消息becomeFirstResponser消息，
在view退出屏幕前发送消息resignFirstResponser信息。简要代码如下：

``` objc
//IVProblemViewController.m
- (id) viewWillDisplay {
	[super viewWillDisplay];
	[self becomeFirstResponser];
}

- (void) viewWillNotdisplay {
	[self resignFirstResponser];
	[super viewWillNotdisplay];
}
- (void) handleShake {
	// handle code ...
}
-(void) motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event {
    if (event.subtype == UIEventSubtypeMotionShake) {
	[self handleShake];
    }
}
```

## 使用场景 ##
大多数场景下，以上两种实现方式效果都一样。在一些特殊的场景中，第二种实现方式会显得不够健壮，如问题：[在tabbar来回切换后晃动检测失效](http://www.cocoachina.com/ask/questions/show/56190)，但使用NSNotificationCenter的实现方式则能正常工作，具体原因还需探究。
