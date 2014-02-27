---
layout: post
title: "iOS开发小结之iOS7新特性(一)"
description: "iOS7发布很久了，随着iOS7的份额不断增大，目前新上架或更新的App都必须使用SDK7编译，也该是时候汇总一下iOS7的一些新特性，方便大家更深入利用这些新特性创造更酷炫、体验更好的App"
category: devops
tags:
    - ios
    - View Controller Transitioning API
    - 视图控制器切换API
    - 多任务处理API
    - Background Fetch
---

关于iOS7的新特性，官方已经在开发者网站给出了详细的[列表](https://developer.apple.com/library/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS7.html), 不过这里先讲讲两个对一般开发者【非游戏开发者】而言比较重要的新特性[Multitasking Enhancements](https://developer.apple.com/library/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS7.html#//apple_ref/doc/uid/TP40013162-SW10)和[Customizing View Controller Transitions](https://developer.apple.com/library/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS7.html#//apple_ref/doc/uid/TP40013162-SW19)。

#### 多任务处理API ####
根据数据传输量的大小，iOS7新增加了后台获取（Background Fetch）和后台传输服务（￼Background Transfer Service），而为了唤醒已休眠的设备进行后台处理，同时也增加了远程静默推送（Silent Remote Notifications）。在iOS7之前，多任务处理中，在后台可进行的处理十分有限，见Apple的文档[App状态与多任务](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/ManagingYourApplicationsFlow/ManagingYourApplicationsFlow.html#//apple_ref/doc/uid/TP40007072-CH4-SW20)，这直接导致一些App无法实现一些挺酷的功能，比如阅读类应用里的离线下载【类似网易阅读的做法其实交互并不有好，下载过程中用户无法进行其他操作】、定时下载【iOS7之前只能做个残缺版，设定时间到时，App需要处理Active状态】。iOS7有了新API后，这类服务就很好实现了，离线下载过程直接使用后台传输服务，在不影响用户使用的情况下下载数据；对于定时功能，则后台保存设置的时间，到时间则Push一条远程静默通知，App接受后启动下载过程，并再完成后使用本地通知通知用户。这里只简单介绍下新API的使用场景，更多、更具体的使用介绍见文章后面的阅读资料。

#### 视图控制器切换API ####
动画效果是iOS应用对Android应用的一大优势，但iOS7之前，开发者多数在某个页面花心思，而比较少关注页面之间的转场效果。究其原因，SDK提供的几个导航控制器【UINavigationController、UITabbarController及ModalViewController】都不支持自定义转场动画，除非另外实现导航控制器【ContainnerViewController】，但这个实现复杂度就大大提高【无觅阅读就使用了自己实现带有右滑返回功能的导航控制器】。iOS7里分别在UINavigationControllerDelegate、UITabBarControllerDelegate增加了控制动画效果的回调，开发者能方便地从回调里取出前后两个ViewController，并做相关的动画。具体使用方法也不多介绍了，不过想用好，还是得对CoreGraphics里面的动画基础多了解【具体就是仿射变化CGAffineTransform、CATransform3DCAAnimation】。


### 阅读资料 ###
* [WWDC 2013 Session笔记 - iOS7中的多任务](http://onevcat.com/2013/08/ios7-background-multitask/)
* [Multitasking in iOS 7](http://www.objc.io/issue-5/multitasking.html)
* [View Controller Transitions](http://www.objc.io/issue-5/view-controller-transitions.html)
* [iOS 7 Tutorial Series: Custom Navigation Transitions & More](http://www.captechconsulting.com/blog/tyler-tillage/ios-7-tutorial-series-custom-navigation-transitions-more)
* [Quartz 2D Transform](https://developer.apple.com/library/mac/documentation/graphicsimaging/conceptual/drawingwithquartz2d/dq_affine/dq_affine.html#//apple_ref/doc/uid/TP30001066-CH204-CJBECIAD)